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
Namecheap offers a free .me domain for 1 year for all Github student pack users. If you don't want a domain at this point and only want your server to be accessible via its IP address, you can skip this step.

First find Namecheap under the student pack website. Get your promotion code and use it at Namecheap to create an account and register a domain.

### Setting up DNS
You want to make your domain's nameserver to point to DigitalOcean's nameservers where your server will be located.

Setup DNS as following at Namecheap: Navigate to Domains -> Registration. Click Manage on domain at Domain List

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
You should avoid logging in as the root user, and instead create a new account for yourself to use. Let's create a new user account with the `adduser` command. Just replace `<username>` with your desired username.
```
adduser <username> sudoers
```
By specifying the groupname `sudoers` in the command we automatically add our new account to the `sudoers` group which gives us the permission to run commands as root.

### Testing your new user account
You should not close your root shell just yet. Open up a new Putty window and try to log in to your server using the user account you just created. Once you are in, you should try running a command as `sudo`, like for example updating and upgrading your server's programs:
```
sudo apt-get update && sudo apt-get upgrade -y
```
The server should prompt you for your password before running the command. If the command runs like it should, you now have a user account that you can use and no longer need the root shell.

### Disabling root login
Once you have confirmed that your new account has sudoer rights, you should disable root login to your server and only use the account that you created.
```
sudoedit /etc/ssh/sshd_config  
```
Find the line that says `PermitRootLogin yes` and change it to `PermitRootLogin no`. After that restart the ssh service with the command `sudo service sshd reload`.

Let's also disable root account locally:
```
sudo usermod --lock root
```

### Setting timezone and locale
You should set the correct timezone:
```
sudo dpkg-reconfigure tzdata
```
For locale select the locales that you need. In my case I have enabled `en_US.UTF-8` and `fi_FI.UTF-8`. Set the locales:
```
sudo dpkg-reconfigure locales
```

## Hosting web applications
If you want to set your server up for hosting Java web applications with Tomcat 8, Nginx and MariaDB, check [this guide](https://github.com/jusju/ownserver/blob/master/webservers.md). It now also covers information on hosting static and PHP pages.

## Maintaining your server

You should log in to your server regularly to install the latest updates. You can fetch the latest repositories and install the updates with one simple command:
```
sudo apt-get update && sudo apt-get upgrade -y
```

## Multitasking in terminal
When you have learned the basics of using the terminal, you might want to have multiple windows or split your terminal pane into two different terminals to make multitasking easier. One solution to this is to make many connections to your server to get many terminals, but there are better ways, like installing `screen` or `tmux`. Personally I find tmux to be more newbie friendly than screen, and it allows splitting the screen in multiple regions easily. Here are links to cheatsheets listing all the different tmux and screen commands that you can use:

[Tmux Cheat Sheet](http://tmuxcheatsheet.com/)

[GNU Screen cheat-sheet](http://neophob.com/2007/04/gnu-screen-cheat-sheet/)

These cheat sheets list all the available commands, but you only need to know a few (creating and closing windows, navigating between windows) to get more productive.

## Enabling swap
Is your server running out of memory but you don't want to upgrade to a more expensive droplet? See [enabling swap](https://github.com/jusju/ownserver/blob/master/enabling_swap.md) as a workaround.

## Using Puppet
Advanced users can use Puppet to automate program installation and configuration. It is not needed, as we have already done all of this manually. [Here](https://github.com/jusju/ownserver/blob/master/puppet.md) is a link to some notes related to using puppet. This section is not finished and only contains some copypasted notes from Jukka.
