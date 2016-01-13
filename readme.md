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

# Softwares Installed
## apache2
```
sudo apt-get install apache2
```
## PostgreSQL
```
sudo apt-get install postgresql postgresql-contrib
```


[1]: http://ec2-52-25-36-89.us-west-2.compute.amazonaws.com/
