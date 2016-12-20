# Setting up your own server
Written by Jukka Juslin and [Nico Hagelberg](https://github.com/nicougit/)

How to setup your own server as a student for free using Digital Ocean and Namecheap. Github offers a student pack, which includes $50 worth of credits to Digital Ocean and a free .me domain for one year from Namecheap. Github student pack also has other benefits that you can use.

Digital Ocean and Namecheap have been widely used already before this offer. Digital Ocean is a cloud VPS (Virtual Private Server) provider and Namecheap offers domain registeration and web hosting. 

This guide has some tips on how to setup your own Digital Ocean server to host your web applications. The instructions in this guide require at least some knowledge of basic Unix terminal commands.

## Additional resources

* [Command line basics - Tero Karvinen](http://terokarvinen.com/2009/command-line-basics-4)
* Some [Digital Ocean community](https://www.digitalocean.com/community) tutorials are good resources, but some of them are outdated or just plain wrong. You should always look for multiple sources when looking for information.

## Getting your student pack
Go to [https://education.github.com/pack](https://education.github.com/pack) and click "get your pack". Getting the pack might take couple of hours or might be ready instantly.

Sign in to github with your github account which you must create if you did not have one already  

## Getting your Digital Ocean credits

Find Digital Ocean under student pack and copy your promotion code. Go to [https://www.digitalocean.com/](https://www.digitalocean.com/) to create your account. You have to insert a valid credit card or a PayPal account during registeration. When filling the billing information you can also add your student pack promotion code to get $50 of credits for free.

## Getting a domain from Namecheap

Find Namecheap under student pack. Get promocode and use it at Namecheap  
Setup DNS as following at namecheap: Domains -> Registration. Click Manage on domain at Domain List

Nameservers, select Custom DNS and add followint nameservers:
```
ns1.digitalocean.com
ns2.digitalocean.com
ns3.digitalocean.com
```

## Creating your Digital Ocean droplet

You can use any Linux distribution that you prefer, but in this guide we will use the latest long time release version of Ubuntu Server (currently Ubuntu 16.04.1 x64). We will pick the cheapest server with 512mb of RAM, which will be enough for simple web hosting. If you need a better server, you can upgrade your droplet later. For your droplet's hostname enter your domain name. If you don't have one yet, just enter any hostname as you can change it later. It is recomment to use Amsterdam 2 or 3 as the location of the VPS. If you have a SSH key you can add it to your droplet now, but it is not required. After you create the droplet, the root password will be emailed to you.

### Domain setup on your droplet

If you got a domain from Namecheap or have another domain, you can now connect it to your droplet. Adding a domain is not required, and you can use your server without completing this step. This part of the guide is limited, but [here](https://www.digitalocean.com/community/tutorials/how-to-set-up-a-host-name-with-digitalocean) is a link to a more detailed one.

Set domain as following (DNS entries at Digital Ocean, Take a look that you have corresponding lines as in example)  
jukka hostname  
Add domain  
juslin.org -> Create Record  
A record @ 188.166.74.17  
A record juslin.org 188.166.74.17  
CNAME www juslin.org.  

## Connecting to your droplet
Log into your droplet using Putty SSH as root. First you will be asked for your initial password that was sent to you by email. You will then be asked to setup a new root password. Make your root password as strong as possible. After you have setup your new password you should now have a root shell on your droplet.

### Adding a new user account
You should avoid logging in as the root user, and instead create a new account for yourself to use. Let's create a new user account with the `adduser` command. Just replace `<username>` with your desired username:
```
adduser <username>
```
Now let's add your new user account to the sudoers group so that you can use the `sudo` command to run individual commands as root. We can edit the sudoers file by typing the command `visudo`. We want to include your new user in the file like so:
```
# User privilege specification
root        ALL=(ALL:ALL) ALL
<username>  ALL=(ALL:ALL) ALL
```
Save the file and exit. Your new user should now have sudoer rights.

### Testing your new user account
You should not close your root shell just yet. Open up a new Putty window and try to log in to your server using the user account you just created. Once you are in, you should try running a command as `sudo`, like for example updating and upgrading your server's programs:
```
sudo apt-get update && sudo apt-get upgrade -y
```
The server should prompt you for your password before running the command. If the command runs like it should, you now have a user account that you can use and no longer need the root shell.

### Disabling root login
You should disable root login to your server and only use the account that you created.
```
sudoedit /etc/ssh/sshd_config  
```
Find the line that says `PermitRootLogin yes` and change it to `PermitRootLogin no`. After that restart the ssh service with the command `sudo service sshd reload`.

### Setting timezone and locale
You should set the correct timezone:
```
sudo dpkg-reconfigure tzdata
```
For locale select the locales that you need. In my case I have enabled `en_US.UTF-8` and `fi_FI.UTF-8`. Set the locales:
```
sudo dpkg-reconfigure locales
```
## Setting up Nginx, Tomcat and MariaDB

First update your repositories:
```
sudo apt-get update
```

Then install all the necessary programs, dependencies will be downloaded and installed automatically:
```
sudo apt-get install -y tomcat8 nginx mariadb-server-10.0
```

### Configuring MariaDB

First run `mysql_secure_installation` to disable test users, databases and to set a MySQL root password. You can then open a MySQL shell as the root user with the command `sudo mysql`. You should create a new MySQL user and database for each one of your web applications that require database access.

#### Creating a new user in MySQL
In this example we create a MySQL user "testuser" with the password "demopassword". We also create a database "testdatabase" and grant full priviliges to "testuser" in that database.
```
CREATE USER 'testuser'@'localhost' IDENTIFIED BY 'demopassword';
CREATE DATABASE testdatabase;
GRANT ALL PRIVILEGES ON testdatabase.* TO 'testuser'@'localhost';
```

Now when we exit the MySQL root shell we can try to log in as testuser to verify that we have succeeded:
```
mysql testdatabase -u testuser -p
```

### Configuring Tomcat 8

Tomcat doesn't need a lot of configuration. What we want to do though, is making it listen only to connections from localhost. That way we can proxy connections from Nginx to Tomcat, so that it can be accessed via the default HTTP port 80. You could also just set up Tomcat to listen to port 80 without Nginx, but to my knowledge it is not considered good practice. The benefit of running Nginx or Apache in front of Tomcat is also that you can also host static websites or for example websites made with PHP at the same time.

To make Tomcat listen to localhost only, we edit the file `/var/lib/tomcat8/conf/server.xml`. Depending on your Linux distribution or Tomcat version the location of Tomcat installation directory might differ. Below is the relevant bit in the configuration that we have to change. Find the line with `<Connector port="8080".....` in your `server.xml` and add `address="127.0.0.1"` inside the tag, like so:

```
    <Connector port="8080" protocol="HTTP/1.1"
               connectionTimeout="20000"
               URIEncoding="UTF-8"
	             address="127.0.0.1"
               redirectPort="8443" />
```
Now restart your Tomcat service with `sudo service tomcat8 restart` and it should no longer allow connections from the internet.

### Configuring Nginx

Nginx is the web server that we will be using to proxy connections to Tomcat. For this we need to edit the file `/etc/nginx/sites-available/default`. By default the file has some good examples of how to set up different kind of stuff, like PHP support and HTTPS. You can make a backup of the original file in case you want to look at it later:
```
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default_backup
```

We can then edit original file. Here is a minimal configuration that works by proxying all requests to Tomcat:
```
server {
	listen 80 default_server;
	listen [::]:80 default_server;

	root /var/www/html;

	index index.html index.htm;

	server_name _;

	location / {
		proxy_pass http://localhost:8080;
	}
}
```
After saving and exiting your editor, restart Nginx with the command `sudo service nginx`. Now by navigating to your server's address you should see a Tomcat 8 welcome page.

## Maintaining your server

You should log in to your server regularly to install the latest updates. You can fetch the latest repositories and install the updates with one simple command:
```
sudo apt-get update && sudo apt-get upgrade -y
```

## Multitasking in terminal
When you have learned the basics of using the terminal, you might want to have multiple windows or split your terminal pane into two different terminals to make multitasking easier. One solution to this is to make many connections to your server to get many terminals, but there are better ways, like installing `screen` or `tmux`. Tmux is more newbie friendly than screen, and allows splitting the screen in multiple regions. Here are links to cheatsheets listing all the different tmux and screen commands that you can use:

[Tmux Cheat Sheet](http://tmuxcheatsheet.com/)

[GNU Screen cheat-sheet](http://neophob.com/2007/04/gnu-screen-cheat-sheet/)

These cheat sheets list all the available commands, but you only need to know a few (creating and closing windows, navigating between windows) to get more productive.

## Enabling swap
Is your server running out of memory but you don't want to upgrade to a more expensive droplet? See [enabling swap](https://github.com/jusju/ownserver/blob/master/enabling_swap.md) as a workaround.

## Using Puppet

Advanced users can use Puppet to automate program installation and configuration. It is not needed, as we have already done all of this manually. This section is not finished and only contains some copypasted notes from Jukka.
  
sudo apt-get update  
sudo apt-get -y upgrade  
sudo apt-get -y install puppet git tmux  
git clone https://github.com/jusju/puppet.git #forked from nicougit  
cd puppet  
Poista server.xml myy browser tomcat  
LOGWATCH TO FIX  
root@ubuntu-512mb-ams2-01:~/puppet# cp -r nginx/ tomcat8/ mariadb/ fail2ban/ logwatch/ /etc/puppet/modules  

  
root@ubuntu-512mb-ams2-01:~/puppet# cd /etc/puppet/manifests/  
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# puppet apply site.pp  


run  
sudo mysql_secure_installation  
sudo mysql must be used when accessing mariadb as root  
mysql configuration  
  

add similar database user as at HH  
grant rights to given database user  
  

jusju@jukka:~$ sudo mysql -u root -p  
Enter password:  
Welcome to the MariaDB monitor.  Commands end with ; or \g.  
Your MariaDB connection id is 52  
Server version: 10.0.28-MariaDB-0ubuntu0.16.04.1 Ubuntu 16.04  
  

Copyright (c) 2000, 2016, Oracle, MariaDB Corporation Ab and others.  

  
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.  

  
MariaDB [(none)]> create database test;  
Query OK, 1 row affected (0.00 sec)  
MariaDB [(none)]> create user 'test'@'localhost' identified by 'test';  
Query OK, 0 rows affected (0.00 sec)  
MariaDB [(none)]> GRANT ALL ON test.* to 'test'@'localhost' ;  
Query OK, 0 rows affected (0.01 sec)  

  
MariaDB [(none)]>  

  
CTRL+Z  

  
Add swap otherwise Java EE applications won’t run:  
  

https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04
  
nmap --top-ports 4000 hostname  
  
Configure tomcat to use ipV4 with CATALINA 
  
sudo apt-get -y install haveged  
  
netstat -atpn  

Also set time  

google digitalocean time 
sudo dpkg-reconfigure tzdata    
