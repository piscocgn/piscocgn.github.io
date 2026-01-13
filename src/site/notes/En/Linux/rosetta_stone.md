---
{"dg-publish":true,"permalink":"/en/linux/rosetta-stone/"}
---

## Rosetta Stone Linux (or Inter-Linux)
The following scheme compares some commands and tasks by some distributions, to face differences and similarities. Maybe it's usefull when switching to an other Linux-Distributon.
<flattr>
title=Rosetta Stone Linux (or Inter-Linux)
uid=30553
description=A cheat sheet for Distribution-hoppers
category=software, linux
align=left
</flattr>

##  package management 

|Action |Debian((http://www.us.debian.org/doc/manuals/reference/ch-package.en.html#s-apt-install)) | Red Hat/CentOS/Fedora((http://docs.fedoraproject.org/yum/en/)) | openSUSE(([Zypper](http://en.opensuse.org/Zypper) since 10.2 since Beta1. It uses libzypp.)) | Gentoo((http://www.gentoo.de/doc/en/handbook/handbook-x86.xml?part=2&chap=1)) |
|---|---|---|---|---|
|update repositories | aptitude update| -- | zypper refresh | emerge --sync |
|install upgrades | aptitude safe-upgrade| yum update | zypper update | emerge -u[p]D world |
|search package by name | aptitude search [package]| yum list [package]| zypper search [package] | emerge -s [package] |
|show package description | aptitude show [package]| yum info [package]| zypper info [package] | emerge -s [package] |
|install package | aptitude install [package]| yum install [package]| yast2 install / -i [package]  zypper install | emerge [package] |
|show the package to wehre  a specified file belongs |dpkg -S [filename]|  rpm -qf [filename]  || equery((The command *equery* shipped by the package *app-portage/gentoolkit*)) belongs [filename] |
|which of the not installed  packages ships a specified file |apt-file find [filename] ((you have to run apt-file update the first time to build the cache))| yum whatprovides [filename] | |  -((Gentoo compiles packages/software from source. So usualy it's not possible to query files against non-installed packages.))  |
|show content of a non-installed package  available in the repositories | apt-file -F list [package] ((you have to run apt-file update the first time to build the cache)) | | | |
|show content of a local available  package file | dpkg -c [package.deb]((-c or --contents)) |  rpm -qpl [package.rpm]  || |
|show information of a local available  package file | dpkg -I [package.deb]((-I or --info)) |  rpm -qpi [package.rpm]  || |
|verify installed packages| debsums [package]((The MD5-databes must be initialized before with *debsums_init*.))  debsums -as|  rpm -V [package]  rpm -Va  ||  -  |
|extract package  without installation | 1. ar xv [package].deb  2. tar -xzvf data.tar.gz |  <html>rpm2cpio [package].rpm | cpio -idmv</html>  ||  -((Because of Gentoo will be compiled from the source code of packages, it is possible to do a ''emerge -f [package]'', then copy the package from ${DISTDIR} (e.g. ''/usr/portage/distfiles'')  and extract with tar))  |
|build package from source | apt-build install --rebuild [package]((maybe you have to run apt-get build-dep [package] before to install required libraries.)) |  rpm -i [package].src.rpm  cd /usr/src/packages/SPEC  rpmbuild -ba [package].spec  || this is done by default in gentoo |
| compile kernel |||||
|---|---|---|---|---|
|install required tools | aptitude install build-essential | | zypper install -t pattern devel_kernel | | 
|compile kernel an build package | sudo fakeroot make-kpkg   --revision $REVISION --initrd   kernel_image modules_image | make oldconfig    maybe ''rm *.spec''((when doing a new revision))  make binrpm-pkg   rpm -i /usr/src/packages/RPMS/i386 ..   mkinitrd -k /boot/Kernel-file -i /boot/initrdfile  ||  |

##  management of users and groups  

|Action |Debian((http://www.us.debian.org/doc/manuals/reference/ch-tutorial.en.html#s-newuser)) | Red Hat/CentOS/Fedora(([Documentation CentOS](http://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-users-groups.html))) | openSUSE | Gentoo |
|---|---|---|---|---|
|add user |  useradd [-m][-g primary group((The primary group also called*initial group* or *login group*))] user  ||||
|change users primary group((The primary group also called*initial group* or *login group*))|  usermod -g group user  ||||
|delete user |  userdel [-r] user  ||||
|add group |  groupadd group  ||||
|add user user to a  group | adduser user group((Debian-specific))  gpasswd -a user group | usermod -aG group[,group] user  gpasswd -a user group |groupmod -A user group | usermod -aG group[,group] user  gpasswd -a user group |
|remove a user from a specified group | deluser user group((Debian-specific))  gpasswd -d user group | gpasswd -d user group |groupmod -R user group | gpasswd -d user group |
|declare a user to a groups administrator | gpasswd -A user,... group ||no more impelemted (since 10.x?) ((over LDAP))| gpasswd -A user,... group |
|add a list of users  to a specified group ((replace all existing memberships))|  gpasswd -M user,... group  ||||

##  system settings 
|action |Debian | Red Hat/CentOS/Fedora | openSUSE | Gentoo |
|---|---|---|---|---|
| change umask system-wide | /etc/profile |  |  /etc/login.defs | |
|activate services for (Default-) Runlevel | update-rc.d [name] defaults |  insserv [name] ''|'' chkconfig -a name(([Unofficial SUSEFAQ - Starting and Stoping Services: insserv](http://susefaq.sourceforge.net/faq/services.html))) || |


#linux #unvollständig 