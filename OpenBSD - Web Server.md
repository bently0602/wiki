# OpenBSD - Web Server

## Caddy

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

### Login as User

```
doas -u _caddy /bin/ksh -l
```

### Download Caddy

```
wget -q -O caddy https://caddyserver.com/api/download?os=openbsd&arch=amd64
chmod +x caddy
```

### Back as root

```
exit
```

### Caddyfile

/etc/Caddyfile

```
example.com {
        root * /var/www/htdocs/example.com
        encode gzip
        file_server
}
```

### PF

```
pass in on egress proto tcp from any to any port 80 rdr-to localhost
```

## OpenBSD HTTPD
https://citizen428.net/blog/self-hosting-static-site-openbsd-httpd-relayd/
https://dev.to/nabbisen/setting-up-openbsds-httpd-web-server-4p9f
