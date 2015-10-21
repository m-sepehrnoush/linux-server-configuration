# Linux-based Server Configuration

The website is hosted and fully functional at the following IP address:
52.88.110.111

## Connecting using SSH:
In order to connect to server using ssh enter the following command:
```bash
ssh -i ~/.ssh/udacity_key.rsa grader@52.88.110.111 -p 2200
```
udacity_key.rsa being your privatekey(Not provided here)

### Configuration steps:

 * Launched my Virtual Machine with my Udacity account.
 * Made a .ssh directory in my local home directory and copied the provided privatekey.
 * Changed access level of the file to 600 using chmod
 * SSHed to the server using following command:
 ```bash
 ssh -i ~/.ssh/udacity_key/rsa root@52.88.110.111
 ```
 * Made user **grader**
 * User **grader** was given sudo access by adding and modifying /etc/sudoers.d/grader
 * Added publickey to **grader**'s *~/.ssh/authorized_keys*
 * In order to restrict access to *.ssh* directory and files for other users:
 ```bash
chmod 700 .ssh
 chmod 644 .ssh/authorized_keys
 ```
 * By editing */etc/ssh/sshd_config:
	- Key-based SSH authentication is enforced
	- SSH is hosted on a non-default port (2200)
	- Remote login of the root user has been disabled
 * In order for changes to take effect:
 ```bash
 sudo service ssh restart
 ```
 * Updated all currently installed packages:
 ```bash
sudo apt-get update
sudo apt-get upgrade
```
 * Time was already set on UTC but can be reconfigured using:
 ```bash
 dpkg-reconfigure tzdata
 ```
 and following the instructions.
 * Check the current status of firewall using:
 ```bash
 sudo ufw status
 ```
 * Next, all outgoing traffic was allowed and all incoming blocked:
 ```bash
sudo ufw default allow outgoing
 sudo ufw default deny incoming
 ```
 * Except the following ports for *SSH*, *HTTP* and *NTP* respectively:
 ```bash
sudo ufw allow 2200/tcp
 sudo ufw allow 80/tcp
 sudo ufw allow 123/udp
 ```
 * Enabled the firewall using command *enable*
 * Installed Apache web server with:
 ```bash
 sudo apt-get install apache2
 ```
 * Confirmed it is up & running by visiting *52.88.110.111*
 * Installed the following packages using apt-get
	- libapache2_mod_wsgi
	- python2.7-dev
	- postgresql
	- python-psycopg2
	- python-pip
 * Enabled *mod_wsgi* using **a2enmod**
 * Remote connections were already restricted in:
	*/etc/postgresql/9.3/main/pg_hba.conf*
 * Created a new database and a new user with access to said database with following commands:
 ```bash
 sudo su postgres
 psql
 ```
 ```sql
CREATE DATABASE catalogdb;
 CREATE USER catalog;
 ALTER ROLE catalog WITH PASSWORD 'lilo20';
 GRANT ALL PRIVILEGES ON DATABASE catalogdb TO catalog;
 \q
 ```
 ```bash
 exit
 ```
 *Installed and cloned git repository with following commands:
 ```bash
sudo apt-get install git
cd /var/www/
git clone https://github.com/m-sepehrnoush/bookexchangeprogram.git
```
 * Renamed *bookexchangeprogram* to *bep*
 * After changing directory to *bep* and installed following packages using pip:
 * Installed following packages using pip:
	- virtualenv
	- Flask==0.9
 * Setup and activated a virtual development environment using:
 ```bash
sudo virtualenv /tempenv
 source /tempenv/bin/activate
 ```
 * Renamed *project.py* file to *__init__.py
 * Edited all files to reflect the changes in database file:
	*sqlite://library.db* changed to *postgresql://catalog:lilo20@localhost/catalogdb*
 * Installed remaining dependencies using pip:	
	- SQLAlchemy==0.8.4
	- werkzeug==0.8.3
	- Flask-Login==0.1.3
	- google_api_python_client==1.4.1
 * To create db file and check if everything works:
 ```bash
sudo python database_setup.py
 sudo python __init__.py
 ```
 * *deactivate*ed tempenv
 * Configured a new Virtual Host with creating and modifying:
 ```bash
 sudo nano /etc/apache2/sites-available/bep.conf
 ```
 and added the following to bep.conf:
 ```bash
<VirtualHost *:80>
		ServerName 52.88.110.111
		WSGIScriptAlias / /var/www/bep/bep.wsgi
		<Directory /var/www/bep/bep/>
			Order allow,deny
			Allow from all
		</Directory>
		Alias /static /var/www/bep/bep/static
		<Directory /var/www/bep/bep/static/>
			Order allow,deny
			Allow from all
		</Directory>
		ErrorLog ${APACHE_LOG_DIR}/error.log
		LogLevel warn
		CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
 * And enabled it using:
 ```bash
sudo a2ensite bep
```
 * Changed directory to */var/www/bep* and created **bep.wsgi** file
 * Added the following to *bep.wsgi*:
 ```bash
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/bep/")

from bep import app as application
application.secret_key = 'super_secret_key'
```
 * Specified absolute path for every mention of CLIENT_ID file for avoiding [client-secret.json not found] error (https://discussions.udacity.com/t/client-secret-json-not-found-error/34070)
 * Added *52.88.110.111* to *Authorized JavaScript origins* in google's developers console


####IMPORTANT NOTES
Private key should never be on the server. Private key must reside in *./ssh* folder of your local home directory. Public key, recognizable by *.pub* extension,
must reside in *./ssh* folder of remote user and in the authorized_keys file.
Keep in mind that the remote user must have appropriate access level to their own *./ssh* folder and *authorized_keys* file. This can be achieved using following commands respectively:
```bash
chmod 700
chmod 644
```
and making sure that remote user is in fact the owner of *.ssh* and *authorized_keys*. In order to check ownership:
```bash
ls -al
```
to change owner:
```bash
chown [username] [folder]
```
to change group:
```bash
chgrp [username] [folder]
```

##### Stuff used to make this:

 * [Markdown Basics](https://help.github.com/articles/markdown-basics/)
 * [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
 * [createuser](http://www.postgresql.org/docs/9.1/static/app-createuser.html)
 * [The pg_hba.conf File](http://www.postgresql.org/docs/9.1/static/auth-pg-hba-conf.html)
 * [CREATE DATABASE](http://www.postgresql.org/docs/9.1/static/sql-createdatabase.html)
 * [use PostgreSQL with Flask or Django](http://killtheyak.com/use-postgresql-with-django-flask/)
 * [Markedly underwhelming and potentially wrong resource list for P5](https://discussions.udacity.com/t/markedly-underwhelming-and-potentially-wrong-resource-list-for-p5/8587)
 
