readme .md
# FSND-Linux-Server-Configuration
This document details my process for setting up and configuring a Linux server using AWS Lightsail, Ubuntu, Apache, and WSGI to host a PostgreSQL database-driven python catalog website.  As with most websites and web services, processes and user interfaces change. Some of the instructions here when referencing with Amazon Web Services interfaces may or may not be valid.

## Amazon Lightsail Setup
1. Access Amazon Lightsail's website (https://lightsail.aws.amazon.com), create an account or log into an existing account.
2. "Create an instance" and select Linux/Unix, OS Only, and Ubuntu 16.04 LTS. 
3. Choose the hosting plan desired, give the instance a name, and select the "Create instance" button at the bottom.
4. Once your instance is started, select the Networking tab.  Here, along with the main page, you will find the public IP of the server.  

## Linux Configuration
On the Connect page of the Lightsail instance, click the "Connect using SSH" button. A window will appear and connect to the instance using the username "Ubuntu."
### Modify Account Permissions
Add grader account - add to sudoers
Change to root user to modify config and remove root access
### Set Up Firewall
* Change SSH port
* Default allow outgoing
* Default deny incoming
* Allow 80
* allow 123
* Allow 2200
* enable (be careful)

### Force Key-based Authentication
* ```sudo nano /etc/ssh/sshd_config```
Change PasswordAuthentication from yes to no


### Install/Update/Remove Packages
