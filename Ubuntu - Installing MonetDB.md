Installing and Initial Set Up of MonetDB
================================================

## Installing

> Taken from: https://www.monetdb.org/downloads/deb/

Create a file _/etc/apt/sources.list.d/monetdb.list_ with the following contents (replacing "suite" with your system's code name which you may be able to get by running the command lsb_release -cs, ex. xenial for 16.04, bionic for 18.04):

```bash
deb https://dev.monetdb.org/downloads/deb/ bionic monetdb
deb-src https://dev.monetdb.org/downloads/deb/ bionic monetdb
```

Then issue the following command to install the MonetDB GPG public key:

```bash
wget --output-document=- https://www.monetdb.org/downloads/MonetDB-GPG-KEY | sudo apt-key add -
```

After this, you can use apt to install the MonetDB packages. First run:

```bash
sudo apt update
```

To install MonetDB/SQL, use the command:

```bash
sudo apt install monetdb5-sql monetdb-client
```

If you wish (and your system uses it), you can have systemd manage the MonetDB service. If you do, the databases will be in _/var/monetdb5/dbfarm_:

```bash
sudo systemctl enable monetdbd
sudo systemctl start monetdbd
```

Add any users who are allowed to run a database server to the monetdb group, which was automatically created in the previous step, replace [USER] with your username:

```bash
sudo usermod -a -G monetdb [USER]
```

Then log out and back in again to activate this change. After this you can run monetdbd or mserver5.

## Setup

Make sure systemd service is up:

```bash
sudo systemctl start monetdbd
netstat -tulpn
```

Make monetdb listen externally:

```bash
monetdbd set listenaddr=0.0.0.0 /var/monetdb5/dbfarm
```

Create a DB in the farm:

```bash
monetdb create monet
monetdb release monet
```

Set a new password for default "monetdb" user:
```bash
mclient -u monetdb -d monet
     ALTER USER SET UNENCRYPTED PASSWORD 'newpassword' USING OLD PASSWORD 'monetdb';
```

Restart MonetDB:

```bash
sudo systemctl stop monetdbd
sudo systemctl start monetdbd
```
