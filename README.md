# Own Server

Written by Jukka Juslin and Nico Hagelberg

How to setup your own server as a student for free using Digital Ocean and Namecheap

Github offers so called student pack, which includes credit coupons to Digital Ocean and Namecheap. Digital Ocean and namecheap have been widely used already before this offer. Digital Ocean is a cloud VPS (Virtual Service Provider) and namecheap gives DNS mappings. Due to lack of time these instructions are not written for a newbie level.

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
It is recomment to use Amsterdam 2 or 3 as the location of the VPS

Set domain as following (DNS entries at Digital Ocean, Take a look that you have corresponding lines as in example)
A record @ 188.166.74.17
A record juslin.org 188.166.74.17
CNAME www juslin.org.
NS ns1.digitalocean.com.
NS ns2.digitalocean.com.
NS ns3.digitalocean.com.

Log into your droplet using Putty SSH as root. Remove from sshd config remote root logins, create user account for yourself using adduser command. Add using visudo root permissions to your account.

Set your VPS services for hosting Java EE code as following. In front of is nginx because it is not recommended to run tomcat in front.

sudo apt-get update
sudo apt-get install puppet
sudo apt-get install git
git clone https://github.com/nicougit/puppet.git
cd puppet
root@ubuntu-512mb-ams2-01:~/puppet# cp -r nginx/ tomcat8/ mariadb/ /etc/puppet/manifests/
root@ubuntu-512mb-ams2-01:~/puppet# ls /etc/puppet/manifests/
mariadb  nginx  tomcat8
root@ubuntu-512mb-ams2-01:~/puppet# cp site.pp /etc/puppet/
etckeeper-commit-post  manifests/             puppet.conf
etckeeper-commit-pre   modules/
root@ubuntu-512mb-ams2-01:~/puppet# cp site.pp /etc/puppet/manifests/
root@ubuntu-512mb-ams2-01:~/puppet# cd /etc/puppet/manifests/
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# ls
mariadb  nginx  site.pp  tomcat8
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# mv mariadb/ nginx/ tomcat8/ ../modules/
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# ls site.pp
following command must be done twice
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# puppet apply site.pp
root@ubuntu-512mb-ams2-01:/etc/puppet/manifests# puppet apply site.pp

mysql_secure_installation

mysql configuration
add similar database user as at HH
grant right to given database user





