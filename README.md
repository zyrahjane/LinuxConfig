# Udacity - Linux server configuration project

## AWS info

IP address(publicIP): 54.206.123.68

Accessible SSH port: 2200.

Application URL(AppURL): ec2-54-252-251-201.ap-southeast-2.compute.amazonaws.com

## Steps Taken

### 1 - Amazon Lightsail setup

1. Networking tab in Firewall section 'Add Another' 1 for custom TCP with port number 2200 and UDP 123.
2. Save setting
3. In account details, SSH keys download .pem file

### 2 - Create a new user named *grader* and grant this user sudo permissions.

1. Log into the remote VM as *ubuntu* user through ssh: `$ ssh ubuntu@54.206.123.68 -p 22 -i LightsailDefaultPrivateKey-ap-southeast-2.pem`. (make sure that the pem file is in directory)
2. Add a new user called *grader*: `$ sudo adduser grader`.
3. Create a new file under the suoders directory: `$ sudo nano /etc/sudoers.d/grader`. Fill that newly created file with the following line of text: "grader ALL=(ALL:ALL) ALL", then save it.


### 3 - Update all currently installed packages

1. `$ sudo apt-get update`.
2. `$ sudo apt-get upgrade`.
3. Install *finger*. `sudo apt-get install finger`

### 4 - Configure the local timezone to UTC

1. Open time configuration dialog and set it to UTC with: `$ sudo dpkg-reconfigure tzdata`.
2. Install *ntp daemon ntpd* for a better synchronization of the server's time over the network connection: `$ sudo apt-get install ntp`.
3. Check with:`$ date`

Source: [UbuntuTime](https://help.ubuntu.com/community/UbuntuTime).

### 5 - Configure the key-based authentication for *grader* user

1. Generate an encryption key **on your local machine** with: `$ ssh-keygen -f ~/.ssh/udacity_key`.
2. Log into the remote VM as *root* user through ssh and create the following folder if not exists and file: `$ sudo mkdir /home/grader/.ssh`
`$ sudo nano /home/grader/.ssh/authorized_keys`.
3. Copy the content of the *udacity_key.pub* file from your local machine to the */home/grader/.ssh/authorized_keys* file you just created on the remote VM. Then change some permissions:
    1. `$ sudo chmod 700 /home/grader/.ssh`.
    2. `$ sudo chmod 644 /home/grader/.ssh/authorized_keys`.
    3. Finally change the owner from *root* to *grader*: `$ sudo chown -R grader:grader /home/grader/.ssh`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_key.rsa grader@54.206.123.68`.

### 6 - ssh_config changes
1. `$ sudo nano /etc/ssh/sshd_config`.
2. Find the *PasswordAuthentication* line and edit it to *no*.
3. Find the *PermitRootLogin* line and edit it to *no*.
4. `$ sudo service ssh restart`.

### 7 - Change the SSH port from 22 to 2200
1. `$ sudo nano /etc/ssh/sshd_config`.
2. Find the *Port* line and edit it to *2200*.
3. `$ sudo service ssh restart`.
4. Now you are able to log into the remote VM through ssh with the following command: `$ ssh -i ~/.ssh/udacity_key -p 2200 grader@54.206.123.68`.

Source: [Ubuntu forums](http://ubuntuforums.org/showthread.php?t=1739013).

### 8 - Configure the Uncomplicated Firewall (UFW)

Project requirements need the server to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).

1. `$ sudo ufw allow 2200/tcp`.
2. `$ sudo ufw allow 80/tcp`.
3. `$ sudo ufw allow 123/udp`.
4. `$ sudo ufw enable`.
5. check: `$ sudo ufw status`

### 9 - Configure firewall to monitor for repeated unsuccessful login attempts and ban attackers

Install *fail2ban* in order to mitigate brute force attacks by users and bots alike.
s
1. `$ sudo apt-get update`.
2. `$ sudo apt-get install fail2ban`.ß
3. We need the *sendmail* package to send the alerts to the admin user: `$ sudo apt-get install sendmail`.
4. Create a file to safely customize the *fail2ban* functionality: `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local` .
5. Open the *jail.local* and edit it: `$ sudo nano /etc/fail2ban/jail.local`. Set the *destemail* field to admin user's email address.

**Notes**: It doesn't make much sense to use *fail2ban* when the ssh key-based authentication is enforced. Though it is still useful for other things, like smtp/imap-logins.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04), [Reddit](https://www.reddit.com/r/linuxadmin/comments/2lravs/fail2ban_does_not_detect_my_ssh_privatekey/).

### 10 - Configure cron scripts to automatically manage package updates

1. Install *unattended-upgrades* if not already installed: `$ sudo apt-get install unattended-upgrades`.
2. To enable it, do: `$ sudo dpkg-reconfigure --priority=low unattended-upgrades`.

### 11 - Install Apache, mod_wsgi

1. `$ sudo apt-get install apache2`.
2. Mod_wsgi is an Apache HTTP server mod that enables Apache to serve Flask applications. Install *mod_wsgi* with the following command: `$ sudo apt-get install libapache2-mod-wsgi python-dev`.
3. Enable *mod_wsgi*: `$ sudo a2enmod wsgi`.
3. `$ sudo service apache2 start`.

### 12 - Install Git

1. `$ sudo apt-get install git`.
2. Configure your username: `$ git config --global user.name <username>`.
3. Configure your email: `$ git config --global user.email <email>`.

### 13 - Setup for deploying a Flask Application on Ubuntu VPS
Source: [DigitalOcean][20]

1. Extend Python with additional packages that enable Apache to serve Flask applications:
  `$ sudo apt-get install libapache2-mod-wsgi python-dev`
2. Enable mod_wsgi (if not already enabled):
  `$ sudo a2enmod wsgi`
3. `$ cd /var/www`. Then: `$ sudo mkdir catalog`.
4. Change owner for the *catalog* folder: `$ sudo chown -R grader:grader catalog`.


### 14 - Create Flask app

1. Create the file that will contain the flask application logic:
  `$ sudo nano __init__.py`
2. Paste in the following code:
```python
from flask import Flask
app = Flask(__name__)
@app.route("/")
def hello():
    return "Veni vidi vici!!"
if __name__ == "__main__":
    app.run()
```
### 15 - Install Flask
1. Install pip installer:
`$ sudo apt-get install python-pip`
2. Install virtualenv:
`$ sudo pip install virtualenv`
3. Set virtual environment to name 'venv':
`$ sudo virtualenv venv`
4. Enable all permissions for the new virtual environment (no sudo should be used within):
Source: [Stackoverflow][21]
`$ sudo chmod -R 777 venv`
5. Activate the virtual environment:
`$ source venv/bin/activate`
6. Install Flask inside the virtual environment:
`$ pip install Flask`
7. Run the app:
`$ python __init__.py`
8. Deactivate the environment:
`$ deactivate`

### 16 - Configure and Enable a New Virtual Host#
1. Create a virtual host config file
`$ sudo nano /etc/apache2/sites-available/catalog.conf`
2. Paste in the following lines of code and change names and addresses regarding your application:
```
<VirtualHost *:80>
    ServerName 54.206.123.68
    ServerAdmin admin@54.206.123.68
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

### 17 - Create the .wsgi File and Restart Apache
1. Create wsgi file:
`$ cd /var/www/catalog` and `$ sudo nano catalog.wsgi`
2. Paste in the following lines of code:

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/catalog/")
activate_this = '/var/www/catalog/venv/bin/activate_this.py'
execfile(activate_this, dict(__file__=activate_this))
from catalog import app as application
application.secret_key = 'Add your secret key'
```
3. Restart Apache:
`$ sudo service apache2 restart`

Sources: [Flask](http://flask.pocoo.org/docs/0.10/deploying/mod_wsgi/#working-with-virtual-environments)

### 18 - Clone the Catalog app from Github

1. Clone the catalog repository from Github:
`$ cd /var/www/catalog`
`$ git clone https://github.com/zyrahjane/FEND-Catalog.git`.
2. Move inside that newly created folder:
`$ mv -v /var/www/catalog/FEND-Catalog/* /var/www/catalog/catalog`
3. Remove git folder:
`$rm -rf FEND-Catalog/`

### 19 - Make the GitHub repository inaccessible:
  1. Create and open .htaccess file:
    `$ cd /var/www/catalog/` and `$ sudo nano .htaccess`
  2. Paste in the following:
    `RedirectMatch 404 /\.git`


### 20 - Install need packages and enable virtual host

1. Install all the other project's dependencies in the venv:
 `$ source /var/www/catalog/venv/bin/activate`
 `$ pip install bleach httplib2 requests oauth2client sqlalchemy psycopg2`.

Sources: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps), [Dabapps](http://www.dabapps.com/blog/introduction-to-pip-and-virtualenv-python/).


2. Enable the new virtual host:
`$ sudo a2ensite catalog`.

Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-run-django-with-mod_wsgi-and-apache-with-a-virtualenv-python-environment-on-a-debian-vps).

### 21 - Install and configure PostgreSQL

1. Install some necessary Python packages for working with PostgreSQL:
`$ sudo apt-get install libpq-dev python-dev`.
2. Install PostgreSQL:
`$ sudo apt-get install postgresql postgresql-contrib`.
3. Postgres is automatically creating a new user during its installation, whose name is 'postgres'. That is a tusted user who can access the database software. So let's change the user with:
`$ sudo su - postgres`,
then connect to the database system with:
 `$ psql`.
4. Create a new user called 'catalog' with his password:
`# CREATE USER catalog WITH PASSWORD 'password';`.
5. Give *catalog* user the CREATEDB capability:
`# ALTER USER catalog CREATEDB;`.
6. Create the 'catalog' database owned by *catalog* user:
`# CREATE DATABASE catalog WITH OWNER catalog;`.
7. Connect to the database: `# \c catalog`.
8. Revoke all rights:
`# REVOKE ALL ON SCHEMA public FROM public;`.
9. Lock down the permissions to only let *catalog* role create tables: `# GRANT ALL ON SCHEMA public TO catalog;`.
10. check:
`# \du`
11. Exit:
`# q`, then
`$ exit`

### 22 - Set up database
1. There are a few changes in the python files, using `nano` for lotsofcocktail.py and application.py update to this line:
`engine = create_engine('postgresql://catalog:password@localhost/catalog')`
2. Update json file paths in application.py by replacing `'XXXX.json'` with `r'/var/www/catalog/catalog/XXXX.json'`
3. Rename application.py:
  `$ mv application.py __init__.py`
4. run:
`python /var/www/catalog/catalog/database_setup_zb.py`, then
`python /var/www/catalog/catalog/lotsofcocktails.py`
4. check in sql
5. To prevent potential attacks from the outer world we double check that no remote connections to the database are allowed. Open the following file:
`$ sudo nano /etc/postgresql/9.5/main/pg_hba.conf` and edit it, if necessary, to make it look like this:
```
local   all             postgres                                peer
local   all             all                                     peer
host    all             all             127.0.0.1/32            md5
host    all             all             ::1/128                 md5
```
Source: [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).

### 23 - Install system monitor tools

1. `$ sudo apt-get update`.
2. `$ sudo apt-get install glances`.
3. To start this system monitor program just type this from the command line: `$ glances`.
4. Type `$ glances -h` to know more about this program's options.

Source: [eHowStuff](http://www.ehowstuff.com/how-to-install-and-use-glances-system-monitor-in-ubuntu/).

### 24 Update OAuth authorized JavaScript origins

1. To let users correctly log-in change the authorized URI to (http://54.206.123.68) on both Google (https://console.developers.google.com) and Facebook developer dashboards (https://developers.facebook.com/apps/).
2. Update `sudo nano /var/www/catalog/catalog/client_secrets.json`
making sure that the public IP address is in javascript_origins `"javascript_origins":["http://localhost:5000", "http://54.206.123.68"]`

### 25 - Restart Apache to launch the app
1. `$ sudo service apache2 restart`.
2. check in browser - http://54.206.123.68

#### Special thanks to [*stueken*](https://github.com/stueken) and [*iliketomatoes*](https://github.com/iliketomatoes) their README files were super helpful

