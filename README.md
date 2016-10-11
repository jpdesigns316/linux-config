# Linux Server Configuration

This is project 5 of the [Udacity] (http://www.udacity.com) nanoDegree for [Full Stack Web Developer] (https://www.udacity.com/course/full-stack-web-developer-nanodegree--nd004). The purpose of this proect was to learn how to set up the network configuration via a Linux server. Network security is important to have and to be able to configure. As a Full Stack Web Developer, one would neeed to know how this process works without having a Linux System Administartor setting things up.

*NOTE:* **DO NOT** navigate through the system as root. After creating the 'grader' account and giving him adminitrator permissions (use of sudo), immediately change to that user. Navigating a Linux system as root can be a security risk.

# Table of Contents
1. [Getting Started] (#Getting)
2. [Updating, Upgrading, and Installing] (#Updating)
3. [Configuring Timezone to UTC] (#configuring)
4. [Modifying SSH port] (#modifying)
5. [What is a Fire Wall?] {#what) also include how to set up UFW.
6. [Prepping Linux Filesystem to run Python web server] (#prepping)
7. [Sources Used] (#sources)

---
## Getting started

Copy the text in the note declaring the PGP key into a file called `udacity_key.rsa` in the `~/.ssh/` directory of your computer. Then you will be ready to connect to the server.
```
ssh -i ~/.ssh/udacity_key.rsa -p 2200 root@54.71.92.238
```


### Creating users and configuring permissions:


To add a user you would use the ```adduser``` command.
```
adduser grader
```

### Giving user permissions to use `sudo` command

Source: Udacity [Giving Sudo Access] (https://youtu.be/32u1qTPN5eQ), (Negus 59-61)

The file that is used to grant sudo permissions is  `/etc/sudoers` however, this file could be modified by updrading the os or installing packages. There still is another method which could be used to give permissions, and that is via ```/etc/sudoer.d``` directory.

Using your favorite text editor create a new file:
```
nano grader
```
Type the following:
```
grader ALL=(ALL) NOPASSWD:ALL
```
Save the file and exit. Switch to the newly created 'grader' account that now how `sudo` permissions, and go to the home directoy. You no longer need to be in root.
```
su grader
cd ~
```

## Updating, Upgrading, and Installing

Source: (Negus, 35-40)

For  ```apt-get``` route use these commands:
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get clean
```
The first command is used to update files. The second will upgrade the software,
and the last one will clean up the files. (Bresnahan 68-70, Negus 25-31)


The following will install the needed packages for this project
```
sudo apt-get install apache2
sudo apt-get install protgresql-9.9
sudo apt-get install python-pip python-dev build-essential
sudo pip install --upgrade pip
sudo apt-get install git-all
sudo apt-get install ntp
```
- _apache2_ - Apache Web Server
- _protgresql-9.4_ - Progres SQL v9.4
- _python-pip_ - Pip installation tool for Python (Instructions that follow are how to set it up)
-  _git-all_ - Git
-  _ntp_ - Network Time Protocol

Finally use the `git` command to put a copy of your project on the server.
```
cd /www
git clone https://github.com/jpdesigns316/item-catalog.git item-catalog
```
## Configuring Timezone to UTC

**Setting time via CLI**
Source: (Negus 202)
The Linux filesytem store infomration about each timezone in ``/usr/share/timezone`` directory. To get this done you need to run the command:
```
sudo cp -f /usr/share/timezone/UCT /etc/localtime
```
The ```-f``` attribute will force the system to overwrite the current ```localtime``` file.

**Using to set Timezone to UTC**

Setting time via NTP (Network Time Protocol (Source: Negus 204)

Instead of using the ```/etc/ntp.conf``` file to set to the time, and possibly create security risks ```ntpdate```. This would be a better command to use to set the time daily via ```cron```.  Use this command to set it with this command:

```
sudo ntpudate pool.ntp.org
```
## Modifying SSH port

To change the SSH port you need to edit the `/etc/ssh/ssh_config` file.
```
sudo nano /etc/ssh/ssh_config
```
Change the port from 22 to 2200

## Making new account secure

### Creating a PGP (Pretty Good Privacy)

Source: (Bersnahan, pg 557-558)


To create a pgp key, on your local machine (not connected via ssh) in the ```~/.ssh`` directory type:
```
ssh-keygen
```
Enter `grader_key` for the pgp file. After confiuring your pgp file, you will need to put the public key on the server that you have set up by doing the following in the home directory:
```
mkdir -m 700 .ssh
touch .ssh/authorized_keys
nano .ssh/authorized_keys
```
On your local machine copy the content of the `pub` file of the keygen you created and paste into the ```~/.ssh/authorized_keys`` file and save it then type:
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
alias grader="ssh -i ~/.ssh/grader_key -p 2200 grader@ip-10-20-59-174"
```

## What is a Fire Wall?

Source: Udacity [Intro to URW] (https://youtu.be/NYHrggI0wK0)

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

## Prepping Linux Filesystem to run Python web server





### Creating virtual environment
A virtual envrionment is ususually important to create when running applications Python/Djangos. This is so you can create and environment that hold the dependencies for specific modules. One program could depend on one version of a file, whereas another could use a different version. Instead of pointing in multiple locations, just use a vitural environment. The following is how to set it up:

Source: [Hitchhiker's Guide to Python] (http://docs.python-guide.org/en/latest/dev/virtualenvs/)

If you do not currently have `venv` installed get it with the following:
```
sudo pip install virtualenv
```
If you already have it installed, change to the directory of the project that you wish to create the virtual environment for. For this project it is `/www/item-catalog/src`, however it could be different depending on what you have created.
```
cd /www/item-catalog/src
virutalenv
```
virtualenv syntax: `venv -p [location of Python version] venv`
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
