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
## 1. User Management
1. Create user named `grader`
```bash
adduser grader
```
1. Perform these steps to grant sudo permissions to grader user. Create a file with user's name in following location `/etc/sudoers.d` i.e;
```bash
sudo nano /etc/sudoers.d/grader
```
1. Add the following lines to this file
```bash
grader    ALL=(ALL) NOPASSWD:ALL
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
