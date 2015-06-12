# FSND-P5_Linux-Server-Configuration

A Baseline installation of Ubuntu Linux on a virtual machine to host a Flask web application. This includes the installation of updates, securing the system from a number of attack vectors and installing/configuring web and database servers.

**Note:** The below step-by-step walkthrough is the solution to project 5 of the [Udacity Full Stack Web Developer Nanodegree][1] and deploys the [solution of project 3][2] on the virtual machine.

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
  `$ ssh -i ~/.ssh/udacity_key.rsa root@IP-Address`

### 3 & 4 - User Management: Create a new user and give user the permission to sudo
Source: [DigitalOcean][4]

1. Create a new user:  
  `$ adduser newuser`
2. Give new user the permission to sudo
  1. Open the sudo configuration:  
    `$ visudo`
  2. Add line below 'root ALL...':  
    `newuser ALL=(ALL:ALL) ALL`
  3. * List all users (Source: [Ask Ubuntu][5]):  
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


[1]: https://de.wikipedia.org/wiki/Flask "Wikipedia entry to Flask"
[2]: https://github.com/stueken/FSND-P3_Music-Catalog-Web-App "GitHub repository of an item catalog web app"
[3]: https://www.udacity.com/account#!/development_environment "Instructions for SSH access to the instance"
[4]: https://www.digitalocean.com/community/tutorials/how-to-add-and-delete-users-on-an-ubuntu-14-04-vps "How To Add and Delete Users on an Ubuntu 14.04 VPS"
[5]: http://askubuntu.com/questions/410244/a-command-to-list-all-users-and-how-to-add-delete-modify-users "How to list, add, delete and modify users"
