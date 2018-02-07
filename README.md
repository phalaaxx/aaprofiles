aaprofiles
===
AppArmor profiles for binary-only programs in Debian Jessie

Installation
---
It is necessary to install apparmor and optionally apparmor-profiles:
```sh
apt-get install apparmor apparmor-profiles
```
It is then necessary to enable apparmor from the bootloader configuration. To do so edit `/etc/default/grub` file and add `security=apparmor` at the end of `GRUB_CMDLINE_LINUX_DEFAULT` variable. Then run `update-grub` as root to enable the new configuration and finally reboot to activate apparmor LSM module.  
Check if apparmor is properly enabled in your system after reboot with the following command:
```sh
grep apparmor /proc/cmdline >/dev/null && echo OK
```
To enable AppArmor profiles copy each file in `/etc/apparmor.d` directory and change ownership and permissions:
```sh
chown root /etc/apparmor.d/{opt.viber.Viber,usr.bin.Skype}
chmod 0644 /etc/apparmor.d/{opt.viber.Viber,usr.bin.Skype}
```
These profiles while not great do very good job in adequately restricting these binary-only programs to a level that makes them secure enough for use in Linux.

opt.viber.Viber
---

This is a profile for viber 6.0.1.5 in Debian Jessie. Viber on Linux does not seem to attempt to do funny things like Skype does, but it's still a binary-only non-free program which is enough not to trust it.  
It's a bit hard to figure out how to install under Debian Jessie due to lack of proper error messages and documentation. In order to work it needs a couple of certificates that are not installed by default:
```sh
cd /usr/local/share/ca-certificates
wget https://www.thawte.com/roots/thawte_Premium_Server_CA.pem
wget https://raw.githubusercontent.com/katmagic/https-everywhere/master/cert-validity/mozilla/builtin-certs/Thawte_Premium_Server_CA.crt
update-ca-certificates
```
This profile allows read and write access to ${HOME} directory only (subdirectories are not allowed) to be able to send files from there and receive sent files.  

usr.bin.skype
---

Skype is the most untrustworthy binary-only (and lately Microsoft-owned) application I've ever seen running under Linux. For some unspecified reasons it eagerly attempts to read a number of files and walk directories, like:

* /etc/group
* /etc/passwd
* ${HOME}/.mozilla/
* etc.

This behaviour can be observed in AppArmor logs (audit or system logs) when the profile is active and skype is started.

This may all be just to be able to uniquely identify the device it's running on, but this is not good reason enough to give it access to browser profiles, browser history and most of all - user certificates for access to various places. For this reason the AppArmor profile for Skype is as much restrictive as it is currently possible (with Skype version 4.3.0.37). Unfortunately that's not always possible, since version 2.0+ Skype now refuses to start if it does not have access to /etc/group file. However most of the things it tries to access are already filtered, including access to video devices (audio is still allowed).  
This profile allows read and write access to ${HOME} directory only (subdirectories are not allowed) to be able to send files from there and receive sent files.
It also supports multiple Skype instances running different accounts with Skype profile names starting with ${HOME}/.Skype, like for example for a second instance:
```sh
skype --dbpath=~/.Skype2
```

usr.share.skypeforlinux.skypeforlinux
---

Note: skypeforlinux now curiously reads contents in /dev/disk/by-id and if it is unable to do so it just does not start properly. This could be considered harmful since it contains information about disks attached to the system with their serial numbers. I suppose this is done for the purpose of uniqely identifying a computer running skype, but then that's the real problem: I don't think Microsoft deserves to uniqely identify my computer(s) in any way. I don't run Windows because I don't trust them and I suppose soon I'll have to stop using skypeforlinux as well for the very same reasons - since skype is getting harmful (again).


opt.telegram.Telegram
---

Telegram profile expects the binary executable to be placed in /opt/telegram/Telegram, which makes it necessary to manually update when a new version is available.
