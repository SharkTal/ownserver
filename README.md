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




