
# FSND-Linux-Server-Configuration
This document details my process for setting up and configuring a Linux server using AWS Lightsail, Ubuntu, Apache, and WSGI to host a PostgreSQL database-driven python catalog website.  As with most websites and web services, processes and user interfaces change. Some of the instructions here when referencing with Amazon Web Services interfaces may or may not be valid.

**IP Address:** 3.86.203.1
**SSH Port:** 2200
**URL to Application:** http://fsnd.3.86.203.1.xip.io/

## Amazon Lightsail Setup
1. Access Amazon Lightsail's website (https://lightsail.aws.amazon.com), create an account or log into an existing account.
2. "Create an instance" and select Linux/Unix, OS Only, and Ubuntu 16.04 LTS. 
3. Choose the hosting plan desired, give the instance a name, and select the "Create instance" button at the bottom.
4. Once your instance is started, select the Networking tab.  Here, along with the main page, you will find the public IP of the server.  

## Linux Configuration
On the Connect page of the Lightsail instance, click the "Connect using SSH" button. A window will appear and connect to the instance using the username "Ubuntu."

### Modify Account Permissions
* To allow Udacity's reviewer to access the server, create a new user account called "grader."  
```
$ sudo adduser grader
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Changing the user information for grader
Enter the new value, or press ENTER for the default
Full Name []: Udacity Grader
Room Number []:
Work Phone []:
Home Phone []:
Other []:
Is the information correct? [Y/n] Y
$ 
```
* Give the *grader* account *sudo* access: ```$ sudo usermod -aG sudo grader```
* Change over to the *root* account  to modify the config and remove root access.
 ```
$ sudo -s
root@ip-xx-xxx-xxx-x:$ nano /etc/ssh/sshd_config
```
* Change the line ```PermitRootLogin prohibit-password``` to ```PermitRootLogin no```
* Use CTRL+X to save and close the file.
* Restart SSH ```$ sudo service ssh restart```
*  Get out of *root* and access the *grader* account ```$ su grader```

### Change SSH Port
* Go back into the previous file to change the standard SSH port (22) to a non-standard port (2200).
```$ sudo nano /etc/ssh/sshd_config```
* Change ```Port 22``` to ```Port 2200```
* Use CTRL+X and Y to save and close.
 ***NOTE**: Check to make sure the firewall (see below) is either disabled first, or that port 2200 is allowed.*
* Restart SSH ```$ sudo service ssh restart```
* Log out of SSH ```$ logout```
* Log back in using the new port ```$ ssh grader@[server's public IP] -p 2200``` 

### Set Up Firewall
* Start by checking the firewall's status ```$ sudo ufw status numbered```
* Default deny incoming: ```$ sudo ufw default deny incoming```
* Default allow outgoing: ```$ sudo ufw default allow outgoing```
* Allow WWW on port 80: ```$ sudo ufw allow 80/tcp```
* Allow NTP on port 123: ```$ sudo ufw allow 123```
* Allow SSH on port 2200: ```$ sudo ufw allow 2200/tcp```
* Check the numbered list of ports open on the firewall and remove unnecessary ports *(example below)*
```
$ sudo ufw status numbered

Status: inactive

To 		Action  		From
-- 		------  		----
[ 1] 	80/tcp 			ALLOW IN  Anywhere
[ 2] 	123  			ALLOW IN  Anywhere
[ 3] 	2200/tcp 		ALLOW IN  Anywhere
[ 4] 	22				ALLOW IN  Anywhere
[ 5] 	123 (v6) 		ALLOW IN  Anywhere (v6)
[ 6] 	2200/tcp (v6)	ALLOW IN  Anywhere (v6)
[ 7] 	80/tcp (v6)  	ALLOW IN  Anywhere (v6)
[ 8]	22				ALLOW IN  Anywhere (v6)
```
```
$ sudo ufw delete 4
Deleting:
allow 22
Proceed with operation (y|n)? y
Rule deleted
```
* Recheck the status of the firewall as the numbers may change with each delete or add and to remove additional rules (ex. 22 (v6))
```
$ sudo ufw status numbered

Status: inactive

To 		Action  		From
-- 		------  		----
[ 1] 	80/tcp 			ALLOW IN  Anywhere
[ 2] 	123  			ALLOW IN  Anywhere
[ 3] 	2200/tcp 		ALLOW IN  Anywhere
[ 4] 	123 (v6) 		ALLOW IN  Anywhere (v6)
[ 5] 	2200/tcp (v6)	ALLOW IN  Anywhere (v6)
[ 6] 	80/tcp (v6)  	ALLOW IN  Anywhere (v6)
[ 7]	22				ALLOW IN  Anywhere (v6)
```
```
$ sudo ufw delete 7
Deleting:
allow 22
Proceed with operation (y|n)? y
Rule deleted (v6)
```
**WARNING:** Review your firewall settings again ```$ sudo ufw status numbered```, especially to ensure that your SSH port will be open once you enable the firewall. Your connection is reliant on the rules being correct.
* Enable your firewall (be careful): ```$ sudo ufw enable```

### Force Key-based Authentication
Once you have properly enabled your firewall, you need to remove access to SSH through the use of a password. This requires key-based authentication.  
* To generate you public and private SSH key pair, follow the instructions at [https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html](https://confluence.atlassian.com/bitbucketserver/creating-ssh-keys-776639788.html)
* Take note of the name and location of the public key and, in a web browser, access your AWS Lightsail account.
* In the Account area, select *SSH Keys* and click the *Upload new* icon.
* Upload the .pub key generated in the previous steps.
* In your terminal, edit the permissions on the /home/grader/.ssh folder ```$ sudo chmod 700 ~/.ssh``` and the authorized_keys file inside ```$ sudo chmod 644 ~/.ssh/authorized_keys``` .
* Once again, go back into the sshd_config file to force key-based authentication: ```sudo nano /etc/ssh/sshd_config```
* Edit  ```PasswordAuthentication yes``` to ```PasswordAuthentication no```
* Save and exit the sshd_config file using CTRL+X and Y.

### SSH Service Restart
Once you have reviewed all of the changes to users, login, and authentication, restart the the SSH service ```$ sudo service ssh restart```.

### Install/Update/Remove Packages
```$ sudo apt-get install git```
```$ sudo apt-get install python-pip```
```$ sudo pip install virtualenv```

### Configure Timezone for UTC
Set the timezone to UTC with the following command: ```$ sudo timedatectl set-timezone UTC```. Then download and install NTP ```$ sudo apt-get install ntp```. This will make sure to regularly adjust the time to endure that no large time gaps occur. 

## Apache and WSGI Config
Install Apache and configure it to run your Python application using mod_wsgi.
```$ sudo apt-get install apache2```
```$ sudo apt-get install libapache2-mod-wsgi python-dev```
* Confirm that Apache is successfully installed by going to your AWS Lightsail instance's public IP in your web browser.  You should see the default Apache "It works" page. 
* Enable mod_wsgi ```$ sudo a2enmod wsgi``` 
* Restart the Apache web server by ```$ sudo service apache2 restart```
* Back in your SSH terminal, create a new folder for your Python website.
```
$ cd /var/www
$ sudo mkdir catalog
```
* Make the grader user the owner and make it a recursive (-R) ownership. Then cd to that directory
```
$ sudo chown -R grader:grader catalog
$ cd catalog
```
* Clone your GitHub repository containing your catalog website.
```$ git clone [GitHub Repository URL] catalog```
*Note: My repository is located at https://github.com/espressoj/fsnd-catalog-project.git*
*Note: The location of the cloned files will now be /var/www/catalog/catalog/*
* Create a .wsgi file so that the python website can be produced by Apache.
```
$ cd /var/www/catalog/
$ sudo nano catalog.wsgi
```
* Enter the following into the catalog.wsgi file.
```
#!/usr/bin/python
import sys
import logging
logging.basicConfig(stream=sys.stderr)
sys.path.insert(0, "/var/www/catalog/")

from catalog import app as application
application.secret_key = '[CHANGE THIS TO A LARGE RANDOM STRING - APP SECRET KEY]'
```
* Change the primary python file of the website from "views.py" to ```__init__.py```
```$ mv views.py __init__.py```
* Install python-pip
```$ sudo apt-get install python-pip```
* Install and configure the virtual environment (venv).
 ```
$ sudo pip install virtualenv
$ sudo virtualenv venv
$ source venv/bin/activate
$ sudo chmod -R 777 venv
```
*Note: For more information on virtual environments see https://www.pythonforbeginners.com/basics/how-to-use-python-virtualenv and/or http://flask.pocoo.org/docs/0.12/installation/*
* Install the necessary Python packages to run your website.
```
$ sudo pip install Flask httplib2 oauth2client sqlalchemy psycopg2 sqlaclemy_utils requests
$ sudo pip install json logging datetime itsdangerous
```
* Adjust the client_secrets.json path in the ```__init__.py``` file so that it can be found using the full path of its new location. Modify it as follows:
```/var/www/catalog/catalog/client_secrets.json```
* Change references to the app running on an incorrect IP or port in your ```__init__.py``` file.
```app.run(host='0.0.0.0', port=8000) ``` to ```app.run()```
* Access the virtual hosts directory and add the catalog.conf file.
```
$ cd /etc/apache2/sites-available
$ sudo nano catalog.conf
```
Add the following code to that catalog.conf file:
```
<VirtualHost *:80>
	ServerName [REPLACE ME WITH YOUR SERVER'S EXTERNAL IP]
	ServerAlias http:[REPLACE ME]
	ServerAdmin admin@[EXTERNAL IP]
	WSGIDaemonProcess catalog python-path=/var/www/catalog:/var/www/catalog/venv/lib/python2.7/site-packages
	WSGIProcessGroup catalog
	WSGIScriptAlias / /var/www/catalog/catalog.wsgi
	WSGIScriptReloading On
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
	LogLevel info
	CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```
* Replace the [BRACKETED INFORMATION] above with your own server information.
* You can find the ServerAlias using the following: http://www.hcidata.info/host2ip.cgi.
* Use CTRL+X and "Y" to save and close.
 
## References and Resources...
* https://github.com/jaysimonkay/Linux-Configuration
* https://www.howtoforge.com/tutorial/how-to-run-python-scripts-with-apache-and-mod_wsgi-on-ubuntu-18-04/
* http://flask.pocoo.org/docs/0.12/deploying/mod_wsgi/
* https://www.jakowicz.com/flask-apache-wsgi/
