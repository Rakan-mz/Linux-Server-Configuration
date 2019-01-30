# Linux Server Configuration

This is the final project for Udacity's [Full Stack Web Developer Nanodegree](https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004).
## Goal of project
turning a brand-new, bare bones, Linux server into the secure and efficient web application host your applications need. And how to install and configure a web and database server and actually host a web application.
. 

This page explains how to secure and set up a Linux distribution on a virtual machine, install and configure a web and database server to host a web application. 
- The Linux distribution is [Ubuntu](https://www.ubuntu.com/download/server) 16.04 LTS.
- The virtual private server is [Amazon Lighsail](https://lightsail.aws.amazon.com/).
- The web application is my [Item Catalog project](https://github.com/boisalai/udacity-catalog-app) created earlier in this Nanodegree program.
- The database server is [PostgreSQL](https://www.postgresql.org/).
- My local machine is a MacBook Pro (Mac OS X 10_12_6).

You can visit http://13.232.176.48.xip.io/ for the website deployed.



## Get a server

### Step 1: Start a new Ubuntu Linux server instance on Amazon Lightsail 

- Login into [Amazon Lightsail](https://lightsail.aws.amazon.com/ls/webapp/home/resources) using an Amazon Web Services account.
- Once you are login into the site, click `Create instance`. 
- Choose `Linux/Unix` platform, `OS Only` and  `Ubuntu 16.04 LTS`.
- Choose a instance plan (I took the cheapest, $3/month).
- Keep the default name provided by AWS or rename your instance.
- Click the `Create` button to create the instance.
- Wait for the instance to start up unti finishing padding .

**Reference**
- ServerPilot, [How to Create a Server on Amazon Lightsail](https://serverpilot.io/community/articles/how-to-create-a-server-on-amazon-lightsail.html).


### Step 2: SSH into the server

- From the `Account` menu on Amazon Lightsail, click on `SSH keys` tab and download the Default Private Key.
- Move this private key file named `LightsailDefaultPrivateKey-*.pem` into the local folder `~/.ssh` and rename it `lightsail_key.rsa`.
- In your terminal, type: `chmod 600 ~/.ssh/lightsail_key.rsa`.
- To connect to the instance via the terminal: `ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.232.176.48`, 
  where `13.232.176.48` is the public IP address of the instance.

***
Public IP address is 13.232.176.48.
ssh -i ~/.ssh/lightsail_key.rsa ubuntu@13.232.176.48
***

## Secure the server

### Step 3: Update and upgrade installed packages

```
sudo apt-get update
sudo apt-get upgrade
```


### Step 4: Change the SSH port from 22 to 2200

- Edit the `/etc/ssh/sshd_config` file: `sudo nano /etc/ssh/sshd_config`.
- Change the port number on line 5 from `22` to `2200`.
- Save and exit using CTRL+X and confirm with Y.
- .
- Restart SSH: `sudo service ssh restart`. on Amazon Lightsail firewall
- Click on the Manage option of the Amazon Lightsail Instance, go to Networking tab,
and then change the firewall configuration to match the internal firewall settings
above.
![IMG_7067.JPG](https://raw.githubusercontent.com/Heba-ahmad/Linux-Server-Configuration/master/image/firewall1.png)
- Allow ports 80(TCP), 123(UDP), and 2200(TCP), and deny the default port 22.
![Capture.JPG](https://www.dropbox.com/s/hrbvustj8e43viv/Capture.JPG?dl=0&raw=1)

- From your local terminal, run: `ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@13.232.176.48`, where `13.232.176.48` is the public IP address of the instance.

***
Public IP address is 13.59.39.163.
ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@13.59.39.163
***
### Step 5: Configure the Uncomplicated Firewall (UFW)

- Configure the default firewall for Ubuntu to only allow incoming connections for SSH (port 2200), HTTP (port 80), and NTP (port 123).
  ```
  sudo ufw status                  # The UFW should be inactive.
  sudo ufw default deny incoming   # Deny any incoming traffic.
  sudo ufw default allow outgoing  # Enable outgoing traffic.
  sudo ufw allow 2200/tcp          # Allow incoming tcp packets on port 2200.
  sudo ufw allow www               # Allow HTTP traffic in.
  sudo ufw allow 123/udp           # Allow incoming udp packets on port 123.
  sudo ufw deny 22                 # Deny tcp and udp packets on port 53.
  ```

- Turn UFW on: `sudo ufw enable`. The output should be like this:
  ```
  Command may disrupt existing ssh connections. Proceed with operation (y|n)? y
  Firewall is active and enabled on system startup
  ```

- Check the status of UFW to list current roles: `sudo ufw status`. The output should be like this:

  ```
  Status: active
  
  To                         Action      From
  --                         ------      ----
  2200/tcp                   ALLOW       Anywhere                  
  80/tcp                     ALLOW       Anywhere                  
  123/udp                    ALLOW       Anywhere                  
  22                         DENY        Anywhere                  
  2200/tcp (v6)              ALLOW       Anywhere (v6)             
  80/tcp (v6)                ALLOW       Anywhere (v6)             
  123/udp (v6)               ALLOW       Anywhere (v6)             
  22 (v6)                    DENY        Anywhere (v6)
  ```

- Exit the SSH connection: `exit`.

- Click on the `Manage` option of the Amazon Lightsail Instance, 
then the `Networking` tab, and then change the firewall configuration to match the internal firewall settings above.




# References
- Official Ubuntu Documentation, [UFW - Uncomplicated Firewall](https://help.ubuntu.com/community/UFW).
- TechRepublic, [How to install and use Uncomplicated Firewall in Ubuntu](https://www.techrepublic.com/article/how-to-install-and-use-uncomplicated-firewall-in-ubuntu/).






### Step 5.3: Updated packages to most recent versions

  ```
  sudo apt-get update
  sudo apt-get dist-upgrade
  sudo shutdown -r now
  ```

- Logged back in, and I now see this message:
  ```
  Alains-MBP:udacity-linux-server-configuration boisalai$ ssh -i ~/.ssh/lightsail_key.rsa -p 2200 ubuntu@13.59.39.163
  Welcome to Ubuntu 16.04.3 LTS (GNU/Linux 4.4.0-1039-aws x86_64)

   * Documentation:  https://help.ubuntu.com
   * Management:     https://landscape.canonical.com
   * Support:        https://ubuntu.com/advantage

    Get cloud support with Ubuntu Advantage Cloud Guest:
      http://www.ubuntu.com/business/services/cloud

  0 packages can be updated.
  0 updates are security updates.

  Last login: Tue Oct 31 06:35:28 2017 from 24.201.154.77
  ubuntu@ip-172-26-0-7:~$ 
  ```



### Step 6

## Give `grader` access


### Step 6: Create a new user account named `grader`

- While logged in as `ubuntu`, add user: `sudo adduser grader`. 
- Enter a password (twice) and fill out information for this new user.


### Step 7: Give `grader` the permission to sudo

- Edits the sudoers file: `sudo visudo`.
- Search for the line that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `grader` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `grader` has sudo permissions. Run `su - grader`, enter the password, 
run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for grader on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User grader may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
      (ALL : ALL) ALL
  ```




### Step 8: Create an SSH key pair for `grader` using the `ssh-keygen` tool

- On the local machine:
  - Run `ssh-keygen`
  - Enter file in which to save the key (I gave the name `grader_key`) in the local directory `~/.ssh`
  - Enter in a passphrase twice. Two files will be generated (  `~/.ssh/grader_key` and `~/.ssh/grader_key.pub`)
  - Run `cat ~/.ssh/grader_key.pub` and copy the contents of the file
  - Log in to the grader's virtual machine
- On the grader's virtual machine:
  - Create a new directory called `~/.ssh` (`mkdir .ssh`)
  - Run `sudo nano ~/.ssh/authorized_keys` and paste the content into this file, save and exit
  - Give the permissions: `chmod 700 .ssh` and `chmod 644 .ssh/authorized_keys`
  - Check in `/etc/ssh/sshd_config` file if `PasswordAuthentication` is set to `no`
  - Restart SSH: `sudo service ssh restart`
- On the local machine, run: `ssh -i ~/.ssh/grader_key -p 2200 grader@13.59.39.163`.


    



## Prepare to deploy the project

### Step 9: Configure the local timezone to UTC

- While logged in as `grader`, configure the time zone: `sudo dpkg-reconfigure tzdata`. You should see something like that:

  ```
  Current default time zone: 'America/Montreal'
  Local time is now:      Thu Oct 19 21:55:16 EDT 2017.
  Universal Time is now:  Fri Oct 20 01:55:16 UTC 2017.
  ```

### Step 10: Install and configure Apache to serve a Python mod_wsgi application

- While logged in as `grader`, install Apache: `sudo apt-get install apache2`.
- Enter public IP of the Amazon Lightsail instance into browser. If Apache is working, you should see:

![سس.png](https://www.dropbox.com/s/0svwutwdq564g4w/%D8%B3%D8%B3.png?dl=0&raw=1)

-  My project is built with Python 2. So, I need to install the Python2 mod_wsgi
package:
`sudo apt-get install libapache2-mod-wsgi python-dev`.
for python3 run the following command:
sudo apt-get install libapache2-mod-wsgi-py3`

- Enable `mod_wsgi` using: `sudo a2enmod wsgi`.


### Step 11: Install and configure PostgreSQL

- While logged in as `grader`, install PostgreSQL:
 `sudo apt-get install postgresql`.
- PostgreSQL should not allow remote connections. In the  `/etc/postgresql/9.5/main/pg_hba.conf` file, you should see:
  ```
  local   all             postgres                                peer
  local   all             all                                     peer
  host    all             all             127.0.0.1/32            md5
  host    all             all             ::1/128                 md5
  ```
** Create a new PostgreSQL for catalog **

- Switch to the `postgres` user: `sudo su - postgres`.
- Open PostgreSQL interactive terminal with `psql`.
- Create the `catalog` user with a password and give them the ability to create databases:
  ```
  postgres=# CREATE ROLE catalog WITH LOGIN PASSWORD 'catalog';
  postgres=# ALTER ROLE catalog CREATEDB;
  ```

- List the existing roles: `\du`. The output should be like this:
  ```
                                     List of roles
   Role name |                         Attributes                         | Member of 
  -----------+------------------------------------------------------------+-----------
   catalog   | Create DB                                                  | {}
   postgres  | Superuser, Create role, Create DB, Replication, Bypass RLS | {}
  ```

- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.
- Create a new Linux user called `catalog`: `sudo adduser catalog`. Enter password and fill out information.
- Give to `catalog` user the permission to sudo. Run: `sudo visudo`.
- Search for the lines that looks like this:
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  ```

- Below this line, add a new line to give sudo privileges to `catalog` user.
  ```
  root    ALL=(ALL:ALL) ALL
  grader  ALL=(ALL:ALL) ALL
  catalog  ALL=(ALL:ALL) ALL
  ```

- Save and exit using CTRL+X and confirm with Y.
- Verify that `catalog` has sudo permissions. Run `su - catalog`, enter the password, run `sudo -l` and enter the password again. The output should be like this:

  ```
  Matching Defaults entries for catalog on ip-172-26-13-170.us-east-2.compute.internal:
      env_reset, mail_badpass,
      secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin
  
  User catalog may run the following commands on ip-172-26-13-170.us-east-2.compute.internal:
      (ALL : ALL) ALL
  ```

- While logged in as `catalog`, create a database: `createdb catalog`.
- Run `psql` and then run `\l` to see that the new database has been created. The output should be like this:
  ```
                                    List of databases
     Name    |  Owner   | Encoding |   Collate   |    Ctype    |   Access privileges   
  -----------+----------+----------+-------------+-------------+-----------------------
   catalog   | catalog  | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   postgres  | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | 
   template0 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
   template1 | postgres | UTF8     | en_US.UTF-8 | en_US.UTF-8 | =c/postgres          +
             |          |          |             |             | postgres=CTc/postgres
  (4 rows)
  ```
- Exit psql: `\q`.
- Switch back to the `grader` user: `exit`.

**Reference**
- DigitalOcean, [How To Secure PostgreSQL on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps).



### Step 12: Install git

- While logged in as `grader`, install `git`: `sudo apt-get install git`.

## Deploy the Item Catalog project

### Step 13.1: Clone and setup the Item Catalog project from the GitHub repository 

- While logged in as grader, create a directory for our catalog application and make
the user grader the owner.


   ``` 
$ cd /var/www
$ sudo mkdir catalog
$ sudo chown -R grader:grader catalog
$ cd catalog
   ``` 

- Change to that directory and clone the catalog project:<br>
`sudo git clone https://github.com/Rakan-mz/Item-Catalog catalog`.
- From the `/var/www` directory, change the ownership of the `catalog` directory to `grader` using: `sudo chown -R grader:grader catalog/`.
- Create the .wsgi file by` $ sudo nano catalog.wsgi `and make sure your secret key
matches with your project secret key:
***
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")
from catalog import app as application
application.secret_key = 'super_secret_key'
***

 Create our virtual environment, make sure you are in /var/www/catalog.
 ```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
(venv) will appear like this (venv)grader@ip00-00-00-00:/var/www/catalog$ While our
virtual environment is activated we need to install all packages required for our Flask
application:
  ```
$ sudo apt-get install python-pip
$ sudo pip install flask
$ sudo pip install httplib2 oauth2client sqlalchemy psycopg2
$ sudo pip install requests
$ sudo pip install --upgrade oauth2client
$ sudo apt-get install libpq-dev
$ sudo pip install sqlalchemy_utils
  ```
- Change to the `/var/www/catalog/catalog` directory.
- Rename the `application.py` file to `__init__.py` using: `mv application.py __init__.py`.

- In `__init__.py`, replace line 27:
  ```
  # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
  ```
 Update path of client_secrets.json file:
`'/var/www/catalog/catalog/client_secrets.json'`

- In `database.py`,`teammenu.py`, replace line 9:
   ```
   # engine = create_engine("sqlite:///catalog.db")
   engine = create_engine('postgresql://catalog:PASSWORD@localhost/catalog')
   ``` 

### Step 13.2: Authenticate login through Google

- Go to [Google Cloud Plateform](https://console.cloud.google.com/).
- Click `APIs & services` on left menu.
- Click `Credentials`.
- Create an OAuth Client ID (under the Credentials tab), and add http://13.232.176.48  and 
http://ec2-13.232.176.48 .us-east-2.compute.amazonaws.com as authorized JavaScript 
origins.
- Add http://ec2-13.232.176.48 .us-east-2.compute.amazonaws.com/oauth2callback 
as authorized redirect URI.
- Download the corresponding JSON file, open it et copy the contents.
- Open `/var/www/catalog/catalog/client_secret.json` and paste the previous contents into the this file.



### Step 14.2: Set up and enable a virtual host

- Add the following line in `sudo nano /etc/apache2/mods-enabled/wsgi.conf` 

  Paste in the following:


-following lines to configure the virtual host:
Paste in the following:
  ```
  <VirtualHost *:80>
	  ServerName 13.59.39.163
    ServerAlias ec2-13-59-39-163.us-west-2.compute.amazonaws.com
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
  
  
  If you need help finding your servers hostname use the following links and paste the IP
address. Save and quit nano:
- https://whatismyipaddress.com/ip-hostname
- http://www.nmonitoring.com/ip-to-domain-name.html

### Step 14.5: Disable the default Apache site

- Disable the default Apache site: `sudo a2dissite 000-default.conf`. 
The following prompt will be returned:

  ```
  Site 000-default disabled.
  To activate the new configuration, you need to run:
    service apache2 reload
  ```

- Reload Apache: `sudo service apache2 reload`.

### Step 14.6: Launch the Web Application

- Change the ownership of the project directories: `sudo chown -R www-data:www-data catalog/`.
- Restart Apache again: `sudo service apache2 restart`.
- Open your browser to http://13.232.176.48.xip.io/ or http://ec2-13.232.176.48.xip.io/.us-east-2.compute.amazonaws.com.




## Useful commands

 - To get log messages from Apache server: `sudo tail /var/log/apache2/error.log`.
 - To restart Apache: `sudo service apache2 restart`.

## Other Helpful Resources

- DigitalOcean [How To Deploy a Flask Application on an Ubuntu VPS](https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vps)
- GitHub Repositories https://raw.githubusercontent.com/Heba-ahmad/Linux-Server-Configuration/master/image/firewall1.png
  - [adityamehra/udacity-linux-server-configuration](https://github.com/adityamehra/udacity-linux-server-configuration)
  - [anumsh/Linux-Server-Configuration](https://github.com/anumsh/Linux-Server-Configuration)
  - [bencam/linux-server-configuration](https://github.com/bencam/linux-server-configuration)
  - [iliketomatoes/linux_server_configuration](https://github.com/iliketomatoes/linux_server_configuration)
   - https://github.com/stueken/FSND-P5_Linux-Server-Configuration/blob/master/README.md
    - https://github.com/boisalai/udacity-catalog-app
  - https://github.com/stueken/FSND-P5_Linux-Server-Configuration/blob/master/README.md
  - https://github.com/stueken/FSND-P5_Linux-Server-Configuration
  - https://github.com/chuanqin3/udacity-linux-configuration

