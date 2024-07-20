# OpenBSD - Web Server

## Caddy

### Create Service User

Name = "_service" Home = "/home/service"
```
useradd -g =uid -c "I am a service that does stuff." -L daemon -s /sbin/nologin -d /home/service -m _service
```



## OpenBSD HTTPD
https://citizen428.net/blog/self-hosting-static-site-openbsd-httpd-relayd/
https://dev.to/nabbisen/setting-up-openbsds-httpd-web-server-4p9f
