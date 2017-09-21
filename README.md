Server Info

IP address: 13.126.84.42

SSH port: 2200

Web URL: http://13.126.84.42/

Software/Libraries Installed

These are the software/libraries that I have installed:

git
Apache
PostgreSQL
Flask
SQLAlchemy
PIP (for installing Python libraries)
oauth2client
psycopg2
requests
httplib2
Create a new user named grader

 sudo adduser grader
Give the grader user permission to sudo**

	 echo "grader ALL=(ALL) ALL" > /etc/sudoers.d/grader

Without "NOPASSWD" in the config, the user is prompted for password whenever sudo is used.
Set up key-based authentication for grader**

	#did this on my local vagrant machine
	 ssh-keygen

A private key (.rsa) and a public key (.pub) were created.

Then I got into the Udacity remote machine and copied contents of public key (.pub file) to `/home/grader/.ssh/authorized_keys`

	 sudo -i -u grader
	 mkdir .ssh
	 touch .ssh/authorized_keys
             nano .ssh/authorized_keys 
	(copied contents of .pub file into this file)
	 chmod 700 .ssh
	 chmod 644 .ssh/authorized_keys```
Disable remote SSH login as root

	#set this in /etc/ssh/sshd_config
	PermitRootLogin no
Disable password-based authentication for SSH

	#set this in /etc/ssh/sshd_config
	PasswordAuthentication no
Change the SSH port from 22 to 2200

	#set this in /etc/ssh/sshd_config
	Port 2200

With all the SSH config settings done, I restarted the SSH service:

	grader:~$ sudo service ssh restart
Will now need to use the following command to login to the server:

ssh -i ~/.ssh/udacity_key.rsa grader@13.126.84.42 -p 2200

Update all currently installed packages

sudo apt-get update - to update the package indexes

sudo apt-get upgrade - to actually upgrade the installed packages

Set-up SSH keys for user grader

As root user do:

mkdir /home/grader/.ssh
chown grader:grader /home/grader/.ssh
chmod 700 /home/grader/.ssh
cp /root/.ssh/authorized_keys /home/grader/.ssh/
chown grader:grader /home/grader/.ssh/authorized_keys
chmod 644 /home/grader/.ssh/authorized_keys
Can now login as the grader user using the command: ssh -i LightsailDefaultPrivateKey-ap-south-1.pem ubuntu@13.126.84.42 -p 2200

Change timezone to UTC

Check the timezone with the date command. This will display the current timezone after the time. If it's not UTC change it like this:

sudo timedatectl set-timezone UTC

Configuration Uncomplicated Firewall (UFW)

To check the status of the firewall, use:

sudo ufw status By default, block all incoming connections on all ports:

sudo ufw default deny incoming

Allow outgoing connection on all ports:

sudo ufw default allow outgoing

Allow incoming connection for SSH on port 2200:

sudo ufw allow 2200/tcp

Allow incoming connections for HTTP on port 80:

sudo ufw allow www

Allow incoming connection for NTP on port 123:

sudo ufw allow ntp

To check the rules that have been added before enabling the firewall use:

sudo ufw show added

To enable the firewall, use:

sudo ufw enable

To check the status of the firewall, use:

sudo ufw status

Install Apache to serve a Python mod_wsgi application

Install Apache:

sudo apt-get install apache2

Install the libapache2-mod-wsgi package:

sudo apt-get install libapache2-mod-wsgi

Install and configure PostgreSQL

Install PostgreSQL with:

sudo apt-get install postgresql postgresql-contrib

Create a PostgreSQL user called catalog with:

sudo -u postgres createuser -P catalog

You are prompted for a password. This creates a normal user that can't create databases, roles (users).

Create an empty database called catalog with:

sudo -u postgres createdb -O catalog catalog

Install Flask, SQLAlchemy, etc

Issue the following commands:

sudo apt-get install python-psycopg2 python-flask
sudo apt-get install python-sqlalchemy python-pip
sudo pip install oauth2client
sudo pip install requests
sudo pip install httplib2
sudo pip install flask-seasurf
Install Git version control software

sudo apt-get install git

I cloned the Item Catalog project using git:

	grader$ sudo mkdir -p /var/www/itemcatalog
	grader$ sudo git clone https://github.com/tyagiboy1996/ITEM-CATALOG-.git /var/www/itemcatalog
Protected .git folder:

	grader$ sudo chmod 700 /var/www/itemcatalog/.git
I added http://13.126.84.42/ to Authorized JavaScript Origins in the project in Google Developer Console, then downloaded the JSON file. Copied the JSON file contents to a new file /var/www/itemcatalog/client_secrets.json.

Installed additional packages and libraries necessary to host this application:

	grader:~$ sudo apt-get install python-psycopg2
	grader:~$ sudo apt-get install python-flask
	grader:~$ sudo apt-get install python-sqlalchemy
	grader:~$ sudo apt-get install python-pip
	grader:~$ sudo pip install oauth2client
	grader:~$ sudo pip install requests
	grader:~$ sudo pip install httplib2
Changed the project to use PostgreSQL database (it was using SQLite originally):

Changed all references to SQLite database to reference PostgreSQL database:

#change argument of all create_engine() calls to use postgresql instead of sqlite `engine = create_engine('postgresql://catalog:grader@localhost/catalog')

  The above step was done for
  * `/var/www/itemcatalog/app.py`
  * `/var/www/itemcatalog/database_init.py`
  * `/var/www/itemcatalog/database_setup.py`
Setup the new PostgreSQL database catalog and populate it with some test data

   python /var/www/itemcatalog/database_setup.py
#import Flask's app as the WSGI-required application object from application import app as application

Edited `/etc/apache2/sites-available/000-default.conf`:

	

<VirtualHost *:80>
    # The ServerName directive sets the request scheme, hostname and port that
    # the server uses to identify itself. This is used when creating
    # redirection URLs. In the context of virtual hosts, the ServerName
    # specifies what hostname must appear in the request's Host: header to
    # match this virtual host. For the default virtual host (this file) this
    # value is not decisive as it is used as a last resort host regardless.
    # However, you must set it for any further virtual host explicitly.
    #ServerName www.example.com

    ServerAdmin webmaster@localhost
    #DocumentRoot /var/www/html
    WSGIDaemonProcess catalog user=www-data group=www-data threads=5
    WSGIScriptAlias / /var/www/item-catalog/catalog.wsgi
    <Directory /var/www/item-catalog>
    WSGIProcessGroup catalog
    WSGIApplicationGroup %{GLOBAL}
    Order deny,allow
    Allow from all
    </Directory>

    Alias /static /var/www/item-catalog/static
    <Directory /var/www/item-catalog/static>
    Require all granted
    </Directory>
    # Available loglevels: trace8, ..., trace1, debug, info, notice,
    #warn, error, crit, alert, emerg. It is also possible to
    #configure the loglevel for particular modules, e.g. LogLevel
    #info ssl:warn


    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined

    # For most configuration files from conf-available/, which are
    # enabled or disabled at a global level, it is possible to
    # include a line for only one particular virtual host. For example the
    # following line enables the CGI configuration for this host only
    # after it has been globally disabled with "a2disconf".
    #Include conf-available/serve-cgi-bin.conf
After changes were made to the config files, I restarted the Apache server:

	grader:~$ sudo apache2ctl restart 

During this whole process, if any error pages were served, I would check:

	grader:~$ sudo cat /var/log/apache2/error.log

I also added a logging system to my app.py file so that Flask exceptions get logged (if not, Flask exceptions will just show up as 500 Internal Server Error with nothing showing up in Apache's error log!)

Created the log file first for `catalog` user:

	grader:~$ sudo mkdir /var/log/itemcatalog
	grader:~$ sudo touch /var/log/itemcatalog/error.log
	grader:~$ sudo chown catalog:catalog /var/log/itemcatalog/error.log
	grader:~$ sudo chmod 640 /var/log/itemcatalog/error.log

With all these done, the server is up and running, and the application can be viewed at ('http://13.126.84.42/')
## Update catalog.wsgi file for this installation
Absolute paths are updated to where the catalog is located. The application `secret_key` is set to
something random and the PostgreSQL for the `muskan` user is set. In this case, the file


## Update the Google OAuth client secrets file
Fill in the `client_id` and `client_secret` fields in the file `client_secrets.json`.
Also change the `javascript_origins` field to the IP address and AWS assigned URL of the host.
In this instance that would be:
`"javascript_origins":["http://13.126.84.42"]`

These addresses also need to be entered into the Google Developers Console -> API Manager
-> Credentials, in the web client under "Authorized JavaScript origins".


The catalog app should now be available at `http://13.126.84.42` 

