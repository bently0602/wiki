Ubuntu - Caddy Running as systemd Service
================================================

## Instructions

We will assume the following:

* that you want to run caddy as user `www-data` and group `www-data`, with UID and GID 33
* you are working from a non-root user account that can use 'sudo' to execute commands as root

Adjust as necessary or according to your preferences.

First, either pipe the install script from the web into bash:

```bash
curl https://getcaddy.com | bash
OR
curl https://getcaddy.com | bash -s personal
```

OR 
put the caddy binary in the system wide binary directory and give it
appropriate ownership and permissions:

```bash
sudo cp /path/to/caddy /usr/local/bin
```

Then setup permission on the binary:
```bash
sudo chown root:root /usr/local/bin/caddy
sudo chmod 755 /usr/local/bin/caddy
```

Give the caddy binary the ability to bind to privileged ports (e.g. 80, 443) as a non-root user:

```bash
sudo setcap 'cap_net_bind_service=+ep' /usr/local/bin/caddy
```

Set up the user, group, and directories that will be needed:

```bash
sudo groupadd -g 33 www-data
sudo useradd \
  -g www-data --no-user-group \
  --home-dir /var/www --no-create-home \
  --shell /usr/sbin/nologin \
  --system --uid 33 www-data

sudo mkdir /etc/caddy
sudo chown -R root:root /etc/caddy
sudo mkdir /etc/ssl/caddy
sudo chown -R root:www-data /etc/ssl/caddy
sudo chmod 0770 /etc/ssl/caddy
```

Optionally add root user to www-data group:

```
usermod -a -G www-data root
```

Create your caddyfile:

```bash
echo "root /var/www" > /etc/caddy/Caddyfile
```

OR

Place your caddy configuration file ("Caddyfile") in the proper directory

```bash
sudo cp /path/to/Caddyfile /etc/caddy/
```

Then give it appropriate ownership and permissions:

```bash
sudo chown root:root /etc/caddy/Caddyfile
sudo chmod 644 /etc/caddy/Caddyfile
```

Create the home directory for the server and give it appropriate ownership
and permissions:

```bash
sudo mkdir /var/www
sudo chown www-data:www-data /var/www
sudo chmod 555 /var/www
```

Let's assume you have the contents of your website in a directory called 'example.com'.
Put your website into place for it to be served by caddy:

```bash
sudo cp -R example.com /var/www/
sudo chown -R www-data:www-data /var/www/example.com
sudo chmod -R 555 /var/www/example.com
```

You'll need to explicitly configure caddy to serve the site from this location by adding
the following to your Caddyfile if you haven't already:

```
example.com {
    root /var/www/example.com
    ...
}
```

Install the systemd service unit configuration file, reload the systemd daemon,
and start caddy:

```bash
wget https://raw.githubusercontent.com/mholt/caddy/master/dist/init/linux-systemd/caddy.service
sudo cp caddy.service /etc/systemd/system/
sudo chown root:root /etc/systemd/system/caddy.service
sudo chmod 644 /etc/systemd/system/caddy.service
sudo systemctl daemon-reload
sudo systemctl start caddy.service
```

Have the caddy service start automatically on boot if you like:

```bash
sudo systemctl enable caddy.service
```

If caddy doesn't seem to start properly you can view the log data to help figure out what the problem is:

```bash
journalctl --boot -u caddy.service
```

Use `log stdout` and `errors stderr` in your Caddyfile to fully utilize systemd journaling.

If your GNU/Linux distribution does not use *journald* with *systemd* then check any logfiles in `/var/log`.

If you want to follow the latest logs from caddy you can do so like this:

```bash
journalctl -f -u caddy.service
```

You can make other certificates and private key files accessible to the `www-data` user with the following command:

```bash
setfacl -m user:www-data:r-- /etc/ssl/private/my.key
```

### caddy.service
```
[Unit]
Description=Caddy HTTP/2 web server
Documentation=https://caddyserver.com/docs
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service

[Service]
Restart=on-abnormal

; User and group the process will run as.
User=www-data
Group=www-data

; Letsencrypt-issued certificates will be written to this directory.
Environment=CADDYPATH=/etc/ssl/caddy

; Always set "-root" to something safe in case it gets forgotten in the Caddyfile.
ExecStart=/usr/local/bin/caddy -log stdout -agree=true -conf=/etc/caddy/Caddyfile -root=/var/tmp
ExecReload=/bin/kill -USR1 $MAINPID

; Use graceful shutdown with a reasonable timeout
KillMode=mixed
KillSignal=SIGQUIT
TimeoutStopSec=5s

; Limit the number of file descriptors; see `man systemd.exec` for more limit settings.
LimitNOFILE=1048576
; Unmodified caddy is not expected to use more than that.
LimitNPROC=512

; Use private /tmp and /var/tmp, which are discarded after caddy stops.
PrivateTmp=true
; Use a minimal /dev (May bring additional security if switched to 'true', but it may not work on Raspberry Pi's or other devices, so it has been disabled in this dist.)
PrivateDevices=false
; Hide /home, /root, and /run/user. Nobody will steal your SSH-keys.
ProtectHome=true
; Make /usr, /boot, /etc and possibly some more folders read-only.
ProtectSystem=full
; … except /etc/ssl/caddy, because we want Letsencrypt-certificates there.
;   This merely retains r/w access rights, it does not add any new. Must still be writable on the host!
ReadWriteDirectories=/etc/ssl/caddy

; The following additional security directives only work with systemd v229 or later.
; They further restrict privileges that can be gained by caddy. Uncomment if you like.
; Note that you may have to add capabilities required by any plugins in use.
;CapabilityBoundingSet=CAP_NET_BIND_SERVICE
;AmbientCapabilities=CAP_NET_BIND_SERVICE
;NoNewPrivileges=true

[Install]
WantedBy=multi-user.target
```