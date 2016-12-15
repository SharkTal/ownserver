#Own Backend Server  
Written by Jukka Juslin and Nico Hagelberg  
  

How to setup your own server as a student for free using Digital Ocean and Namecheap  
Github offers so called student pack, which includes credit coupons to Digital Ocean and Namecheap. Digital Ocean and namecheap have been widely used already before this offer. Digital Ocean is a cloud VPS (Virtual Service Provider) and namecheap gives DNS mappings. Due to lack of time these instructions are not written for a newbie level.  
  

Some Digital Ocean “wiki” pages are good.  
  

Go to https://education.github.com/pack  
Click "get your pack". Getting the pack might take couple of hours or might be ready instantly.  
Sign in to github with your github account which you must create if you did not have one already  
Find Namecheap under student pack  
Get promocode and use it at Namecheap  
Setup DNS as following at namecheap:  
Domains -> Registration  
Click Manage on domain at Domain List  
Nameservers, select Custom DNS and add followint nameservers:  
ns1.digitalocean.com  
ns2.digitalocean.com  
ns3.digitalocean.com  
Find Digital Ocean under student pack  
Store promotion code to be inserted to Digital Ocean to get service for free  
Use that code of Digital Ocean ordering site  

  
Create droplet  
Ubuntu 16.04.1 x64  
1GB RAM  
Choose a hostname: put your domain name  

  
It is recomment to use Amsterdam 2 or 3 as the location of the VPS  

  
Set domain as following (DNS entries at Digital Ocean, Take a look that you have corresponding lines as in example)  
jukka hostname  
Add domain  
juslin.org -> Create Record  
A record @ 188.166.74.17  
A record juslin.org 188.166.74.17  
CNAME www juslin.org.  

  


Log into your droplet using Putty SSH as root. Remove from sshd config remote root logins, create user account for yourself using adduser command. Add using visudo root permissions to your account.  
  

Disable remote root logins from ssh.  
sudo nano -w sshd_config  
/etc/ssh/PermitRootLogin no  
  

sudo service sshd reload  
  

Exit root shell to your account  
sudo jusju  

  
Set your VPS services for hosting Java EE code as following. In front of is nginx because it is not recommended to run tomcat in front.  

  
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

  
MariaDB [(none)]> create database db_passi;  
Query OK, 1 row affected (0.00 sec)  
MariaDB [(none)]> create user 'access'@'localhost' identified by 'MarilynMonroe1926';  
Query OK, 0 rows affected (0.00 sec)  
MariaDB [(none)]> GRANT ALL ON db_passi.* to 'access'@'localhost' ;  
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
