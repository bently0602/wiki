# OpenBSD

## Add packages 
`pkg_add vim`

### Add ZSH and BASH
`pkg_add bash zsh`

#### Change Shell for User
```chsh -s bash```

Recommend you dont do this. Shells are installed in /usr/local/bin. KSH is in /bin

## Firewall 

### Base configuration

pfctl is the main firewall command.

1. Modify /etc/pf.conf 
2. Add: 
```
# Skip filtering on the loopback interface (lo0)
set skip on lo0
 
# Block all traffic by default
block all
 
# Allow incoming TCP traffic to port 22 (SSH), 80 (HTTP), 443 (HTTPS)
pass in proto tcp to port { 22 80 443 }

# Allow outgoing TCP and UDP traffic to ports 22 (SSH), 53 (DNS), 80 (HTTP), 123 (NTP), 443 (HTTPS)
pass out proto { tcp udp } to port { 22 53 80 123 443 }
 
# Allow outgoing ICMP traffic for echo requests (ping)
pass out inet proto icmp icmp-type { echoreq }
 
# Block and log outgoing TCP and UDP traffic for the user _pbuild, returning an error message
block return out log proto {tcp udp} user _pbuild
```

3. Restart PF

Reboot
OR

- Check firewall rules:
```
pfctl -vnf /etc/pf.conf
```

- Reload pf:
```
pfctl -f /etc/pf.conf
```

4. Check Rules 
```pfctl -s rules```
 
## Updating 

### Update Security Sources

#### Update Sources 
`syspatch –c`

#### Update security
`syspatch`

### Package and Package Dependency Updates
`pkg_add -uU`

### Configuration Merge Diff
`sysmerge -d`

### Typical Update
```
syspatch
pkg_add -uU
sysmerge -d
```

## Admin

### Users

#### Add User
```useradd -m -s /bin/sh username```

The account will not be able to be logged into with out a password.

-m gives a home directory

#### Change User Password
```passwd username```

### Enable DOAS

Edit /etc/doas.conf:

```
vim /etc/doas.conf
```

Add:

```

```

## Links

https://www.digitalocean.com/community/tutorials/how-to-configure-packet-filter-pf-on-freebsd-12-1

http://webcache.googleusercontent.com/search?q=cache:mTbhAKM1zTMJ:rtate.se/technology/bsd/guide/2016/06/20/SSHGuard-OpenBSD.html&cd=5&hl=en&ct=clnk&gl=us&client=safari

https://www.openbsd.org/faq/faq15.html

https://www.openbsdhandbook.com/openbsd_for_linux_users/

https://www.openbsdhandbook.com/system_management/updates/

https://docs.vultr.com/introduction-to-doas-on-openbsd


