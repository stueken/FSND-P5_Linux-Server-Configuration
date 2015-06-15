# FSND-P5_Linux-Server-Configuration

A Baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

**Note:** The below step-by-step walkthrough is the solution to project 5 of the [Udacity Full Stack Web Developer Nanodegree][1] and deploys the [solution of project 3][2] on the virtual machine. The solution is graded with "Exceeds Specifications".

## Step by Step Walkthrough
### 1 & 2 - Create Development Environment: Launch Virtual Machine and SSH into the server
Source: [Udacity][3]  

1. Create new development environment.
2. Download private keys and write down your public IP address.
3. Move the private key file into the folder ~/.ssh:  
  `$ mv ~/Downloads/udacity_key.rsa ~/.ssh/`
4. Set file rights (only owner can write and read.):  
  `$ chmod 600 ~/.ssh/udacity_key.rsa`
5. SSH into the instance:  
  `<pre>$ ssh -i ~/.ssh/udacity_key.rsa root@PUPLIC-IP-ADDRESS`

### 3 & 4 - User Management: Create a new user and give user the permission to sudo
Source: [DigitalOcean][4]  

1. Create a new user:  
  `$ adduser NEWUSER`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ visudo`
  2. Add the following line below `root ALL...`:  
    `NEWUSER ALL=(ALL:ALL) ALL`
  3. *List all users (Source: [Ask Ubuntu][5]):    
    `$ cut -d: -f1 /etc/passwd`

### 5 - Update and upgrade all currently installed packages
Source: [Ask Ubuntu][6]  
    
1. Update the list of available packages and their versions:  
  `$ sudo apt-get update`
2. Install newer vesions of packages you have:  
  `$ sudo sudo apt-get upgrade`

#### 5** - Include cron scripts to automatically manage package updates
Source: [Ubuntu documentation][7]  

1. Install the unattended-upgrades package:  
  `$ sudo apt-get install unattended-upgrades`
2. Enable the unattended-upgrades package:  
  `$ sudo dpkg-reconfigure -plow unattended-upgrades`

### 6 - Change the SSH port from 22 to 2200 and configure SSH access
Source: [Ask Ubuntu][8]  

1. Change ssh config file:
  1. Open the config file:  
    `$ vim /etc/ssh/sshd_config` 
  2. Change to Port 2200.
  3. Change `PermitRootLogin` from `without-password` to `no`.
  4. * To get more detailed logging messasges, open `/var/log/auth.log` and change LogLevel from `INFO` to `VERBOSE`. 
  5. Temporalily change `PasswordAuthentication` from `no` to `yes`.
  6. Append `UseDNS no`.
  7. Append `AllowUsers NEWUSER`.  
**Note:** All options on [UNIXhelp][9]
2. Restart SSH Service:  
  `$ /etc/init.d/ssh restart` or `# service sshd restart` 
3. Create SSH Keys:  
  Source: [DigitalOcean][10]  

  1. Generate a SSH key pair on the local machine:  
    `$ ssh-keygen`
  2. Copy the public id to the server:  
    `$ ssh-copy-id username@remote_host -p**_PORTNUMBER_**`
  3. Login with the new user:  
    `$ ssh -v grader@PUBLIC-IP-ADDRESS -p2200`
  4. Open SSHD config:  
    `$ sudo vim /etc/ssh/sshd_config`
  5. Change `PasswordAuthentication` back from `yes` to `no`.
4. *Get rid of the warning message `sudo: unable to resolve host ...` when sudo is executed:  
Source: [Ask Ubuntu][11]  

  1. Open `$ vim /etc/hostname`.
  2. Copy the hostname.
  3. Append the hostname to the first line:  
    `$ sudo sudonano /etc/hosts`
5. * Simplify SSH login:  
  1. Logout of the SSH instance:  
    `$ exit`
  2. Open the SSH config file on your local machine:  
    `$ sudo vim .ssh/config`
  3. Add the following lines:  
    ```
    Host NEWHOSTNAME
      HostName PUPLIC-IP-ADDRESS
      Port 2200
      User NEWUSER
    ```
  4. Now, you can login into the server more quickly:  
    `$ ssh NEWHOSTNAME`
6. *Handle the message `System restart required` after login:  
Source: [Super User][12]  

  1. List all packages which cause the reboot:  
    `$ cat /var/run/reboot-required.pkgs`
  2. List everything with high security issues:  
    `$ xargs aptitude changelog < /var/run/reboot-required.pkgs | grep urgency=high`
  3. If wanted or needed, reboot the system:  
    `$ sudo shutdown -r now`  
    **Note**: More info on rebooting on [Ask Ubuntu][13].

### 7 - Configure the Uncomplicated Firewall (UFW) to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123)
Source: [Ubuntu documentation][14]  

1. Turn UFW on with the default set of rules:  
  `$ sudo ufw enable` 
2. *Check the status of UFW:  
  `$ sudo ufw status verbose`
3. Allow incoming TCP packets on port 2200 (SSH):  
  `$ sudo ufw allow 2200/tcp` 
4. Allow incoming TCP packets on port 80 (HTTP):  
  `$ sudo ufw allow 80/tcp` 
5. Allow incoming UDP packets on port 123 (NTP):  
  `$ sudo ufw allow 123/udp`  

#### 7** - Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
Source: [DigitalOcean][15]  

1. Install Fail2ban:  
  `$ sudo apt-get install fail2ban`
2. Copy the default config file:  
  `$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local`
3. Check and change the default parameters:  
    1. Open the local config file:  
      `$ sudo vim /etc/fail2ban/jail.local`
    2. Set the following Parameters:  
    ```  
      set bantime  = 1800  
      destemail = YOURNAME@DOMAIN  
      action = %(action_mwl)s  
      under [ssh] change port = 2220  
    ```  
4. Install needed software for our configuration:  
  `$ sudo apt-get install sendmail iptables-persistent` 
5. Set up a basic firewall only allowing connections from the above ports:  
  `$ sudo iptables -A INPUT -i lo -j ACCEPT`  
  `$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT`  
  `$ sudo iptables -A INPUT -p tcp --dport 2200 -j ACCEPT`  
  `$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT`  
  `$ sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT`  
  `$ sudo iptables -A INPUT -j DROP`  
6. *Check the current firewall rules:  
  `$ sudo iptables -S`
7. Stop the service:  
  `$ sudo service fail2ban stop`
8. Start it again:  
  `$ sudo service fail2ban start`

### 8 - Configure the local timezone to UTC
Source: [Ubuntu documentation][16]

1. Open the timezone selection dialog:  
  `$ sudo dpkg-reconfigure tzdata`
2. Then chose 'None of the above', then UTC.
3. *Setup the ntp daemon ntpd for regular and improving time sync:  
  `$ sudo apt-get install ntp`
4. *Chose closer NTP time servers:  
  1. Open the NTP configuration file:  
    `$ sudo vim /etc/ntp.conf`
  2. Open http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list.  

### 9 - Install and configure Apache to serve a Python mod_wsgi application
Source: [Udacity][17]

1. Install Apache web server:  
  `$ sudo apt-get install apache2`
2. Open a browser and open your public ip address, e.g. http://52.25.0.41/ - It should  say 'It works!' on the top of the page.
3. Install **mod_wsgi** for serving Python apps from Apache and the helper package **python-setuptools**:  
  `$ sudo apt-get install python-setuptools libapache2-mod-wsgi`
4. Restart the Apache server for mod_wsgi to load:  
  `$ sudo service apache2 restart`  
5. *Get rid of the message "Could not reliably determine the servers's fully qualified domain name" after restart
  Source: [Ask Ubuntu][18]
  1. Create an empty Apache config file with the hostname:  
    `$ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf`
  2. Enable the new config file:  
    `$ sudo a2enconf fqdn`

### 11 - Install git, clone and setup your Catalog App project
#### 11.1 - Install and configure git
        https://help.github.com/articles/set-up-git/#platform-linux
        $ sudo apt-get install git - to install git
        $ git config --global user.name "YOUR NAME" - to set your name, e.g. for the commits
        $ git config --global user.email "YOUR EMAIL ADDRESS" - to connect your commits to your account
    11b - Setup for deploying a Flask Application on Ubuntu VPS
        https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
        $ sudo apt-get install libapache2-mod-wsgi python-dev - to extend Python with additional packages that enable Apache to serve Flask applications
        *$ sudo a2enmod wsgi - to enable mod_wsgi (if not already enabled)
    11c - Create a Flask app
        $ cd /var/www - to move to the www directory
        $ sudo mkdir catalog - to create a directory for the app, e.g. catalog
        $ cd catalog AND $ sudo mkdir catalog
        $ cd catalog AND $ sudo mkdir static templates
        $ sudo nano __init__.py - to create the file that will contain the flask application logic
        Paste in the following code:
            from flask import Flask
            app = Flask(__name__)
            @app.route("/")
            def hello():
                return "Veni vidi vici!!"
            if __name__ == "__main__":
                app.run()
    11d - Install Flask
        $ sudo apt-get install python-pip - to install pip installer
        $ sudo pip install virtualenv - to install virtualenv
        $ sudo virtualenv venv - set virtual environment to name venv
        Enable all permissions for the new virtual environment (no sudo should be used within):
            http://stackoverflow.com/questions/14695278/python-packages-not-installing-in-virtualenv-using-pip
            $ sudo chmod -R 777 venv
        $ source venv/bin/activate - to activate the virtual environment
        $ pip install Flask - install Flask inside the virtual environment
        $ python __init__.py - to run the app
        $ deactivate - to deactivate the environment
    11e - Configure and Enable a New Virtual Host
        $ sudo nano /etc/apache2/sites-available/catalog.conf - to create a virtual host config file
        Paste in the following lines of code and change names and address regarding your application:
            <VirtualHost *:80>
                ServerName 52.25.0.41
                ServerAdmin admin@52.25.0.41
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
        $ sudo a2ensite catalog - to enable the virtual host
    11f - Create the .wsgi File and Restart Apache
        $ cd /var/www/catalog AND $ sudo nano catalog.wsgi - to create wsgi file
        Paste in the following lines of code
            #!/usr/bin/python
            import sys
            import logging
            logging.basicConfig(stream=sys.stderr)
            sys.path.insert(0,"/var/www/catalog/")
            from catalog import app as application
            application.secret_key = 'Add your secret key'
        $ sudo service apache2 restart
        !!! Logging
            https://docs.python.org/2.4/lib/minimal-example.html
            https://docs.python.org/2/howto/logging.html
            http://docs.python-guide.org/en/latest/writing/logging/
            Permissions for log files
                http://stackoverflow.com/questions/18547855/permission-denied-when-writing-log-file
    11g - Clone GitHub repository and make it web inaccessible
        $ git clone https://github.com/stueken/FSND-P3_Music-Catalog-Web-App.git - Clone repository on GitHub
        move all content of created FSND-P3_Music-Catalog-Web-App directory to  /var/www/catalog/catalog/ directory and delete the leftover empty directory
        http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
        change to /var/www/catalog/ directory
        $ sudo nano .htaccess - create .htaccess files
        Paste in the following:
            RedirectMatch 404 /\.git
    11h - Install needed modules & packages
        $ source venv/bin/activate
        $ pip install httplib2 - to install httplib2 module in venv
        $ pip install requests - to install requests module in venv
        $ *sudo pip install flask-seasurf - to install flask.ext.seasurf (only seems to work when installed globally)
        $ sudo pip install --upgrade oauth2client - to install oauth2client.client (only seems to work when installed globally)
        $ sudo pip install sqlalchemy (only seems to work when installed globally)
    $ sudo apt-get install python-psycopg2 - to install the Python PostgreSQL adapter psycopg



---
[1]: https://de.wikipedia.org/wiki/Flask "Wikipedia entry to Flask"
[2]: https://github.com/stueken/FSND-P3_Music-Catalog-Web-App "GitHub repository of an item catalog web app"
[3]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[4]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[5]: http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users "How to list, add, delete and modify users"
[6]: http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade "What is the difference between apt-get update and upgrade?"
[7]: https://help.ubuntu.com/community/AutomaticSecurityUpdates "AutomaticSecurityUpdates"
[8]: http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server "Create a new SSH user on Ubuntu Server"
[9]: http://unixhelp.ed.ac.uk/CGI/man-cgi?sshd_config "UNIX man page: SSHD_CONFIG"
[10]: https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server "How To Configure SSH Key-Based Authentication on a Linux Server"
[11]: http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none "Error message when I run sudo: unable to resolve host (none)"
[12]: http://superuser.com/questions/815433/how-urgent-is-a-system-restart-required-for-security "How urgent is a *** System restart required *** for security?"
[13]: http://askubuntu.com/questions/483670/what-causes-ssh-problems-after-rebooting-a-14-04-server "What causes SSH problems after rebooting a 14.04 server?"
[14]: https://help.ubuntu.com/community/UFW "UFW - Uncomplicated Firewall"
[15]: https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04 "How To Install and Use Fail2ban on Ubuntu 14.04"
[16]: https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29 "Ubuntu Time Management"
[17]: http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html "A Step by Step Guide to Install LAMP (Linux, Apache, MySQL, Python) on Ubuntu"
[18]: http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name "Could not reliably determine the server's fully qualified domain name?"
