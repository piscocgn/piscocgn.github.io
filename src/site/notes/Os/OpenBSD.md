---
{"dg-publish":true,"dg-permalink":"/os/openbsd","permalink":"/os/openbsd/"}
---

#unix 

* Dienste neu starten

```bash
  kill -HUP `cat /var/run/sshd.pid`
```
  
##  GNOME 
* [Gnome README](http://www.openbsd.org/cgi-bin/cvsweb/ports/x11/gnome/session/files/Attic/README.OpenBSD?rev=1.42;content-type=text%2Fplain)
* gnome-session gnome-panel metacity

##  Login über GDM 
Standardmäßig ist der XDM-Loginmanager aktiviert. Dieser kann in der Datei ''/etc/rc'' deaktiviert werden.

`/etc/rc`
```diff
--- a/rc	Wed Jan 26 18:46:15 2011 +0100
+++ b/rc	Wed Jan 26 18:48:15 2011 +0100
@@ -847,8 +847,8 @@
 fi

 # Alternatively, on some architectures, xdm may be started in /etc/ttys.
-if [ X"${xdm_flags}" != X"NO" -a -x /usr/X11R6/bin/xdm ]; then
-	echo 'starting xdm...';		/usr/X11R6/bin/xdm ${xdm_flags}
-fi
+#if [ X"${xdm_flags}" != X"NO" -a -x /usr/X11R6/bin/xdm ]; then
+#	echo 'starting xdm...';		/usr/X11R6/bin/xdm ${xdm_flags}
+#fi

 exit 0
```

Folgender Eintrag startet gdm beim Systemstart (info aus  ''pkg_info gdm''):
`/etc/rc.local`
```diff
--- a/rc.local	Wed Jan 26 18:49:31 2011 +0100
+++ b/rc.local	Wed Jan 26 18:50:05 2011 +0100
@@ -9,6 +9,9 @@

 # Add your local startup actions here.

+if [ -x /usr/local/sbin/gdm ]; then
+        echo -n ' gdm'; (sleep 5; /usr/local/sbin/gdm)&
+fi

 echo '.'
```
