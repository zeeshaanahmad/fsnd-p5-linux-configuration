# Project 5 - Linux server configuration
This project is submission for P5: Linux Server Configuration

# Description
This project requires to configure and secure a Linux virtual machine on Amazon AWS instance.

# How to access?
### Public IP Address
`52.25.36.89`
### SSH Port
`2200`

# Hosted Application
`Item Catalog` - [`http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/`][1]

# Screenshots
![Item Catalog Home](http://i.imgur.com/J8mUH2d.png)
**Homepage**

![Imgur](http://i.imgur.com/odppUxu.png)
**Items**

# Step by Step
## 1. Connect to Linux VM
  * Open terminal
  * Enter following command
  ```
  ssh -i ~/.ssh/udacity_key.rsa root@52.25.36.89
  ```
  NOTE: Use the public key for root which is provided by Udacity. Key was downloaded and stored in `~/.ssh`.

## 2. User Management
  *Source: [Udacity > Configuring Linux Web Servers > Lession 2: Linux Security](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089468)*
1. Create user named `grader`
```
adduser grader
```
1. Perform these steps to grant sudo permissions to grader user.
  1. Create a file with user's name in following location `/etc/sudoers.d` i.e;
  ```
  sudo nano /etc/sudoers.d/grader
  ```
  1. Add the following lines to this file
  ```
  grader    ALL=(ALL) NOPASSWD:ALL
  ```

## 3. Security
  1. Enforce Key based authentication

    *Source: [Udacity > Configuring Linux Web Servers > Lession 2: Linux Security](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089477)*
    * Open another terminal on your local machine
    * Generate key pairs on local machine
    ```
    ssh-keygen
    ```
    * Provide the filename (e.g. `fsnd-grader`) and location to store the generated keys along with passphrase.
    * Switch to terminal connected to Linux VM in Step 1 through root user
    * Change user to grader
    ```
    su grader
    ```
    * Go to grader home directory
    ```
    cd /home/grader
    ```
    * Create a folder name `.ssh`
    ```
    mkdir .ssh
    ```
    * Go in to `.ssh`
    ```
    cd .ssh
    ```
    * Create a file name `authorized_keys`
    ```
    touch authorized_keys
    ```
    * Copy the contents of `fsnd-grader.pub` from local machine
    * Open the ~/.ssh/authorized_keys on remote VM and paste the copied key contents
    ```
    sudo nano ~/.ssh/authorized_keys
    ```
    * Save and exit the nano editor
    * Change permissions on the `.ssh` and `.ssh/authorized_keys` using following commmands
    ```
    sudo chmod 700 ~/.ssh
    sudo chmod 644 ~/.ssh/authorized_keys
    ```
    * Close the connection to remote VM to retry using `grader` user
    * Connect to remote Linux VM using following command; It uses the generated key file which is stored on local machine
    ```
    ssh grader@52.25.36.89 -i ~/.ssh/fsnd-grader
    ```
    * It should connect to remote Linux VM without requiring password using the key based authentication
    * Open `/etc/sshd_config` file to set `PasswordAuthentication` entry to `no`. This will enforce key based authentication.
    * Restart the SSH service to make the changes effective
    ```
    sudo service ssh restart
    ```

  2. Configure UFW

    *Source: [Udacity > Configuring Linux Web Servers > Lession 2: Linux Security > Configuring Ports in UFW](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089499)*
    * First check the status of firewall if it is already active
    ```
    sudo ufw status
    ```
    * As of now, the status should be inactive
    ```
    Status: inactive
    ```
    * Start configuring the firewall
    * Set to block all incoming traffic by default
    ```
    sudo ufw default deny incoming
    ```
    * Set to allow outgoing traffic
    ```
    sudo ufw default allow outgoing
    ```
    * Now start allowing incoming traffic only on needed ports i.e. SSH, HTTP, NTP
    ```
    sudo ufw allow ssh
    sudo ufw allow www
    sudo ufw allow ntp
    ```
    * As we want to change our SSH port to some non-default port number later on, lets add that port to firewall as well. This step is necessary in order to avoid being locked out of server, if the port is changed to some non-default value and that was not allowed in firewall.
    ```
    sudo ufw allow 2200/tcp
    ```
    * Enable firewall
    ```
    sudo ufw enable
    ```

  3. Change SSH Port 22 to 2200, Limit login to `grader` user

    *Source: [Initial Server Setup with Ubuntu 12.04](https://www.digitalocean.com/community/tutorials/initial-server-setup-with-ubuntu-12-04)*
    * Open `/etc/sshd_config`
    ```
    sudo nano /etc/sshd_config
    ```
    * Update the `Port` entry from `22` to `2200`
    ```
    Port 2200
    Protocol 2
    ```

  4. Block root login and limit login to only `grader` user

    * Update the entry `PermitRootLogin` to `no`
    ```
    PermitRootLogin no
    ```
    * Add following lines
    ```
    UseDNS no
    AllowUsers grader
    ```
    * Save and Exit
    * Restart the SSH service to take effect
    * `exit` and login again using new port

    ```
    ssh grader@52.25.36.89 -i ~/.ssh/fsnd-grader -p 2200
    ```

  5. Update firewall to deny traffic on default ssh port 22

    ```
    sudo ufw deny 22
    sudo ufw deny 22/tcp
    ```

  6. **Monitor unsuccessful login attempts**

    *Source: [How To Protect SSH with Fail2Ban on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-protect-ssh-with-fail2ban-on-ubuntu-14-04)*

    To implement monitoring of unsuccessful login attempts, fail2ban can be installed and configured.
    * Install fail2ban

    ```
    sudo apt-get install fail2ban
    ```

    * Copy the default config `/etc/fail2ban/jail.conf` to `/etc/fail2ban/jail.local`

    ```
    sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
    ```

    * Edit the `jail.local` to override default configuration

    ```
    sudo nano /etc/fail2ban/jail.local
    ```

    * Make following changes to `jail.local`

    ```
    # This value is in seconds. It will block the client IP address for 30 minutes
    bantime = 1800

    ...

    destemail = <Enter email address to recieve fail2ban notifications>

    ...

    # Set action to %(action_mwl)s to include log lines in the email alerts
    action = %(action_mwl)s
    ...    
    ```

    * Change SSH Port to 2200 from 22
    Look for [ssh], and change the port to 2200

    ```
    [ssh]
    ...
    enabled = true
    port = 2200
    ...
    ```

    * Save and exit

    * Install `sendmail`

    ```
    sudo apt-get install sendmail
    ```

    * Stop the `fail2ban` service

    ```
    sudo service fail2ban stop
    ```

    * Start the `fail2ban` service

    ```
    sudo service fail2ban start
    ```

    *Stopping and starting the service will let the new changes take effect.*

## 4. Update Installed Packages
  *Source: [Udacity > Configuring Linux Web Servers > Lession 2: Linux Security > Updating Available Package Lists](https://www.udacity.com/course/viewer#!/c-ud299-nd/l-4331066009/m-4801089452)*
  * Update the installed packages using `apt-get`
  ```
  sudo apt-get update
  sudo apt-get upgrade
  ```

  NOTE: If following message appears
  ```
  A new version of /boot/grub/menu.lst is available, but the version installed currently has been locally modified.
  ```

  Choose the second option,
  ```
  keep the local version currently installed
  ```

  ### Automatic updates
  *Source: [AutomaticSecurityUpdates](https://help.ubuntu.com/community/AutomaticSecurityUpdates)*

  Use following commands to enable automatic updates

  * Install unattended-upgrades

  ```
  sudo apt-get install unattended-upgrades
  ```

  * Enable auto updates

  ```
  sudo dpkg-reconfigure --priority=low unattended-upgrades
  ```

## 5. Apache Web Server
Apache web server will be used to host out item catalog web application. It will be installed and configured to serve python mod_wsgi application.

*Source: [A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu](http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html)*

### 1. Install apache2
Install apache web server using following command.
```
sudo apt-get install apache2
```

### 2. Install dependencies
`mod_wsgi` ensures Apache and python work together. So we need to install this.
```
sudo apt-get install python-setuptools libapache2-mod-wsgi
```

Restart `apache2` service for `mod_wsgi` to load.
```
sudo service apache2 restart
```

**At this point, the apache is up and running on the remote VM. It can be checked by loading this url in browser. [http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/](http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/)**

## 6. PostgreSQL

*Source: [How To Install and Use PostgreSQL on Ubuntu 14.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-postgresql-on-ubuntu-14-04)*

### 1. Install postgresql
Run following command to install PostgreSQL
```
sudo apt-get install postgresql postgresql-contrib
```

### 2. Add new user
Switch to `postgres` user. This user is created by default with the installation of PostgreSQL.
```
su -i -u postgres
```

Create a new user named `catalog` for PostgreSQL database for our Item Catalog Application
```
createuser -P catalog
```
It will prompt for password because of -P switch. User will be created with limited permissions.

### 3. Secure postgres

*Source: [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)*

Make sure PostgreSQL does not allow remote connections. PostgreSQL is set to disable remote connections by default and it can be verified as below.
* Open `/etc/postgresql/9.3/main/pg_hba.conf` and check that the entries should look like this.

```
local   all             postgres                                peer

# TYPE  DATABASE        USER            ADDRESS                 METHOD
"local" is for Unix domain socket connections only
local   all             all                                     peer
# IPv4 local connections:
host    all             all             127.0.0.1/32            md5
# IPv6 local connections:
host    all             all             ::1/128                 md5
```

*As the host type entries have values 127.0.0.1/32 and ::1/128, this ensures the remotes connection are disabled.*

## 7. git
In order to serve existing Item Catalog application, it needs to be cloned on Linux VM machine from [github](https://github.com/zeeshaanahmad/fsnd-p3-itemcatalog). git needs to be installed to access github and clone the application source.

### 1. Install git
* Run following command to install git
```
sudo apt-get install git
```

### 2. Clone Item Catalog
* Go to `/var/www/` and create a folder `itemcatalog`
```
cd /var/www/
mkdir itemcatalog
cd itemcatalog
sudo git clone https://github.com/zeeshaanahmad/fsnd-p3-itemcatalog.git itemcatalog
```
*This will copy the itemcatalog source to `/var/www/itemcatalog/itemcatalog`*

### 3. Make Git web inaccessible

*Source: [Make .git directory web inaccessible](http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible)*

Create a file named `.htaccess` in `/var/www/itemcatalog`
Write this in `.htaccess`

```
RedirectMatch 404 /\.git
```

## 8. Hosting application

*Source: [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)*

### 1. Enable `mod_wsgi`
Run following command to enable `mod_wsgi`

```
sudo a2enmod wsgi
```

### 2. Virtual Environment
Setting up a virtual environment will keep the application and its dependencies isolated from the main system. Changes to it will not affect the cloud server's system configurations.

Go to `/var/www/itemcatalog/itemcatalog`

#### Install `pip`
`pip` is needed to install `virtualenv` and `Flask`. So first install `pip` using `apt-get`.

```
sudo apt-get install python-pip
```

#### 2. Install `virtualenv`
Using `pip`, install `virtualenv`

```
sudo pip install virtualenv
```

#### 3. Create Virtual Environment
create virtual environment using `virtualenv` with name `venev` and modify permissions.

```
sudo virtualenv venv
sudo chmod -R 777 venv
```

#### 4. Activate Virtual Environment
Activate the virtual environment

```
source venv/bin/activate
```

### 3. Install dependencies within Virtual Environment

#### Flask
Install `Flask` using `pip`

```
sudo pip install Flask
```

#### oauth2client.client
Install `oauth2client.client` using `pip`

```
sudo pip install --upgrade oauth2client
```

#### httplib2

Install `httplib2` using `pip`

```
sudo pip install httplib2
```

#### requests
Install `requests` using `pip`

```
sudo pip install requests
```

#### SQLAlchemy
Install `SQLAlchemy` using `pip`

```
sudo pip install sqlalchemy
```

#### python-psycopg2
Install the `python-psycopg2` using `apt-get`

```
sudo apt-get install python-psycopg2
```

### 4. Apache Virtual Host Configuration

#### 1. Create new Virtual Host

* Create a file in /etc/apache2/sites-available/ with the name of itemcatalog.conf
This will hold the configuration for Virtual Host pointing towards item catalog application.

```
sudo nano /etc/apache2/itemcatalog.conf
```

* Insert the following lines in this file

```
<VirtualHost *:80>
      ServerName 52.25.36.89
      ServerAdmin admin@52.25.36.89
      WSGIScriptAlias / /var/www/itemcatalog/itemcatalog.wsgi
      <Directory /var/www/itemcatalog/itemcatalog/>
          Order allow,deny
          Allow from all
      </Directory>
      Alias /static /var/www/itemcatalog/itemcatalog/static
      <Directory /var/www/itemcatalog/itemcatalog/static/>
          Order allow,deny
          Allow from all
      </Directory>
      ErrorLog ${APACHE_LOG_DIR}/error.log
      LogLevel warn
      CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

* Save and Exit

#### 2. Create `.wsgi` file

* Go to `/var/www/itemcatalog`
* Create a file named `itemcatalog.wsgi`

```
sudo nano itemcatalog.wsgi
```

* Add following python code in this file

```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0,"/var/www/itemcatalog")

from itemcatalog import app as application
#from itemcatalog import app as application
application.secret_key = 'super_secret_key'
```

* Save and Exit
* Go to `/var/www/itemcatalog/itemcatalog`

```
cd /var/www/itemcatalog/itemcatalog
```

* Rename `/var/www/itemcatalog/itemcatalog/app.py` to `/var/www/itemcatalog/itemcatalog/__init__.py`

```
mv app.py __init__.py
```

### 5. Item Catalog source modifications for production

#### 1. Remove debugging settings from __init__.py

* Open `__init__.py`

```
sudo nano __init.py__
```

* Remove the application's debugging settings in the end and replace with `app.run()`
Replace this section

```
if __name__ == '__main__':
    app.secret_key = 'super_secret_key'
    app.debug = True
    app.run(host='0.0.0.0', port=5000)
```

With

```
if __name__ == '__main__':
    app.run()
```

#### 2. Update database connection string
Update database connection string in create_engine method call in `__init__.py`, db_setup.py, populate_item_categories.py files. This will switch the database to PostgreSQL instead of previously configured sqlite.
Replace this line

```
engine = create_engine('sqlite:///itemcatalogwithusers.db')
```

With

```
engine = create_engine('postgresql://catalog:catalog@localhost/catalog')
```

#### 3. Change server address in JS file
Replace the address used in REST calls  i.e. `localhost:5000` with `ec2-52-25-36-89.us-west-2.compute.amazonaws.com`

#### 4. Permissions on uploads folder
Give permissions to all for writing and reading files from uploads directory.

```
sudo chmod -R 777 /var/www/itemcatalog/itemcatalog/static/uploads
```

#### 5. Use os.chdir() before reading client_secrets.json in `__init__.py`
Add following line `os.chdir(r'/var/www/itemcatalog/itemcatalog')` before `CLIENT_ID = json.loads(open('client_secrets.json', 'r').read())['web']['client_id']`. This will resolve the error of not being able to read client_secrets.json on server.

## 9. Configuration for Google+/Facebook Sign in
### Google Sign in
* Login to `console.developers.google.com` and select the previously created project for item catalog
* Go to API Manager > credentials
* Pick the entry for previously configured OAuth 2.0 client ID from table
* Add `http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com` to Authorized JavaScript Origins
* Add `http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/oauth2callback` to Authorized redirect URIs
* Save Changes

### Facebook Sign in
* Login to `developers.facebook.com` and select the previously created project for item catalog
* Go to settings
* Update the Site URL to `http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com`
* Save Changes

## Just a few steps
* Enable the site

```
sudo a2ensite itemcatalog
```

* Restart `apache2`

```
sudo service apache2 restart
```

# All Done! The Site is Up and Running!

## Monitoring
Installed `Glances` for monitoring of web server

```
sudo apt-get install Glances
```

Launch Glances to monitor system

```
glances
```
![Glances](http://i.imgur.com/eo8BgaJ.png)

[1]: http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/
