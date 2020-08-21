Ubuntu - Install Jupyterlab Public Accessible
================================================

## Setup up A and AAAA records

Get your IP4 and IP6 addresses of your VPS and then setup appropriate A records in your DNS or domain provider. This will allow the automatic Letsencrypt features of Caddy to work.

Whenever [example.com] is in the below, replace with your domain name.

## Setup firewall

Allow SSH and HTTP/S.

```bash
ufw allow 22
ufw allow 80
ufw allow 443
ufw enable
```

## Install Anaconda

[Anaconda Install Page](https://www.anaconda.com/download/#linux)

I like to install in /opt/anaconda when it prompts if you want to override install location.

```bash
cd ~/
wget https://repo.anaconda.com/archive/Anaconda3-5.3.1-Linux-x86_64.sh
chmod +x Anaconda3-5.3.1-Linux-x86_64.sh
./Anaconda3-5.3.1-Linux-x86_64.sh
```

## Install Jupyterlab

```bash
conda install -c conda-forge jupyterlab
```

## Install Caddy

```bash
cd ~/
curl https://getcaddy.com | bash -s personal http.forwardproxy,http.jwt,http.login,http.ratelimit

# getcaddy.com script moves it to usr/local/bin for you
# cp ~/caddy /usr/local/bin

chown root:root /usr/local/bin/caddy
chmod 755 /usr/local/bin/caddy
```

### Allow non-admin access for Caddy to 80 and 443

```bash
setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy
```

## Create needed files and folders for Caddy

### Ensure groups and user of www-data exists

```bash
groupadd -g 33 www-data
useradd \
  -g www-data --no-user-group \
  --home-dir /var/www --no-create-home \
  --shell /usr/sbin/nologin \
  --system --uid 33 www-data
```

### Add config, ssl, and www folders

```bash
mkdir /etc/caddy
chown -R root:root /etc/caddy

mkdir /etc/ssl/caddy
chown -R root:www-data /etc/ssl/caddy
chmod 0770 /etc/ssl/caddy

mkdir -p /var/www
chown www-data:www-data /var/www
chmod 555 /var/www
```

### Create/Add any virtual directory folders, append to the /var/www folder

__from here [example.com] will be your domain you have mapped.__

```bash
mkdir -p /var/www/[example.com]
chown -R www-data:www-data /var/www/[example.com]
chmod -R 555 /var/www/[example.com]
```

## Create Caddy configuration file and give appropriate permissions

__jupyterlab should default to port 8888__

I took some from this [GIST](https://gist.github.com/cboettig/18e1becaa8974139adff)

__you made need to tweak the rate limit for your purposes (more than one user, etc...)__


/etc/caddy/Caddyfile Contents:
__replace username and password__

```
[example.com]

header /private Cache-Control "no-cache, no-store, must-revalidate"
log stdout

jwt {
        path /
        redirect /login
        allow sub username
}

login {
        success_url /
        simple username=password
        jwt_expiry 24h
        cookie_expiry 2400h
}

ratelimit * / 15 25 second

proxy / localhost:8888 {
        transparent
        websocket
        header_upstream X-Real-IP {remote}
        header_upstream Host {host}
}

```

### Ensures permissions on the Caddyfile:

```bash
chown root:root /etc/caddy/Caddyfile
chmod 644 /etc/caddy/Caddyfile

# confirm Caddyfile
cat /etc/caddy/Caddyfile
```

## Start screen session

### SCREEN 1 - Run caddy as www-data

```bash
su -s /bin/sh -l www-data --command 'ulimit -n 8192; CADDYPATH=/etc/ssl/caddy /usr/local/bin/caddy -log stdout -agree=true -conf=/etc/caddy/Caddyfile -root=/var/www/[example.com]'
```

### SCREEN 2 - Run jupyterlab

__Note:__ This disables authentication on the jupyter lab server relying on authentication setup with the Caddy JWT plugin.

```bash
jupyter lab --allow-root --NotebookApp.token='' --NotebookApp.password=''
```