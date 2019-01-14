# Linux Server Configuration
This is a part of Udacity Full Stack Web Development Nanodegree project.

# Project Description
It is to deploy the [*Item Catalog*](https://github.com/celevantemos/Item-Catalog) web application, which is a previous project on Udacity Full Stack Web Development, on a virtual machine and host it. Also, it is to install or configure web and database servers.

# Server Information
* Public ID address: 18.234.224.227
* Port: 2200
* Username: grader
* Domain Name: ec2-18-234-224-227.compute-1.amazonaws.com
* Web URL: http://18.234.224.227.xip.io

___

# Amazon Lightsail
* Go to [Amazon Lightsail](https://aws.amazon.com/lightsail) and click on get started.
* Go through the registration process if you do not have an account.
* In the lightsail, create an Ubuntu instance.
* Choose a plan
* Create a hostname
* It will take some time for the instance to run

# First: Connect to SSH from Amazon Lightsail
* Click on Connect to SSH
* Update available packages: `$ sudo apt-get update`
* Install all updated list to take affect: `$ sudo apt-get upgrade`
* other information from Amazon Lightsail:
  * public id: 18.234.45.41
  * SSH port: 2200 (default port is 22 so later we will change to 2200)

# Second: Log in to SSH using Terminal
* Download private key from Amazon Lightsail Ubuntu instance
* Rename the file downloaded to `pk.pem`
* Move the private key to `~/.ssh folder`
* change the permission on this key by `chmod 600 ~/.ssh/pk.pem`
* Now log in from terminal using `ssh -i ~/.ssh/pk.pem -p 22 ubuntu@18.234.45.41`
* Finally Update and install packages as mentioned above if not done

# Third: Change SSH Port
* Run `sudo nano /etc/ssh/sshd_config`
* Change the `Port` from `22` to `2200`
* Change `PasswordAuthentication` from `yes` to `no`
* Disable Remote connection by setting `PermitRootLogin` to `no`

# Fourth: Configure UFW
* Deny all incoming and allow all outgoing
* Only allow incoming connection from SSH on port 2200, HTTP port 80 and NTP on port 123
* Code to do all of the above:
```
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow 2200/tcp
sudo ufw allow 123/udp
sudo ufw allow www
sudo ufw allow enable
```
* Now update Amazon Firewall to allow HTTP, SSH with port 2200 and NTP.

# Fifth: Create a new user 'grader' with sudo access
* add the user grader `sudo adduser grader` and create the password and add other infoormation
* Now add this code `grader  ALL=(ALL:ALL) ALL` under User privilege specification by typing in terminal `sudo visudo`
* Then enter `sudo - grader` and then enter the password to verify sudo permissions.
* Now enter `sudo -l` and it will show you results but look for `(ALL : ALL) ALL` and if this is available you have sudo permission on grader user.
### Resources
* Digital Oceans [How to add and delete users on an ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps)

# Sixth: Configure key based authentication for the user **grader**
* On local machine generate encrypted key by `ssh-keygen`
* Save it on `~/.ssh` and name it `grader_key`
* Enter a passphrase or leave empty if no passphrase needed
* Now you will receive a message that your public key has been saved in grader_key.pub
* Copy the content of **grader_key.pub** by displaying the content by `cat grader_key.pub`
## On grader virtual machine
* Make a `.ssh` directory by `mkdir ~/.ssh`
* change permissions of this directory `chmod 700 ~/.ssh`
* Put the public key copied inside the directory under a file call **authorized_keys** by `sudo nano ~/.ssh/authorized_keys` and paste it then save.
* Change permissions on this file by `sudo chmod 644 ~/.ssh/authorized_keys`
* Restart the ssh by `sudo service ssh restart`
* Finally, you can login using `ssh -i ~/.ssh/grader_key -p 2200 grader@18.234.224.227`

# Seventh: Timezone Configuration
* On grader virtual machine run `sudo dpkg-reconfigure tzdata`
* Select `None` then `UTC`

# Eighth: Apache Configuration
* Install apache2 `sudo apt-get install apache2`
* Install mod-wsgi `sudo apt-get install python-setuptools libapache2-mod-wsgi`
* Restart services `sudo service apache2 restart`
* Open a browser and put the public IP and you should see the apache page.

# Ninth: Install python2 and Postgresql
* Install python2 `sudo apt-get install python`
* Install postgresql `sudo apt-get install postgresql python-psycopg2`
* Disable remote connection for postgresql by checking `sudo nano /etc/postgresql/9.5/main/pg_hba.conf` which is done by default.
   * Source: [Digital Ocean: How to secure postgresql against automated attacks](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-against-automated-attacks)

# Tenth: Configure PostgreSQL
* Log in to PSQL as default user postgres `sudo -u postgres psql`
* Create a user with password `CREATE USER catalog WITH PASSWORD 'catalog;`
* Give CREATEDB permission to the user `ALTER USER catalog CREATEDB;`
* Create a `catalog` database with `catalog` owner `CREATE DATABASE catalog WITH OWNER catalog;`
* Connect to user `catalog` by `\c catalog`
* Revoke all rights `REVOKE ALL ON SCHEMA public FROM public;`
* Grant all rights to catalog `GRANT ALL ON SCHEMA public TO catalog;`
* Exit psql `\q`

# Eleventh: Setup Catalog App Project
* Install git `sudo apt-get git`
* Go to directory **www** by `cd /var/www`
* Make a new directory called catalog `sudo mkdir catalog`
* Change owner of this directory `sudo chown -R grader:grader catalog`
* Go to directory catalog `cd catalog`
* Clone the [Item-Catalog](https://github.com/celevantemos/Item-Catalog) by `git clone https://github.com/celevantemos/Item-Catalog` in the new directory **catalog**
* Rename the directory `Item-Catalog` inside `catalog` to `catalog`
* Go to directory cloned catalog `cd catalog`
* Rename the file `application.py` to `__init__.py` by `mv application.py __init__.py`
* Modify `database_setup.py`, `categoriesItems.py` and `__init__.py` which are located in `/var/www/catalog/catalog/`
* The modification for those files are to replace the lines with `engine = create_engine('sqlite:///itemcatalog.db')` to `engine = create_engine('postgresql://catalog:catalog@localhost/catalog')`
* Also, modify `__init__.py` at the end by removing everything inside the `if` statement and adding only `app.run()`
* install pip `sudo apt-get install python-pip`
* Install the virtual environment `sudo virtualenv venv`
* Activate the virtual environment `source venv/bin/activate`
* Change permission `sudo chmod -R 777 venv`

* Install all requirements
```
sudo pip install Flask
sudo pip install --upgrade pip
sudo pip install httplib2 oauth2client sqlalchemy psycopg2 sqlalchemy_utils virtualenv requests psycopg2-binary Flask-HTTPAuth Flask-Login Flask-SQLAlchemy redis flask_httpauth
sudo apt-get install libapache2-mod-wsgi python-dev
sudo pip install --upgrade oauth2client
sudo apt-get install libpq-dev
```
* Run the application `sudo python __init__.py` and you should see this
```
Serving Flask app "__init__" (lazy loading)
Environment: production
  WARNING: Do not use the development server in a production environment.
  Use a production WSGI server instead.
Debug mode: off
Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
```
* deactivate the environment `deactivate`

## Create WSGI file
* Go to directory catalog `cd /var/www/catalog`
* add the code below in this file `sudo nano catalog.wsgi`.

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = "supersecretkey"
```

* restart apache `sudo service apache2 restart`
   * Rsource: [Digital Ocean: How to deploy a flask application](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)

## Configure and Enable a new Virtual Host
* Add the code below in this file we will create `sudo nano /etc/apache2/sites-available/catalog.conf`

```
<VirtualHost *:80>
    ServerName 18.234.224.227
    ServerAdmin grader@18.234.224.227
    ServerAlias ec2-18-234-224-227.compute-1.amazonaws.com
    WSGIScriptAlias / /var/www/catalog/catalog.wsgi
    <Directory /var/www/catalog/catalog/>
        Order allow,deny
        Allow from all
    </Directory>
    Alias /static /var/www/catalog/catalog/static
    <Directory /var/www/catalog/catalog/static/>
        Order allow,deny
        Allow from all
    </Directory>
    ErrorLog ${APACHE_LOG_DIR}/error.log
    LogLevel warn
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Enable virtual host by `sudo a2ensite catalog`
* Make sure to enable wsgi `sudo a2enmod wsgi`

## Populate database
* Go to directory catalog `cd /var/www/catalog/catalog`
* Activate the virtual environment `source venv/bin/activate`
* run the following: `sudo python database_setup.py` and `sudo python categoriesItems.py`
* Deactivate the virtual environment `deactivate`


## Make .git inaccessible by browser
* Create a file in `.git` and name it `.htaccess`
* The command `sudo nano /var/www/catalog/catalog/.git/.htaccess`
* Put this code inside:
```
Order allow,deny
Deny from all
```
* Save it and done

## Disable Apache's Default Site
* Disable the default site by `sudo a2dissite 000-default.conf`
* Reload apache `sudo service apache2 reload`
* Change ownership of project directories `sudo chown -R www-data:www-data catalog`
* Restart Apache `sudo service apache2 restart`
* Now you can open browser and check [http://18.234.224.227](http://18.234.224.227)

# Twelfth: Google OAuth setup
* Go to [Google Cloud Platform](https://console.cloud.google.com)
* Under APIs & Services -> Credentials -> Client
* Put `http://18.234.224.227.xip.io` in Authorized JavaScript origins
* Under Authorized redirect URIs put `http://18.234.224.227.xip.io/` and `http://18.234.224.227.xip.io/gconnect`
* Do no forget to update the JSON file after modification.
* Update Client Secret and Client ID if different in templates and application files.
* Now you can login using google account and add, delete and edit items you have created.

# Update the packages
* connect to ssh
* Run `sudo apt-get update`
* Then run `sudo apt-get dist-upgrade`

# Other Resources:
### Github repositories:
* [rrjoson](https://github.com/rrjoson/udacity-linux-server-configuration)
* [anumsh](https://github.com/anumsh/Linux-Server-Configuration)
