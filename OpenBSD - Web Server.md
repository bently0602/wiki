# OpenBSD - Web Server

## Caddy

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
wget -O caddy https://caddyserver.com/api/download?os=openbsd&arch=amd64
chmod +x caddy
```

## OpenBSD HTTPD
https://citizen428.net/blog/self-hosting-static-site-openbsd-httpd-relayd/
https://dev.to/nabbisen/setting-up-openbsds-httpd-web-server-4p9f
