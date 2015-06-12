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
