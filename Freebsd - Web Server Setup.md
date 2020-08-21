FreeBSD - Web Server Setup
================================================

_https://www.digitalocean.com/community/tutorials/recommended-steps-for-new-freebsd-12-0-servers_

_https://forums.freebsd.org/threads/bind-port-under-1024.53434/_

_https://forums.freebsd.org/threads/creating-a-system-user-like-www.2152/_

## Software

### Updating OS

```bash
sudo freebsd-update fetch install
```

The freebsd-update command is the management utility for software in the base operating system. 
The fetch subcommand downloads any new updates, while the install subcommand applies them to the live system.

### Prebuilt Packages

```bash
sudo pkg update
```

Installing...
```bash
sudo pkg install tmux
```

## Firewall

User either sysrc or edit /etc/rc.conf directly.

```bash
sudo sysrc firewall_enable="YES"
...
```

```bash
sudo vi /etc/rc.conf
```

and add:

```
firewall_enable="YES"
firewall_quiet="YES"
firewall_type="workstation"
firewall_myservices="22/tcp 80/tcp 443/tcp"
firewall_allowservices="any"
firewall_logdeny="NO"
```

then finally start ipfw

```bash
sudo service ipfw start
```

## Time setup

```bash
sudo tzsetup
sudo sysrc ntpd_enable="YES"
sudo sysrc ntpd_sync_on_start="YES"
sudo service ntpd start
```

## Hacks

### Allowing binding to ports under 1024 with unprivledged users

Add:

net.inet.ip.portrange.reservedhigh=0

To:

/etc/sysctl.conf

## Web Server Setup

### Add wwwmain user

Add the group and the user. The user will not be able to login. Can go around this by sudo however. 

```bash
pw addgroup -n wwwmain
pw adduser wwwmain -g wwwmain -d /home/wwwmain -s /usr/sbin/nologin -c "web server user"
pw usershow -P -a | grep groupname
```

### Create Caddy configuration

Make path to serve files from
```bash
sudo -u wwwmain mkdir -p /home/wwwmain/wwwroot/crap.dev
```

Create caddyfile. This serves from the previous directory, translates the markdown into html on the fly, and serves up a webdav share of that direxotry for easy access. make sure to modify the passwords.

```bash
sudo -u wwwmain vim /home/wwwmain/Caddyfile
```

```bash
www.crap.dev, crap.dev {
        gzip
        root /home/wwwmain/wwwroot/crap.dev
        internal /.git

        ext .html .htm .md

        basicauth /webdav webdav_user PASSWORD
        webdav /webdav {
                scope /home/wwwmain/wwwroot/crap.dev

        }

        markdown /
}
```

## Start Caddy

Start shell in tmux/screen. Shell is needed to repoint home directory. Caddy uses the running users shell home directory to store ssl certs, cache, etc.

```bash
sudo -u ws /bin/sh
```

Then start caddy
```bash
cd ~/
./caddy
```
