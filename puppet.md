# Using Puppet

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
