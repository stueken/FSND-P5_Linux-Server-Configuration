# FSND-P5_Linux-Server-Configuration

## Step by Step Walkthrough
### 1 & 2 - Create Development Environment: Launch Virtual Machine and SSH into the server
    https://www.udacity.com/account#!/development_environment
    set file rights (only owner can write and read.
        $ chmod 600 ~/.ssh/udacity_key.rsa
    ssh into instance
        $ ssh -i ~/.ssh/udacity_key.rsa root@IP-Address
### 3 & 4 - User Management
    https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps
    create a new user
        $ adduser newuser
    give new user  the permission to sudo
        $ visudo - to edit the sudo configuration
        add line: newuser ALL=(ALL:ALL) ALL
    *list all users
        http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users
        $ cut -d: -f1 /etc/passwd
### 5 - Update and upgrade all apps
    http://askubuntu.com/questions/94102/what-is-the-difference-between-apt-get-update-and-upgrade
    $ sudo apt-get update - to update the list of available packages and their versions
    $ sudo sudo apt-get upgrade - to actually install newer vesions of packages you have.
#### 5** - Include cron scripts to automatically manage package updates
    https://help.ubuntu.com/community/AutomaticSecurityUpdates
    $ sudo apt-get install unattended-upgrades - to install the unattended-upgrades package
    $ sudo dpkg-reconfigure -plow unattended-upgrades - to enable the unattended-upgrades package
### 6 - Configure SSH access
    http://askubuntu.com/questions/16650/create-a-new-ssh-user-on-ubuntu-server
    Change ssh config file
        $ nano /etc/ssh/sshd_config
        change to Port 2200
        change PermitRootLogin from 'without-password' to 'no'
        *change LogLever from INFO to VERBOSE - to get more detailed logging messasges in /var/log/auth.log
        temporalily change to PasswordAuthentication from no to yes
        append UseDNS no
        append AllowUsers newuser
        all options: http://unixhelp.ed.ac.uk/CGI/man-cgi?sshd_config
    $ /etc/init.d/ssh restart or # service sshd restart - to restart SSH Service
    Create SSH Keys
        https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server
        $ ssh-keygen - on the local machine to generate a SSH key pair
        $ ssh-copy-id username@remote_host -pPORTNUMBER
        $ ssh -v grader@52.25.0.41 -p2200 - to login with the new user
        $ sudo nano /etc/ssh/sshd_config to change PasswordAuthentication back from yes to no
    *Get rid of warning message "sudo: unable to resolve host ..."
        http://askubuntu.com/questions/59458/error-message-when-i-run-sudo-unable-to-resolve-host-none
        $ nano /etc/hostname - to copy the hostname
        $ sudo sudonano /etc/hosts - to append hostname to first line
    *Simplify SSH login
        $ exit - to logout of SSH instance
        $ sudo nano .ssh/config - on local machine
        add the following lines.
            Host SOMENAME
                HostName 52.25.0.41
                Port 2200
                User grader
        $ ssh SOMENAME - to quickly login into the server
    *Handle the message *** System restart required *** after login
        http://superuser.com/questions/815433/how-urgent-is-a-system-restart-required-for-security
        $ cat /var/run/reboot-required.pkgs - to list all packages which cause the reboot
        $ xargs aptitude changelog < /var/run/reboot-required.pkgs | grep urgency=high - to only list everything with high security issues
        http://askubuntu.com/questions/483670/what-causes-ssh-problems-after-rebooting-a-14-04-server
        $ sudo shutdown -r now - to reboot the system if wanted
### 7 - Configure the Uncomplicated Firewall (UFW)
    https://help.ubuntu.com/community/UFW
    $ sudo ufw enable - to turn UFW on with the default set of rules
    *$ sudo ufw status verbose - to check the status of UFW
    $ sudo ufw allow 2200/tcp - to allow incoming tcp packets on port 2200 (SSH)
    $ sudo ufw allow 80/tcp - to allow incoming tcp packets on port 80 (HTTP)
    $ sudo ufw allow 123/udp - to allow incoming udp packets on port 123 (NTP)
7** - Configure Firewall to monitor for repeated unsuccessful login attempts and ban attackers
    https://www.digitalocean.com/community/tutorials/how-to-install-and-use-fail2ban-on-ubuntu-14-04
    $ sudo apt-get install fail2ban - to install the fail2ban tool
    $ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local - to copy the default config file
    $ sudo nano /etc/fail2ban/jail.local - to check and change the default parameters
        set bantime  = 1800
        destemail = yourname@domain
        action = %(action_mwl)s
        under [ssh] change port = 2220
    $ sudo apt-get install sendmail iptables-persistent - installing needed software for our configuration
    Set up a basic firewall only allowing connections from the above ports
        $ sudo iptables -A INPUT -i lo -j ACCEPT 
        $ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT 
        $ sudo iptables -A INPUT -p tcp --dport 2200 -j ACCEPT 
        $ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT 
        $ sudo iptables -A INPUT -p udp --dport 123 -j ACCEPT
        $ sudo iptables -A INPUT -j DROP
    *$ sudo iptables -S - to check the current firewall rules
    $ sudo service fail2ban stop - stop the service
    $ sudo service fail2ban start - start it again
### 8 - Configure the local timezone to UTC
    https://help.ubuntu.com/community/UbuntuTime#Using_the_Command_Line_.28terminal.29
    $ sudo dpkg-reconfigure tzdata - to open the timezone selection dialog (chose 'None of the above', then UTC)
    *$ sudo apt-get install ntp - to setup the ntp daemon ntpd for regular and improving time sync
    *Chose closer NTP time servers
        $ sudo nano /etc/ntp.conf - to open the ntp configuration file
        open the url: http://www.pool.ntp.org/en/ and choose the pool zone closest to you and replace the given servers with the new server list
### 9 - Install and configure Apache to serve a Python mod_wsgi application
    http://blog.udacity.com/2015/03/step-by-step-guide-install-lamp-linux-apache-mysql-python-ubuntu.html
    $ sudo apt-get install apache2 - to install Apache web server
    open a browser and open your public ip address, e.g. http://52.25.0.41/ - It should  say 'It works!' on the top of the page.
    $ sudo apt-get install python-setuptools libapache2-mod-wsgi - to install mod_wsgi for serving Python apps from Apache and the helper package python-setuptools
    $ sudo service apache2 restart - to restart the Apache server for mod_wsgi to load
    *Get rid of message "Could not reliably determine the servers's fully qualified domain name" after restart
        http://askubuntu.com/questions/256013/could-not-reliably-determine-the-servers-fully-qualified-domain-name
        $ echo "ServerName HOSTNAME" | sudo tee /etc/apache2/conf-available/fqdn.conf to create empty Apache conf file with the hostname
        $ sudo a2enconf fqdn - to enable the new conf file
### 11 - Install git, clone and setup your Catalog App project
#### 11a - Install and config git
        https://help.github.com/articles/set-up-git/#platform-linux
        $ sudo apt-get install git - to install git
        $ git config --global user.name "YOUR NAME" - to set your name, e.g. for the commits
        $ git config --global user.email "YOUR EMAIL ADDRESS" - to connect your commits to your account
#### 11b - Setup for deploying a Flask Application on Ubuntu VPS
        https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps
        $ sudo apt-get install libapache2-mod-wsgi python-dev - to extend Python with additional packages that enable Apache to serve Flask applications
        *$ sudo a2enmod wsgi - to enable mod_wsgi (if not already enabled)
#### 11c - Create a Flask app
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
#### 11d - Install Flask
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
#### 11e - Configure and Enable a New Virtual Host
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
#### 11f - Create the .wsgi File and Restart Apache
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
#### 11g - Clone GitHub repository and make it web inaccessible
        $ git clone https://github.com/stueken/FSND-P3_Music-Catalog-Web-App.git - Clone repository on GitHub
        move all content of created FSND-P3_Music-Catalog-Web-App directory to  /var/www/catalog/catalog/ directory and delete the leftover empty directory
        http://stackoverflow.com/questions/6142437/make-git-directory-web-inaccessible
        change to /var/www/catalog/ directory
        $ sudo nano .htaccess - create .htaccess files
        Paste in the following:
            RedirectMatch 404 /\.git
#### 11h - Install needed modules & packages
        $ source venv/bin/activate
        $ pip install httplib2 - to install httplib2 module in venv
        $ pip install requests - to install requests module in venv
        $ *sudo pip install flask-seasurf - to install flask.ext.seasurf (only seems to work when installed globally)
        $ sudo pip install --upgrade oauth2client - to install oauth2client.client (only seems to work when installed globally)
        $ sudo pip install sqlalchemy (only seems to work when installed globally)
        $ sudo apt-get install python-psycopg2 - to install the Python PostgreSQL adapter psycopg
### 10 - Install and configure PostgreSQL
        https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps
        nice: http://killtheyak.com/use-postgresql-with-django-flask/
        $ sudo apt-get install postgresql postgresql-contrib - to install PostgreSQL
        $ sudo su - postgres - to change to default user postgres
        $ psql - to connect to the system
        # \q then $ exit - to exit out of PostgreSQl and the postgres user
        *If warning "perl: warning: Setting locale failed." after command was executed
        *If after installation psql can not connect to server
        $ sudo nano /etc/postgresql/9.3/main/pg_hba.conf to check that no remote connections are allowed (default)
        http://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql
        $ sudo adduser catalog - needed linux user for psql (pw: fsnd)
        $ sudo su - postgres and $ psql -- to log into PostgreSQL
        *# \du - to list current roles and their attributes
        Add postgre user with password
            http://blog.trackets.com/2013/08/19/postgresql-basics-by-example.html &
            http://superuser.com/questions/769749/creating-user-with-password-or-changing-password-doesnt-work-in-postgresql
            # CREATE USER catalog WITH PASSWORD 'fsnd'; - to create user with LOGIN role and set a password
            # ALTER USER catalog CREATEDB; - to allow the user to create database tables
        # CREATE DATABASE catalog WITH OWNER catalog;
        # \c catalog - to connect to the database catalog
        # REVOKE ALL ON SCHEMA public FROM public; - to revoke all rights
        # GRANT ALL ON SCHEMA public TO catalog; - to grant only access to the catalog role
        $ sudo vim database_setup.py - to open the database setup file
        change the line starting with "engine" to:
            engine = create_engine('postgresql://catalog:fsnd@localhost/catalog')
        $ python database_setup.py - to create postgreSQL database schema
        $ sudo vim application.py - to change the same line in application.py respectively
        $ mv application.py __init__.py - rename application.py
#### 11i - Run application 
        $ sudo service apache2 restart
        open a browser and put in your public ip-address as url, e.g. 52.25.0.41 - if everything works the application should come up
        If getting an internal server error, check the Apache error files
            https://www.a2hosting.com/kb/developer-corner/apache-web-server/viewing-apache-log-files
            $ sudo tail -20 /var/log/apache2/error.log - to view the last 100 lines in the error log
            if a file like 'g_client_secrets.json' couldn't been found
                http://stackoverflow.com/questions/12201928/python-open-method-ioerror-errno-2-no-such-file-or-directory
#### 11j - Get OAuth-Logins Working
        http://discussions.udacity.com/t/oauth-provider-callback-uris/20460
        open http://www.hcidata.info/host2ip.cgi and receive the Host name for your public IP-address, e.g. for 52.25.0.41, its ec2-52-25-0-41.us-west-2.compute.amazonaws.com
        http://httpd.apache.org/docs/2.2/de/vhosts/name-based.html
        $ sudo nano /etc/apache2/sites-available/catalog.conf and paste in the following line below ServerAdmin:
            ServerAlias ec2-52-25-0-41.us-west-2.compute.amazonaws.com
        $ sudo a2ensite catalog - to enable the virtual host
        Google+
            Go to the project on the Developer Console: https://console.developers.google.com/project, navigate to APIs & auth > Credentials > Edit Settings and add your host name and public IP-address to your Authorized JavaScript origins and your host name + oauth2callback to Authorized redirect URIs, e.g. http://ec2-52-25-0-41.us-west-2.compute.amazonaws.com/oauth2callback
        Facebook
            Go on the Facebook Developers Site to My Apps https://developers.facebook.com/apps/, click on your App, go to Settings and fill in your public IP-Address including prefixed hhtp:// in the Site URL field
            To leave the development mode, so others can login as well, also fill in a contact email address in the respective field, "Save Changes", click on 'Status & Review' 
#### 11** - Install Monitor application Glances
        http://glances.readthedocs.org/en/latest/glances-doc.html#introduction &
        http://www.webhostbug.com/install-use-glances-ubuntudebian/
        $ sudo apt-get install python-pip build-essential python-dev
        $ sudo pip install Glances
    $ sudo pip install PySensors
