Ubuntu - PostgreSQL Install
================================================

## Sources

locale settings in postgres
- https://askubuntu.com/questions/557124/how-to-specify-a-different-locale-for-postgresql-installation
changing data directory
- https://dba.stackexchange.com/questions/39999/how-to-change-the-data-directory-on-postgresql-9-1-on-ubuntu
- configuring postgres after install
http://suite.opengeo.org/docs/latest/dataadmin/pgGettingStarted/firstconnect.html

## Install Packages

```
apt install postgresql postgresql-contrib
```

## Remove Initial Cluster

Switch to the setup postgres user:
```
sudo -u postgres -i
```

Ubuntu comes with a set of commands that start with the letters pg_ when you install the package from ubuntu repos. These help wiht the administratioon of postgres and are handy. Use these whenever you can., The below instructions use them.

To see clusters installed by default:
```
pg_lsclusters
```

You'll probably want to remove the default cluster and setup your own:
```
pg_dropcluster --stop 10 main
```

Switch back to another user with sudo rights, then stop the service pertaining to that cluster just removed and reload the systemd service list:
```
sudo systemctl stop postgresql@10-main
sudo systemctl daemon-reload
```
Note the 10-main as the previous version-name of the old cluster we just removed. Every cluster runs as a service in Ubuntu using the tools.

## Create a Cluster
Create locale if needed (en_US.UTF-8):
```
locale-gen pt_BR.UTF-8
```

Give postgres user access to needed folder:
```
sudo chown -hR postgres:postgres /mnt/EncryptedTest
```

Create Cluster (note the paths, the version, then the name of the cluster):
```
pg_createcluster -d /mnt/EncryptedTest/pg/data -s /mnt/EncryptedTest/pg/sockets -l /mnt/EncryptedTest/pg/logs --locale en_US.UTF-8 10 prod
```

To start up the new cluster:
```
sudo systemctl daemon-reload
sudo systemctl start postgresql@10-prod.service
sudo systemctl status postgresql@10-prod.service
```

### Configuration

Config is at:
*ls /etc/postgresql/10/prod*
where "10" is postgres version and "prod" is cluster name.

This is just for a quick reference. See this for a short guide _http://suite.opengeo.org/docs/latest/dataadmin/pgGettingStarted/firstconnect.html_.
Note, I've had to look at the SSL settings for the cluster's config to see the generated self-signed SSL certificate so I could import it on another system to trust it.

#### Setting username/password for postgres

```
sudo -u postgres psql postgres
\password postgres
\q
```

#### Allowing External Access

In cluster directory modify pg_hba.conf and postgresql.conf.

Modify pg_hba.conf to something like this:

```
host    all             all             0.0.0.0/0            md5
host    all             all             ::0/0                md5
```

Add this to postgresql.conf

```
listen_addresses = '*'
```

Change these depending on network/firewall setup.
