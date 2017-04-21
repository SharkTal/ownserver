# Hosting web applications

In this guide we will setup your server to be able to run your Java web applications. The programs that we will be using are:
* Tomcat 8
* Nginx
* MariaDB

An alternative to Nginx is [Apache](https://httpd.apache.org/) which is also widely used. This tutorial will only cover setting up Nginx though.

If you wish to host static web pages or PHP applications on your server using Nginx, see 'Nginx advanced setup' on the bottom of this guide.

## Installing the packages

First update your repositories:
```
sudo apt-get update
```

Then install all the necessary programs, dependencies will be downloaded and installed automatically:
```
sudo apt-get install -y tomcat8 nginx mariadb-server-10.0
```

Last time I personally set up a server from ground up, I ran into an issue where Tomcat was starting up very slowly (minutes instead of seconds) which was fixed by installing the package `haveged`, which is a random number generator. You could install it now or do it later if you're having the same issue:
```
sudo apt-get install -y haveged
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
After saving and exiting your editor, restart Nginx with the command `sudo service nginx restart`.

### That's it!
Configuration is now done. By navigating to your server's address you should see a Tomcat 8 welcome page. You should now be able to deploy your .war packages to `/var/lib/tomcat8/webapps/` and Tomcat should automatically deploy them and they should be accessible via the default http port.

# Nginx advanced setup
If you wish to host static pages or PHP applications on your server in addition to Java applications running on Tomcat, you will need to do a little bit of configuration.

## Static web pages
To host static web pages we need to stop Nginx from proxying every single request to Tomcat. In `/etc/nginx/sites-available/default` we need to specify which requests we want to relay to Tomcat:
```
server {
        listen 80 default_server;
        listen [::]:80 default_server;

        root /var/www/html;

        index index.html index.htm;

        server_name _;

        location / {
                try_files $uri $uri/ =404;
        }

        location /demoapp {
                proxy_pass http://localhost:8080/demoapp;
        }

        location /anotherapp {
                proxy_pass http://localhost:8080/anotherapp;
        }
}

```

With this example configuration Nginx would proxy incoming requests with `/demoapp` and `/anotherapp` to Tomcat, while other requested files would be served from `/var/www/html/`. You could then move your static website to that directory and it would be shown to visitors.

## Enabling PHP support

### Installing packages
To enable PHP support we need to install some packages:
```
sudo apt-get install -y php7.0 php7.0-fpm php7.0-cgi php7.0-mysql
```

### Nginx configuration
We need to modify `/etc/nginx/sites-available/default` again to enable PHP support. If you have not completely overwritten the file with the examples provided in this tutorial, the default configuration already has PHP setup examples, but they are commented out (with #). If you still have them, just uncomment the settings and restart Nginx with `sudo service nginx restart`.

Here is a configuration that works for me:
```
location ~ \.php$ {
	fastcgi_pass	unix:/run/php/7.0-fpm.sock;
	try_files $uri =404;
	fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
	include fastcgi_params;
}
```

This needs to be placed inside the server { .... } bit just like the other `location /` settings. After restarting Nginx service your server should now support PHP files. This section of the guide is quite limited, so if you run into any problems and manage to fix them, please add your notes in this guide and make a pull request!
