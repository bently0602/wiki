# Ubuntu - NGINX Install

## Install

```bash
apt install nginx certbot python3-certbot-nginx
```

## First Setup

### Auto renew certs cron

```bash
crontab -e
```

Add:

```
0 5 * * * /usr/bin/certbot renew --quiet
```

### nginx config

Generate dhparams

```bash
openssl dhparam -out /etc/nginx/dhparam.pem 4096.
```

Edit /etc/nginx/sites-enabled/default

```
server {
        listen 80;
        listen [::]:80;

        # listen 443 ssl;
        # listen [::]:443 ssl;

        server_name DNSADDRESS;
        server_tokens off;

        location / {
                proxy_pass              http://IPADDRESS:PORT;
                proxy_http_version      1.1;
                proxy_set_header        Upgrade $http_upgrade;
                # proxy_set_header Connection upgrade; # websockets
                proxy_set_header        Connection keep-alive;
                proxy_set_header        Host $host;
                proxy_cache_bypass      $http_upgrade;
                proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header        X-Forwarded-Proto $scheme;
                proxy_set_header        X-Forwarded-Path /;
                proxy_set_header        X-Forwarded-Prefix /;
        }

        add_header X-XSS-Protection             "1; mode=block";
        add_header X-Frame-Options              "SAMEORIGIN";
        add_header X-Content-Type-Options       nosniff;
        add_header Referrer-Policy              same-origin;

        # ssl_certificate         /etc/letsencrypt/live/DNSADDRESS/fullchain.pem; # managed by Certbot
        # ssl_certificate_key     /etc/letsencrypt/live/DNSADDRESS/privkey.pem; # managed by Certbot

        ssl_dhparam                     /etc/nginx/dhparam.pem;
        ssl_protocols                   TLSv1.2;
        ssl_prefer_server_ciphers       on;
        ssl_ciphers                     "EECDH+ECDSA+AESGCM EECDH+aRSA+AESGCM EECDH+ECDSA+SHA384 EECDH+ECDSA+SHA256 EECDH+aRSA+SHA384 EECDH+aRSA+SHA256 EECDH+aRSA+RC4 EECDH EDH+aRSA HIGH !RC4 !aNULL !eNULL !LOW !3DES !MD5 !EXP !PSK !SRP !DSS";
}
```

```bash
nginx -s reload
```

## Install certs 

```bash
certbot --register-unsafely-without-email --nginx -d DNSADDRESS
```

OR

```bash
certbot --register-unsafely-without-email -d DNSADDRESS
```

## Last Setup

Uncomment:

```
# listen 443 ssl;
# listen [::]:443 ssl;
# ssl_certificate
# ssl_certificate_key
```

```bash
nginx -s reload
```


## Basic Auth

```bash
apt install apache2-utils
htpasswd -c /etc/nginx/.htpasswd nginx
```

Add to nginx location section

```
location / {
        auth_basic "Private Property";
        auth_basic_user_file /etc/nginx/.htpasswd;
}
```

```bash
nginx -s reload
```
