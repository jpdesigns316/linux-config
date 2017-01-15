# Linux Server Configuration

This is project 5 of the [Udacity] (http://www.udacity.com) nanoDegree for [Full Stack Web Developer] (https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). The purpose of this project was to learn how to set up the network configuration via a Linux server. Network security is important to have and to be able to configure. As a Full Stack Web Developer, one would need to know how this process works without having a Linux System administrator setting things up.

*NOTE:* **DO NOT** navigate through the system as root. After creating the 'grader' account and giving him administrator permissions (use of sudo), immediately change to that user. Navigating a Linux system as root can be a security risk.

# Table of Contents
1. [Getting Started] (#getting-started)
2. [Updating, Upgrading, and Installing] (#updating-upgrading-and-unstalling)
3. [Configuring Timezone to UTC] (#configuring-timezone-to-utc)
4. [Modifying SSH port] (#modifying-SSH-port)
5. [Making Account Secure] (#making-account-secure)
6. [What is a Firewall] (#what-is-a-firewall) also include how to set up UFW.
7. [Configuring Protgres SQL] (#configuring-protgres-sql)
8. [Configure Web Server] (#configure-web-server)
9. [Sources Used] (#sources-used)

---
## Getting started

Copy the text in the note declaring the PGP key into a file called `udacity_key.rsa` in the `~/.ssh/` directory of your computer. Then you will be ready to connect to the server.
```
ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@35.167.85.110
```

### Creating users and configuring permissions:

To add a user you would use the ```adduser``` command.
```
adduser grader
```

grader password: Bradysucks

### Giving user permissions to use `sudo` command

Source: Udacity [Giving Sudo Access] (https://youtu.be/32u1qTPN5eQ), [Negus, 59-61] (#sources)

The file that is used to grant sudo permissions is  `/etc/sudoers` however, this file could be modified by upgrading the os or installing packages. There still is another method which could be used to give permissions, and that is via ```/etc/sudoer.d``` directory.

Using your favorite text editor create a new file:
```
nano grader
```
Type the following:
```
grader ALL=(ALL) NOPASSWD:ALL
```
Save the file and exit. Switch to the newly created 'grader' account that now how `sudo` permissions, and go to the home directory. You no longer need to be in root.
```
su grader
cd ~
```

## Updating, Upgrading, and Installing

Source: [Negus, pg 35-40] (#sources)

For  ```apt-get``` route use these commands:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get clean
```
The first command is used to update files. The second will upgrade the software,
and the last one will clean up the files. (Bresnahan 68-70, Negus 25-31)


Using `apt-get` you can install all the needed libraries on one line.
```
sudo apt-get install apache2 protgresql python-pip python-dev build-essential git-all ntp python-psycopg2 libpq-dev libapache2-mod-wsgi python-virtualenv
sudo pip install --upgrade pip
```
Source: [Stack Overflow] (http://stackoverflow.com/questions/28253681/you-need-to-install-postgresql-server-dev-x-y-for-building-a-server-side-extensi)

- _apache2_ - Apache Web Server
- _protgresql-9.4_ - Progres SQL v9.4
- _python-pip_ - Pip installation tool for Python (Instructions that follow are how to set it up)
-  _git-all_ - Git
-  _ntp_ - Network Time Protocol
- _python-psycopg_ - [PostgreSQL adpter for Python] (http://initd.org/psycopg/)
- _libpq-dev_ - Psycopg biniaries
- _python-virutalenv_ - The virtual environment used for installing modules to be used specifically for one app, and not be used to take up space on the hard drive.
- _libapache2-mod-wsgi_ - The Web Server Gateway Interface mod for use on Apache Server.

## Configuring Timezone to UTC

**Setting time via CLI**
Source: [Negus, pg 202] (#sources)
The Linux filesytsem store information about each timezone in ``/usr/share/timezone`` directory. To get this done you need to run the command:
```
sudo cp -f /usr/share/timezone/UTC/etc/localtime
```
The ```-f``` attribute will force the system to overwrite the current ```localtime``` file.

**Using to set Timezone to UTC**

Setting time via NTP (Network Time Protocol (Source: Negus 204)

Instead of using the /etc/ntp.conf` file to set to the time, and possibly create security risks ntpdate`. This would be a better command to use to set the time daily via ```cron```.  Use this command to set it with this command:

```
sudo ntpudate pool.ntp.org
```
## Modifying SSH port

To change the SSH port you need to edit the `/etc/ssh/sshd_config` file.
```
sudo nano /etc/ssh/sshd_config
```
Change the port from 22 to 2200

## Making new account secure

Source: [Dulaney, pg. 139-140] ] (#sources)

It is important to secure a connection. There is a method in which you can use to connect to a site and not have to keep putting in your password.  To do this requires you to set up a PGP key. Think of it as having a locked box in which you only have access to. This is your _private_ key.  This is used when connecting to a site. When you connect to it, the site will check the directories that store the _public_ key information. If your _private_ key matches it, then it will let you in.

It is also important to create a strong password. For the purposes of this project they are simple so that one can configure the Linux server. However, in the real world you should create a password that meets these requirements:
* They cannot contain the user's account name of parts of the user's full name that exceed tow consecutive characters.
* They must be at least  eight characters in length.
* They must have three of the following four set. (A-Z, a-z, 0-9, Non-Alpha Character

### Creating a PGP (Pretty Good Privacy)

Source: [Bersnahan, pg 557-558] (#sources)

To create a pgp key, on your local machine (not connected via ssh) in the `~/.ssh' directory type:
```
ssh-keygen
```
Enter `grader_key` for the pgp file. After confiuring your pgp file, you will need to put the public key on the server that you have set up by doing the following in the home directory:
```
mkdir -m 700 .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
On your local machine copy the content of the `pub` file of the keygen you created and paste into the ~/.ssh/authorized_keys` file and save it then type:
```
grader@ip-10-20-59-174:/home/grader$ chmod 644 .ssh/authorized_keys
```
End the current session and reconnect as the `grader` with the key that was just generated with the following:
```
ssh -i ~/.ssh/grader_key -p 2200 grader@ip-10-20-59-174
```
If prompted for a passphrase, then enter it.

### Simplify SSH Login

Here are some methods which could be used to connect to the server easier just make an `alias`:
```
alias grader="ssh -i ~/.ssh/grader_key -p 2200 grader@35.167.85.110"
```

## What is a Firewall?

Source: Udacity [Intro to UFW] (https://youtu.be/NYHrggI0wK0)

Configuring a firewall is a vital step in Network Security. Many hackers will try to gain access to machines through carelessly open ports. There are many [well-known ports] (https://en.wikipedia.org/wiki/List_of_TCP_and_UDP_port_numbers#Well-known_ports) there are used by various different applications. By knowing good network security yous should learn to close them. Some other tools that could be helpful to learn, in addition to UFW, are `netstat`, `iptables', 'nmap', `lsof`.

### Configuring the Unified Firewall

Source: Udacity [Intro to URW] (https://youtu.be/NYHrggI0wK0)

UFW is a firewall tool that is bundled with the Ubuntu operating system. It is an easy to use tool to open and close ports.

To close all incoming ports type:
```
sudo ufw default deny incoming
```
To open all outgoing ports type:
```
sudo ufw default allow outgoing
```

### To open ports

Source: Udacity [Configuring Ports in UFW] (https://youtu.be/n3PC04x09gc)

_Syntax:_
```
ufw [--dry-run] default allow|deny|reject [incoming|outgoing|routed]
```

```
sudo ufw allow 2200/tcp`
sudo ufw allow www
sudo ufw allow ntp
```

### After configuring the UFW enable it:
```
sudo ufw enable
```

## Configuring Protgres SQL

Create 'catalog' as a user so it could create and modify PSQL files.

Type the following command switches to a user which will allows create/modify actions on how protgres sql works

```
sudo su - protgres
```
Use the command `createuser` to create the user `catalog` to set that account up

```
createuser --interactive
Enter name of role to add: catalog
Shall the new role be a superuser? (y/n) n
Shall the new role be allowed to create databases? (y/n) y
Shall the new role be allowed to create more new roles? (y/n) n
```
Now you need to set up the database for catalog to use
```
psql
postgres=# CREATE DATABASE catalog OWNER catalog;
postgres=#\q
```

To exit the 'postgres' accout type `CTRL-D` or `exit`.

## Confiuring Webserver

There are two things that will be needed to run the Python app that was created in a prior project. They are Apache2 and Python-WSGI


### Setting up Python WSGI

Source: [Digital Ocean] (https://www.digitalocean.com/community/tutorials/how-to-deploy-a-flask-application-on-an-ubuntu-vpscd/)

#### What is WSGI?
In order for a Python program to be running innately, or without using Django, you need to create an interface between the program and the Apache web server.  WSGI stands for Web Server Gateway Interface. For more info on it, [click here] (https://www.python.org/dev/peps/pep-0333/)

### Installing and Configuring WSGI

After you have installed Apache, as per the instructions above, you need to get the latest version of python-wsgi. Once you get it installed, enable it.
```
sudo a2enmod wsgi
```
The directory that was created specifically by Apache is `/var/www` which is the directory that you will be working from. Change to that directory and install the repository,
```
cd /var/www
git clone https://github.com/jpdesigns316/item-catalog.git item-catalog
```

After this you will need to set up some files so that the WSGI will be able to work. Copy the item-catalog file
to `/etc/apache2/sites-available`

```
sudo mv /var/www/item-catalog/item-catalog.conf /etc/apache2/sites-available
```
To enable it type
```
sudo a2ensite item-catalog
```
Restart the Apache server
```
sudo apache2 restart
```

## Creating virtual environment
A virtual environment is important to create when running applications Python/Django. This is so you can create and environment that hold the dependencies for specific modules. One program could depend on one version of a file, whereas another could use a different version. Instead of pointing in multiple locations, just use a virtual environment. The following is how to set it up:

Source: [Hitchhiker's Guide to Python] (http://docs.python-guide.org/en/latest/dev/virtualenvs/)

If you do not currently have `venv` installed get it with the following:
```
sudo -H pip install virtualenv
```
If you already have it installed, change to the directory of the project that you wish to create the virtual environment for. For this project it is `/www/FlaskApp/src`, however it could be different depending on what you have created.
```
cd /www/item-catalog/src
sudo virutalenv venv
```
virtualenv syntax: `sudo virtualenv -p /usr/bin/python2.7 venv`
Now activate the newly created venv and install the modules you wish to use. After you have done so, exit the `venv`.
```
source venv/bin/activate
pip install -r requirements.txt
deactivate
```


---
## Sources used:

1. Bresnahan, Christine, and Richard Blum. [CompTIA Linux Powered by Linux Professional Institute Study Guide: Exams LX0-103 and Exam LX0-104.] (https://www.amazon.com/CompTIA-Security-Study-Guide-SY0-401/dp/1118875079/ref=sr_1_6?s=books&ie=UTF8&qid=1476121151&sr=1-6)
2. Negus, Christopher. [Ubuntu Linux Toolbox: 1000 Commands for Power Users.] (https://www.amazon.com/Ubuntu-Linux-Toolbox-Commands-Debian/dp/1118183525/ref=sr_1_1?ie=UTF8&qid=1475887174&sr=8-1&keywords=ubuntu+toolbox) Wiley, 2013.
3. Dulaney, Emmett, and Chuck Easttom. CompTIA Security Study Guide: SY0-401, 6th Edition. (https://www.amazon.com/CompTIA-Security-Study-Guide-SY0-401/dp/1118875079/ref=sr_1_2?ie=UTF8&qid=1476820442&sr=8-2)John Wiley & Sons, 2014.
