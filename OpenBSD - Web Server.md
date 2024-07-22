# OpenBSD - Web Server

## Caddy

## Modify privledged ports

/etc/sysctl.conf

```
net.inet.ip.porthifirst=0
net.inet.ip.porthilast=0
```

Reboot or run the below for immediate effect.

```
sysctl net.inet.ip.porthifirst=0
sysctl net.inet.ip.porthilast=0
```

### Create website folder

```
mkdir /var/www/htdocs/example.com
echo "<b>TEST</b>" >> /var/www/htdocs/example.com/index.html
```

### Create Service User

Name = "_caddy" Home = "/home/caddy"
```
useradd -g =uid -c "Caddy service user" -L daemon -s /sbin/nologin -d /home/caddy -m _caddy
```

### Download Caddy

```
wget -q -O /usr/local/sbin/caddy https://caddyserver.com/api/download?os=openbsd&arch=amd64
chmod +x /usr/local/sbin/caddy
```

### Caddyfile

/etc/caddyfile

```
{
	log default {
		output file /var/log/caddy
		format json
	}
}

https://example.com {
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
https://www.example.com {
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
```

### rc.d

/etc/rc.d/caddy

```
#!/bin/ksh

daemon="/usr/local/sbin/caddy"
daemon_user="_caddy"
daemon_flags="run --config /etc/caddyfile"

. /etc/rc.d/rc.subr

rc_bg=YES
rc_reload=NO

rc_cmd $1
```

```
chmod 0555 /etc/rc.d/caddy
rcctl -d enable caddy
rcctl -d start caddy
```

### OLD
Login as User

```
doas -u _caddy /bin/ksh -l
```

Download Caddy

```
wget -q -O caddy https://caddyserver.com/api/download?os=openbsd&arch=amd64
chmod +x caddy
```

Back as root

```
exit
```

PF

```
pass in on egress proto tcp from any to any port 80 rdr-to 127.0.0.1 port 8080
```

Caddyfile

/etc/Caddyfile

```
{
	log default {
		output stdout
		format json
		include http.log.access admin.api
	}
}

http://example.com:8080 {
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
http://www.example.com:8080 {
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
```

/etc/rc.d/caddy
```
#!/bin/ksh

daemon="/home/caddy/caddy"
daemon_user="_caddy"
daemon_flags="run --config /etc/Caddyfile"

. /etc/rc.d/rc.subr

rc_bg=YES
rc_reload=NO

rc_cmd $1
```

chmod 0555 /etc/rc.d/searxng
rcctl -d enable searxng
rcctl -d start searxng



## OpenBSD HTTPD
https://citizen428.net/blog/self-hosting-static-site-openbsd-httpd-relayd/
https://dev.to/nabbisen/setting-up-openbsds-httpd-web-server-4p9f
