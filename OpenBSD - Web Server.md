# OpenBSD - Web Server

## Caddy

### Create website folder

```
mkdir /var/www/htdocs/{example.com}
echo "<b>TEST</b>" >> /var/www/htdocs/{example.com}/index.html
```

### Create Service User

Name = "_caddy" Home = "/home/caddy"
```
useradd -g =uid -c "Caddy service user" -L daemon -s /sbin/nologin -d /home/caddy -m _caddy
```

If you ever need to login as that user from root to test something then:

```
doas -u _caddy /bin/ksh -l
```

### PF Config

```
# Skip filtering on the loopback interface (lo0)
set skip on lo0

# Block all traffic by default
block all

# rdr pass on "vio0"  proto tcp from any to any port 80 -> 127.0.0.1 port 80
pass in quick proto { tcp udp } to port 80 rdr-to 127.0.0.1 port 8080
pass in quick proto { tcp udp } to port 443 rdr-to 127.0.0.1 port 8443
pass out proto { tcp udp } from 127.0.0.1 to any port 80
pass out proto { tcp udp } from 127.0.0.1 to any port 443

# Allow incoming TCP traffic to port 22 (SSH), 80 (HTTP), 443 (HTTPS)
pass in proto { tcp udp } to port { 22 80 443 }

# Allow outgoing TCP and UDP traffic to ports 22 (SSH), 53 (DNS), 80 (HTTP), 123 (NTP), 443 (HTTPS)
pass out proto { tcp udp } to port { 22 53 80 123 443 }

# Allow outgoing ICMP traffic for echo requests (ping)
pass out inet proto icmp icmp-type { echoreq }

# Block and log outgoing TCP and UDP traffic for the user _pbuild, returning an error message
block return out log proto {tcp udp} user _pbuild
```

```
pfctl -f /etc/pf.conf
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
		output file /var/log/caddy/current.log
		format json
		level ERROR
	}

        https_port 8443
        http_port 8080

	admin off
}

http://example.com:8080 {
        bind 127.0.0.1
        header Content-Type text/html
        respond <<HTML
                <html>
                        <head><title>Foo</title></head>
                        <body>Foo</body>
                </html>
                HTML 200
}

https://example.com:8443 {
	bind 127.0.0.1
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
https://www.example.com:8443 {
	bind 127.0.0.1
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
```

### Logging

```
mkdir -p /var/log/caddy
chown -R _caddy:_caddy /var/log/caddy
chmod -R 660 /var/log/caddy
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
