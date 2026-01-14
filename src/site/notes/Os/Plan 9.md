---
{"dg-publish":true,"dg-permalink":"os/plan9","permalink":"/os/plan9/"}
---

* [Plan 9 from Bell Labs](https://en.wikipedia.org/wiki/Plan_9_from_Bell_Labs), [Plan 9 (Betriebssystem)](https://de.wikipedia.org/wiki/Plan_9_(Betriebssystem))
* Doc1 http://plan9.bell-labs.com/sys/man/index.html
* Doc2 http://www.cs.bell-labs.com/sys/doc/index.html
* Plan 9 [Tutorial](http://www.magma.com.ni/moin/Plan9Tutorial) von Magma Soft
##  short notes 
* default user is *glenda*
##  Installation 
###  Qemu 
* http://plan9.bell-labs.com/wiki/plan9/Installing_Plan_9_on_Qemu/index.html
##  UI 
* antialiased fonts in Plan 9
    * http://mirtchovski.com/p9/freetype/ (mit hget z.B. in /tmp/)
* customize keyboard layout
    * cat /sys/lib/kbmap/[lang] > /dev/kbmap
    * make default for user in /usr/[username]/lib/profile
* [The Acme environment in Plan9](http://thenewsh.blogspot.com/2010/01/acme-environment-in-plan9.html)