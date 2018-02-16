# Linux Server Setup

Instructions for deploying a flask application on lightsail

_Created in partial fulfillment of Udacity's Full Stack Web Developer Nanodegree_

## Server Information
- Public IP Address: 34.210.9.62
- SSH Port: 2200
- URL: http://ec2-34-210-9-62.us-west-2.compute.amazonaws.com

## Setup
1. Start a new Ubuntu Linux server instance on Amazon Lightsail.

2. SSH into server on terminal and update all installed packages.

```
sudo apt-get update
sudo apt-get upgrade
```

3. Change SSH Port in sshd_config file by changing Port 22 to Port 2200

`sudo nano /etc/ssh/sshd_config`

4. Set and enable firewall settings to allow for ports 80, 123, and 2200.

```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200
sudo ufw allow 80
sudo ufw allow 123
sudo ufw enable
sudo ufw status
```

5. Configure local timezone to UTC

`sudo dpkg-reconfigure tzdata`

6. Add a user with sudo permissions.

```
sudo add user [user]
sudo nano/etc/sudoers.d/[user]
```
In the file, write

`[user] ALL:(ALL) NOPASSWD:ALL`

7. On the local machine create a ssh keypair using ssh-keygen. You can view the public key using

`cat .ssh/[key].pub`

8. Configure key-based authentication for user.

```
mkdir .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys 
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```

The user should now be able to ssh in using the keypair on port 2200
`ssh [user]@34.210.9.62 -p 2200 -i ~/.ssh/[key].pub`

9. Install and configure Apache for a Python mod_wsgi application.

```
sudo apt-get install apache2 apache2-utils libexpat1 ssl-cert python
sudo apt-get install libapache2-mod-wsgi
```

10. Install PostgreSQL with a new database user named catalog with limited permissions to the catalog application. Check that remote connections are not allowed.

```
sudo apt-get install postgresql postgresql-contrib
sudo update-rc.d postgresql enable
sudo service postgresql start
sudo nano /etc/posgresql/9.5/main/pg_hba.conf

sudo su - postgres

psql
postgres=# CREATE DATABASE catalog;
postgres=# CREATE USER catalog;

postgres=# ALTER ROLE catalog WITH PASSWORD 'password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE catalog TO catalog;
postgres=# \q

exit
```

11. Install git.

`sudo apt-get install git`

12. Install virtual environment and necessary dependencies

```
sudo apt-get install python-sqlalchemy python-pip
sudo pip install virtualenv

sudo virtualenv venv

source venv/bin/activate
sudo chmod -$ 777 venv
sudo pip install psycopg2-binary
sudo pip install flask
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2

deactivate
```

13. Clone your project from git. 

`sudo git clone [git url]`

Add your public ip address and server url to authorized javascript origins in google developer console and replace the client_secrets.json file
In all the necessary python files, change sqlite to `postgresql://catalog:password@localhost/catalog`

14. Create a WSGI file inside your project directory 

```
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/CatalogApp")
from catalog import app as application
application.secret_key = 'Insert your application secret key'
```

15. Move the virtual environment into your project directory.

`sudo mv [old location] [new location]`

16. Configure and enable a new virtual host.

```
sudo cp /etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-available/catalog-app.conf
sudo nano  /etc/apache2/sites-available/catalog-app.conf
```

In the catalog-app.conf file, change the server name and add the following:

```
WSGIScriptAlias / /var/www/CatalogApp/catalog/catalogapp.wsgi
<Directory /var/www/CatalogApp/catalog/>
        Order allow,deny
        Allow from all
x</Directory>
        Alias /static /var/www/CatalogApp/catalog/static
<Directory /var/www/CatalogApp/catalog/static/>
        Order allow,deny
        Allow from all
</Directory>
```

Then enable virtual host.

```
sudo a2dissite 000-default.conf
sudo a2ensite catalog-app.conf
sudo service apache2 restart
```

17. To access error log, use 

`sudo tail /var/log/apache2/error.log`
