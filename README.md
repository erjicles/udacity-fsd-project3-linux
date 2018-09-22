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
3. Give grader account sudo access:
    1. Copy file granting default account (ubuntu) sudo access:
        ```
        sudo cp /etc/sudoers.d/90-cloud-init-users /etc/sudoers.d/grader
        ```
    2. Edit the file:
        ```
        sudo nano /etc/sudoers.d/grader
        ```
    3. Replace "ubuntu" with "grader" so the final file looks like this:
        ```
        # User rules for grader
        grader ALL=(ALL) NOPASSWD:ALL
        ```
    4. Save the file
4. Generate authentication key for grader
    1. On your local machine, run the following command:
        ```
        ssh-keygen
        ```
    2. For this purpose, don't enter a passphrase. Just hit enter when prompted
    3. Save the generated files in a location on your local machine
5. Add grader's ssh public key to server
    1. Locate the generated public key from the previous step (.pub file)
    2. Copy its contents on the local machine:
        ```
        cat /path/to/public/key/file.pub
        ```
        Highlight to copy the file contents
    3. ssh into the server
    4. Momentarily switch to the grader account:
        ```
        sudo -u grader -i
        ```
    5. Create the .ssh directory under the grader's home folder:
        ```
        mkdir /home/grader/.ssh
        ```
    6. Create the authorized_keys file under the new directory:
        ```
        nano /home/grader/.ssh/authorized_keys
        ```
    7. Paste in the contents from the local public key file
    8. Save the file
    9. Set permissions on the new directory and file:
        ```
        chmod 700 /home/grader/.ssh
        chmod 644 /home/grader/.ssh/authorized_keys
        ```
    10. Switch back from the grader account to your account
        ```
        exit
        ```


### 6. Configure the server to UTC time

1. ssh into the server
2. Run the following command:
    ```
    sudo dpkg-reconfigure tzdata
    ```
3. Scroll to the bottom of the Continents list and select Etc
or None of the above; in the second list, select UTC


### 7. Install and configure PostgreSQL

1. ssh into the server
2. Install postgresql:
    ```
    sudo apt-get install postgresql
    ```
    Note - on my server, this installed PostgreSQL version 10.5
3. Configure PostgreSQL to not allow remote connections
    1. Edit the PostgreSQL configuration file:
        ```
        sudo nano /path/to/config/file.conf
        ```
        Note - for this server, the configuration file was located here:  
        /etc/postgresql/10/main/postgresql.conf
    2. Locate the listen_addresses setting, and edit it as follows:
        ```
        listen_addresses = 'localhost'
        ```
        Note - this setting defaults to localhost if commented out. We actually
        left it commented out due to that fact.


### 8. Install and configure Apache

1. ssh into the server
2. Install apache:
    ```
    sudo apt-get install apache2
    ```
3. Install Python 3 mod, as the web application was written in Python 3:
    ```
    sudo apt-get install libapache2-mod-wsgi-py3
    ```


### 9. Install git
1. ssh into the server
2. Install git:
    ```
    sudo apt-get install git
    ```


### 10. Bring all applications up to date

1. ssh into the server
2. Run the following commands:
    ```
    sudo apt-get update
    sudo apt-get upgrade
    ```


### 11. Deploy Web Application
0. ssh into the server
1. Clone web application to application directory:
    ```
    cd /var/www
    sudo git clone -b linux-project https://github.com/erjicles/udacity-fsd-project2-item-catalog
    ```
2. Copy **client_secrets.json** and **app_secrets.json** files from local
machine to application directory
3. Create catalog database user for use by my web application:
    1. Momentarily swith to the PostgreSQL super user:
        ```
        sudo -u postgres -i
        ```
    2. Create the new catalog user by running the following command:
        ```
        createuser catalog
        ```
4. Create application database **itemcatalog**:
    1. Open the PostgreSQL command prompt:
        ```
        psql
        ```
    2. Create the database:
        ```
        CREATE DATABASE itemcatalog;
        ```
    3. Allow the **catalog** user to connect:
        ```
        GRANT CONNECT ON DATABASE itemcatalog TO catalog;
        ```
    4. Exit the psql command prompt (type \q), and cd to the app directory:
        ```
        cd /var/www/udacity-fsd-project2-item-catalog
        ```
    5. Create the database structure and seed data:
        ```
        python3 database_setup.py
        python3 prepopulate_database.py
        ```
    6. Reopen the PostgreSQL command prompt, and connect to the
    **itemcatalog** database:
        ```
        psql itemcatalog
        ```
    7. Grant the **catalog** user CRUD permissions to the database tables:
        ```
        GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO catalog;
        ```
    8. Exit the PostgreSQL command prompt and switch back to your user:
        ```
        \q
        exit
        ```
5. Create application service account **catalog**
    ```
    sudo adduser catalog
    ```
    When prompted, give the account the password "catalog"
6. Configure Apache virtual host
    1. Create sites-enabled configuration file for web app:
        ```
        sudo cp /etc/apache2/sites-enabled/000-default.conf /etc/apache2/sites-enabled/itemcatalog.conf
        ```
    2. Edit default sites-enabled config file to listen on closed port 8080:
        ```
        sudo nano /etc/apache2/sites-enabled/000-default.conf
        ```
        Change the first line to read:
        ```
        <VirtualHost *:8080>
        ```
        Save the file
    3. Edit the application sites-enabled config file:
        ```
        sudo nano /etc/apache2/sites-enabled/itemcatalog.conf
        ```
        Modify the file so that it has the following contents, and save:
        ```
        <VirtualHost *:80>
            # The ServerName directive sets the request scheme, hostname and port that
            # the server uses to identify itself. This is used when creating
            # redirection URLs. In the context of virtual hosts, the ServerName
            # specifies what hostname must appear in the request's Host: header to
            # match this virtual host. For the default virtual host (this file) this
            # value is not decisive as it is used as a last resort host regardless.
            # However, you must set it for any further virtual host explicitly.
            ServerName www.erikronaldjohnson.com
            ServerAlias erikronaldjohnson.com


            ServerAdmin webmaster@localhost
            DocumentRoot /var/www/udacity-fsd-project2-item-catalog/public_html

            # Available loglevels: trace8, ..., trace1, debug, info, notice, warn,
            # error, crit, alert, emerg.
            # It is also possible to configure the loglevel for particular
            # modules, e.g.
            #LogLevel info ssl:warn

            ErrorLog ${APACHE_LOG_DIR}/error.log
            CustomLog ${APACHE_LOG_DIR}/access.log combined

            # For most configuration files from conf-available/, which are
            # enabled or disabled at a global level, it is possible to
            # include a line for only one particular virtual host. For example the
            # following line enables the CGI configuration for this host only
            # after it has been globally disabled with "a2disconf".
            #Include conf-available/serve-cgi-bin.conf

            # Custom: Handle requests using WSGI module
            WSGIDaemonProcess itemcatalog user=catalog group=catalog threads=5 home=/var/www/udacity-fsd-project2-item-catalog
            WSGIScriptAlias / /var/www/udacity-fsd-project2-item-catalog/public_html/itemcatalog.wsgi

            Alias /static /var/www/udacity-fsd-project2-item-catalog/static
            <Directory /var/www/udacity-fsd-project2-item-catalog/static>
                    Require all granted
            </Directory>

            <Directory /var/www/udacity-fsd-project2-item-catalog>
                    WSGIProcessGroup itemcatalog
                    WSGIApplicationGroup %{GLOBAL}
                    Require all granted
            </Directory>
        </VirtualHost>
        ```
        This configuration does a few things: 
        - Sets the document root to the public_html directory within the 
        application directory. This ensures that public requests
        only hit that directory and can't access the application directory
        directly.
        - Runs the application using the **catalog** service account,
        and sets its starting working directory to the application directory
        so it can access the application files.
        - Exposes the /static subdirectory to public requests
        - Serves the application for both erikronaldjohnson.com 
        and www.erikronaldjohnson.com
    4. Restart apache
        ```
        sudo service apache2 restart
        ```


## Third Party Resources
I used the following 3rd party resources to aid in completing the project.

- Changing the default SSH port
    - https://www.godaddy.com/help/changing-the-ssh-port-for-your-linux-server-7306
- Configuring UFW:
    - https://wiki.ubuntu.com/UncomplicatedFirewall
    - https://askubuntu.com/questions/30781/see-configured-rules-even-when-inactive
- Configuring UTC time:
    - https://askubuntu.com/questions/138423/how-do-i-change-my-timezone-to-utc-gmt
- Configure PostgreSQL
    - https://www.postgresql.org/docs/10/static/index.html
- Configure Apache
    - http://flask.pocoo.org/docs/1.0/deploying/mod_wsgi/
    - http://httpd.apache.org/docs/2.4/