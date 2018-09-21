# Project: Linux Server Configuration

Author: Erik Johnson  
Date: September 21, 2018  
Course: Udacity Full Stack Web Developer Nanodegree


## Objective

Take a baseline installation of a Linux server and prepare it to host my web
applications. Secure my server from a number of attack vectors, install and
configure a database server, and deploy one of my existing web applications
onto it.


## Linux Server

In this project, I use Amazon Lightsail to host an instance of a Ubuntu Linux
server in the cloud:

IP: 18.216.98.6  
SSH Port: 2200


## Web Application

I chose to use my existing
[Item Catalog](https://github.com/erjicles/udacity-fsd-project2-item-catalog/tree/linux-project)
web application for this project (I created that web application as a project
earlier in the Udacity Full Stack Web Developer Nanodegree program). Before I 
could deploy, I had to branch that project and make some changes to make it
work on the Amazon Lightsail instance. Those changes are detailed in the README 
for this branch of that project, linked above.

The web application uses Google authentication. However, Google Auth requires
a domain name for the javascript redirect url, and doesn't work with just an
IP (see [here](https://stackoverflow.com/questions/14238665/can-a-public-ip-address-be-used-as-google-oauth-redirect-uri)).
To get around this, I used a domain I already owned when configuring the
server and web application. 

Specifically, I had to set the nameservers for my domain to the ones provided
by Lightsail, and I had to add my domain url as a javascript redirect url in
my web application's credentials in the Google developer console.

Here is the final URL to my hosted web application:

http://erikronaldjohnson.com


## Server Configuration

### 1. Configure Amazon Lightsail Firewall
Amazon Lightsail has its own firewall external to the Linux instance. I had to
use this to open ports 80 (www), 2200 (non-default SSH) port, and 123 (NTP).
I also had to close port 22 (default SSH) once the non-default port was verified
to be working.

To manage the Lightsail firewall for my instance, I followed these steps:
1. From the Lightsail home page, Instances tab, locate my instance
2. Click "Manage"
3. Networking tab, manage firewall ports from there

### 2. SSH Port

The first change I made was to change the default SSH port from 22 to 2200:

0. Open port 2200 on the Lightsail firewall (see above)
1. SSH into the server
2. Run the following command:
```
sudo nano /etc/ssh/sshd_config
```
3. Locate the following line:
```
# Port 22
```
4. Remove # and change 22 to 2200
5. Restart the sshd service:
```
sudo sshd restart
```
6. Log out and SSH back in (on the new port) to verify it's working
7. Remove port 22 from the Lightsail firewall

### 3. Configure Uncomplicated Firewall (UFW)

1. SSH into the server
2. Configure UFW to deny incoming requests by default (note - this is the
default setting for UFW):
```
sudo ufw default deny incoming
```
3. Configure UFW to allow outgoing requests by default (note - this is the
default setting for UFW):
```
sudo ufw default allow outgoing
```
4. Open ports 80 (www), 123 (NTP), and 2200 (non-default SSH):
```
sudo ufw allow www/tcp
sudo ufw allow ntp/tcp
sudo ufw allow 2200/tcp
```
5. Check UFW added rules to make sure everything is configured correctly:
```
sudo ufw show added
```
6. Enable UFW:
```
sudo ufw enable
```
7. Check UFW status:
```
sudo ufw status
```

### 4. Disable password logins
1. ssh into the server
2. Edit the ssh configuration file:
```
sudo nano /etc/ssh/sshd_config
```
3. Locate the following line (in my case, this was uncommented out already):
```
#PasswordAuthentication no
```
4. Uncomment the line (if it isn't already), and make sure the value is no
5. Save the file
6. Restart the ssh service:
```
sudo service ssh restart
```

### 5. Give grader access

1. ssh into the server
2. Create grader account:
```
sudo adduser grader
```
3. Give grader account sudo access
4. Generate authentication key for grader

## Third Party Resources
I used the following 3rd party resources to aid in completing the project.

- Changing the default SSH port
    - https://www.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306
- Configuring UFW:
	- https://help.ubuntu.com/community/UFW
	- https://askubuntu.com/questions/30781/see-configured-rules-even-when-inactive