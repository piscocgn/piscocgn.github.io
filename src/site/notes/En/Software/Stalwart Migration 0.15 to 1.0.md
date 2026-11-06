---
{"dg-publish":true,"dg-permalink":"en/software/stalwart-migration","permalink":"/en/software/stalwart-migration/","dg-note-properties":{}}
---

#stalwart #mail #migration #docker #ldap

Field report: migrating a small Stalwart mail server from 0.15.5 to 1.0.3. The setup: Docker Compose, RocksDB, OpenLDAP as the directory, a reverse proxy in front, fewer than 10 accounts, about 13 GB of data.

The migration took place in November 2026 using `stalwart:v0.16.27`, `stalwart-migrate:1.0.6`, `stalwart:v1.0.3` and `stalwart-cli` 1.0.14. Object and field names may change in later versions. All names are placeholders (`example.com`, `<...>`). The official upgrade documentation takes precedence over this page.

**Deutsche Fassung:** [[Software/Stalwart Migration 0.15 auf 1.0\|Stalwart Migration 0.15 auf 1.0]]

## Key points

* There is no direct path: 0.15 → 0.16 → `stalwart-migrate` → 1.0.
* The first start of 0.16 changes the data irreversibly. Settings and directory records are wiped. Work on a copy only, and take a dump first.
* `:latest` now points to 1.0. Pin the old image before you begin, otherwise an accidental pull starts the wrong version on the old data.
* The migration itself went smoothly. The problems came afterwards and involved roles, IP bans, logging and certificates. The checklist below covers them.
* A rehearsal on a copy is well worth it, but only if it ends with real logins. Ours did not, so the worst bug only showed up in production.

## Result

| Phase | Duration |
|---|---|
| Copying 13 GB of data | 2 minutes |
| Starting 0.16, importing the configuration | 1 minute |
| `stalwart-migrate` including full verification | 5 minutes |
| Total outage until 1.0 was public | 9 minutes |
| Logins disrupted afterwards | about 13 minutes, one admin account about 25 minutes |

`verify --full` reported all accounts without a mismatch. The login disruption would have been avoidable with the checklist below.

## Preparation

1. Pin the image and set `pull_policy: never`:
```yaml
image: stalwart-local:pre-upgrade-0.15.5   # local tag of the running image
pull_policy: never
```
2. Run a rehearsal on a copy of the data. Keep the copy isolated (`docker network create --internal`) so it cannot deliver mail.
3. Check which services use the mail server internally. Webmail may reach ports that are not published at all, such as 465 and 4190.
4. Compare content: message count and total size per account via JMAP. The quota counter is no use for this. In 0.15 ours was too low by more than a factor of 2, while the real total matched to the byte after migration.
5. Decide who will be admin. With LDAP active, local accounts cannot have credentials. Without LDAP, `STALWART_RECOVERY_ADMIN` is the only emergency access.
6. Tell your users. OAuth clients must sign in again, and CalDAV, CardDAV and WebDAV clients fetch their data again.

## Procedure

### 1. Dump from 0.15

The `migrate_v016.py` script from the upgrade documentation exports settings and principals through the API. This works while the server is running:

```sh
python migrate_v016.py dump --url https://mail.example.com --username admin \
  --settings settings.json --principals principals.json

python migrate_v016.py convert --settings settings.json --principals principals.json \
  --config config.json --output export.json \
  --patch-paths /opt/stalwart=/var/lib/stalwart --unmigrated-output unmigrated.txt
```

Review `unmigrated.txt`. In our case most keys were not carried over, including the many automatically created IP bans. That is normal, but the list is worth a look.

`export.json` is NDJSON. Obsolete domains, DKIM keys or aliases can be filtered out before the import. Afterwards, make sure no references such as `#create-N` are left dangling.

### 2. Stop and copy

```sh
docker stop -t 60 mailserver
cp -a <0.15-data> <work016>
chown -R 2000:2000 <work016>     # newer images run as UID 2000
```

The original data stays untouched and serves as the rollback.

### 3. 0.16 in recovery mode

`config.json` for 0.16:

```json
{ "@type": "RocksDb", "path": "/var/lib/stalwart/data" }
```

```sh
docker run -d --name stalwart-016 --network <internal-network> \
  -e STALWART_RECOVERY_ADMIN=admin:<password> -e STALWART_RECOVERY_MODE=1 \
  -v <c016>:/etc/stalwart -v <work016>:/var/lib/stalwart/data \
  stalwartlabs/stalwart:v0.16.27
```

The CLI runs in the container's network namespace:

```sh
docker run --rm --network container:stalwart-016 \
  -e STALWART_URL=http://127.0.0.1:8080 -e STALWART_USER=admin -e STALWART_PASSWORD=<password> \
  -v <folder>:/d:ro ghcr.io/stalwartlabs/cli:latest apply --file /d/export.json
```

### 4. Create the LDAP directory

Set fields are objects, not lists, so use `{"mail": true}` rather than `["mail"]`. Lists fail with `invalidPatch` or `invalid value object`.

```json
{
  "description": "LDAP",
  "url": "ldap://ldap.example.internal:1389",
  "baseDn": "dc=example,dc=com",
  "bindDn": "cn=admin,dc=example,dc=com",
  "bindSecret": "<secret>",
  "bindAuthentication": false,
  "filterLogin": "(&(objectClass=inetOrgPerson)(|(mail=?)(uid=?)))",
  "filterMailbox": "(&(objectClass=inetOrgPerson)(|(mail=?)(mailLocalAddress=?)))",
  "attrClass": {"objectClass": true},
  "attrEmail": {"mail": true},
  "attrEmailAlias": {"mailLocalAddress": true}
}
```

```sh
stalwart-cli create Directory/Ldap --stdin < directory.json
stalwart-cli update Authentication singleton --field directoryId=<id>
```

* `bindAuthentication: false` means hash comparison, which worked with `{SSHA}`.
* Neither 0.16.27 nor 1.0.3 appends the default domain to LDAP logins. For logins without a domain to keep working, `filterLogin` must also cover `uid`.
* `filterMailbox` from 0.16 becomes `filterRecipient` in 1.0. `stalwart-migrate` converts it.

### 5. stalwart-migrate

Stop 0.16. Target configuration for 1.0:

```json
{ "@type": "RocksDb", "path": "/var/lib/stalwart-v1/" }
```

```sh
A="--source /etc/stalwart/config.json --target /etc/stalwart-v1/config.json --state /var/lib/stalwart-migrate"
migrate prepare --non-interactive --source /etc/stalwart/config.json --state /var/lib/stalwart-migrate
migrate run $A
migrate run $A --final --yes
migrate verify --full $A
```

`migrate` stands for `docker run --rm -it --ulimit nofile=65536:65536 --network none` with the matching volumes and `stalwartlabs/stalwart-migrate:1.0.6`. The tool needs a TTY, so run the chain inside `screen`. For `--non-interactive`, `answers.toml` must be in the state folder. `prepare` writes this file itself, so you can reuse the one from the rehearsal.

### 6. Set up 1.0 in isolation first

This step is the main departure from the obvious route. Start 1.0 without published ports, configure everything and only then make it public. That way the server is never online half configured.

Do the first start once with `STALWART_UPDATE_ROLE_PERMISSIONS=true`, then remove the variable. The individual settings are in the checklist.

### 7. Switch over

```yaml
mailserver:
  image: stalwartlabs/stalwart:v1.0.3
  environment:
    - STALWART_PUBLIC_URL=https://mail.example.com
  volumes:
    - ./v1/config:/etc/stalwart:ro
    - ./v1/data:/var/lib/stalwart-v1
    - ./v1/logs:/var/log/stalwart
    - ./certs:/data/certs:ro
```

## Checklist before going public

Each of these was missing or caused trouble in our migration.

### Set the default roles

**This was the worst bug.** After the migration all `Default Roles` were empty. Accounts with `{"@type":"User"}` therefore had no permissions at all.

Symptom: the log shows `Authentication successful`, immediately followed by `Unauthorized access` with `reason = "authenticate"`. Clients just report a failed login. It looks like a password or LDAP problem, but it is neither.

```sh
stalwart-cli query Role --fields id,description
stalwart-cli update Authentication singleton \
  --field 'defaultUserRoleIds={"<user>":true}' \
  --field 'defaultGroupRoleIds={"<group>":true}' \
  --field 'defaultTenantRoleIds={"<tenant>":true}' \
  --field 'defaultAdminRoleIds={"<admin>":true,"<user>":true}'
```

Role IDs differ between installations. We did not find out why the fields were empty. Check them before going public, because without roles the `emailReceive` permission is missing too. Port 25 still accepted mail, but whether it was delivered during that time could not be established because of the missing log.

### Admins need two roles

The `System Administrator` role has no IMAP permissions. An admin account can reach the management interface but not its mailbox. The log shows `Unauthorized access` with `imapAuthenticate`. That is why `defaultAdminRoleIds` above lists both roles. Then give the account the role:

```sh
stalwart-cli update Account <id> --field 'roles={"@type":"Admin"}'
```

### Exempt the reverse proxy from automatic bans

Web access suddenly returned HTTP 502, while the container itself correctly answered 401. The cause: bots request paths such as `*.php` or `*/wp-*`, the scan detection only sees the proxy as the source and bans it, with no expiry. In our case this happened within minutes of starting.

```sh
stalwart-cli query BlockedIp --fields id,address,reason
stalwart-cli create AllowedIp --json '{"address":"<internal-subnet>","reason":"Reverse proxy"}'
stalwart-cli delete BlockedIp --ids <id>
```

Restart afterwards, otherwise the ban stayed active from the cache. A subnet is more robust than a single IP because container IPs can change when containers are recreated.

### Check the listeners

After the migration we were missing `imap` (143) and `submission` (587). The others were present:

```sh
stalwart-cli query NetworkListener --fields id,name,bind,protocol
stalwart-cli create NetworkListener --json '{"name":"imap","bind":{"[::]:143":true},"protocol":"imap"}'
stalwart-cli create NetworkListener --json '{"name":"submission","bind":{"[::]:587":true},"protocol":"smtp"}'
```

### Certificate

```sh
stalwart-cli create Certificate --json '{"certificate":{"@type":"File","filePath":"/data/certs/mail.example.com/cert.pem"},"privateKey":{"@type":"File","filePath":"/data/certs/mail.example.com/key.pem"}}'
stalwart-cli update SystemSettings singleton --field defaultCertificateId=<id>
```

* The new image runs as UID 2000. A `key.pem` with `root:root 600` is unreadable and TLS fails.
* If you use `traefik-certs-dumper`: `OVERRIDE_UID=2000` and `OVERRIDE_GID=2000` have no effect on existing, unchanged files. Set the ownership once yourself and check again after the next renewal. We have not been able to do the latter yet.
* Certificate files are only re-read at startup or via `stalwart-cli create Action/ReloadTlsCertificates`. We restart the container once a week via cron, which avoids keeping a password on the server.

### Check the logging

After the migration `docker logs` stayed completely empty, and the migrated tracer wrote to `/var/log/stalwart`, which does not exist in the container. So an empty log does not mean there are no errors. The fix: mount that folder as a volume, owned by UID 2000.

For troubleshooting, a temporary console tracer helps. It takes effect after a restart:

```sh
stalwart-cli create Tracer/Stdout --json '{"enable":true,"level":"debug","eventsPolicy":"exclude","events":{}}'
```

### Keep the CLI current

`stalwart-cli` 1.0.13 could not read the 1.0.3 schema; only 1.0.14 worked. Pull the CLI image again before the window.

## Tests after going live

Test from outside, not just on the container:

* Ports 25, 143, 465, 587, 993, and 4190 depending on your setup
* Login with short name and with full address, via IMAP and SMTP
* Webmail, also after every further restart or role change
* Admin interface, as well as the admin account's mailbox
* Mail in both directions, with a DKIM check at the receiving end
* Unknown addresses and removed domains must be rejected

A test with a deliberately wrong password quickly shows whether the web path is right: 401 is correct, 502 is not.

## Rollback

As long as 1.0 has not accepted new mail: point Compose back to the pinned image and the old volumes. The 0.15 data stays unchanged for this purpose. Mail received later would have to be carried over by hand, so decide early if there are problems.

The 0.16 working copy can go once 1.0 is confirmed working. We kept the 0.15 data for about another week.

## Other observations

* The spam training samples were gone after 0.16, which was expected. 1.0 loaded a spam model at startup. We did not check whether it matches the old one.
* `answers.toml` contains secrets in clear text if any were requested. Dumps and `export.json` also contain hashes and DKIM keys. Store such files with mode `600` and delete them after the migration.
* `STALWART_RECOVERY_ADMIN` also works in normal operation with LDAP down. Remove it after use.
* In shell scripts, `docker run -i` swallows the rest of the script if it runs via stdin. Append `</dev/null`.
* Never delete Account objects, they hold the link to the mailboxes.
