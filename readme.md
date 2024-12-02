Prerequisites You should already have Apache installed and serving web pages before following this guide.

This guide was tested on Ubuntu 24.04, Ubuntu 22.04, Ubuntu 20.04 and Ubuntu 18.04, though it should also be useful for other Debian-based systems.

If you need to install Apache on Ubuntu, please see:

Add Repository The software-properties-common package is required to add Ondřej’s PHP repository, which will allow us to download co-installable versions of PHP.
sudo apt install software-properties-common -y

sudo add-apt-repository ppa:ondrej/php -y

Update repository:

sudo apt update -y

Next install libapache2-mod-fcgid. This starts a number of CGI program instances to handle concurrent requests.

sudo apt install libapache2-mod-fcgid

Install Multiple PHP Versions
You can now install the versions of PHP you require.

As of writing, Ondřej’s repository provides PHP 5.6, 7.0, 7.1, 7.2, 7.3, 7.4, 8.0, 8.1 and 8.2.

In this example, we will install PHP 5.6 and PHP 7.4.

You will also need to install the following packages for each PHP version.

phpX.X-fpm – Fast Process Manager interpreter that runs as a daemon and receives Fast/CGI requests.

phpX.X-mysql – This connects PHP to the MySQL database.

libapache2-mod-phpX.X – This provides the PHP module for the Apache webserver.

Install PHP 5.6 and associated packages. Press y and ENTER when prompted to install.

sudo apt install php5.6 php5.6-fpm php5.6-mysql libapache2-mod-php5.6

Install PHP 7.4 and associated packages. Press y and ENTER when prompted to install.

sudo apt install php7.4 php7.4-fpm php7.4-mysql libapache2-mod-php7.4

Once installed, you should have two new sockets in /var/run/php/.

 ls -la /var/run/php/

-rw-r--r-- 1 root     root     4 Feb 17 16:50 php5.6-fpm.pid
srw-rw---- 1 www-data www-data 0 Feb 17 16:50 php5.6-fpm.sock
-rw-r--r-- 1 root     root     5 Feb 17 16:51 php7.4-fpm.pid
srw-rw---- 1 www-data www-data 0 Feb 17 16:51 php7.4-fpm.sock
In Step 3, we will use the <FilesMatch> directive to tell Apache which PHP socket to use.

Other Extensions/Libraries

Note that if you need any other libraries or extensions, they must be installed separately per PHP version.

The command below includes some of the most popular PHP extensions, which should cover a typical WordPress site. Make sure to replace 7.4 with the version you require.

sudo apt install php7.4-common php7.4-mysql php7.4-xml php7.4-xmlrpc php7.4-curl

Start PHP-FPM Services
You must now start the FPM services per version of PHP you installed.

In this example, we will start PHP 5.6 first.

sudo systemctl start php5.6-fpm

Once started, check the status:

sudo systemctl status php5.6-fpm

Output:

● php5.6-fpm.service - The PHP 5.6 FastCGI Process Manager
     Loaded: loaded (/lib/systemd/system/php5.6-fpm.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-10-24 20:21:22 IST; 14min ago
       Docs: man:php-fpm5.6(8)
   Main PID: 94753 (php-fpm5.6)
     Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
      Tasks: 3 (limit: 1131)
     Memory: 28.2M
     CGroup: /system.slice/php5.6-fpm.service
             ├─94753 php-fpm: master process (/etc/php/5.6/fpm/php-fpm.conf)
             ├─94764 php-fpm: pool www
             └─94765 php-fpm: pool www

Oct 24 20:21:22 devanswers systemd[1]: php5.6-fpm.service: Succeeded.
Oct 24 20:21:22 devanswers systemd[1]: Stopped The PHP 5.6 FastCGI Process Manager.
Oct 24 20:21:22 devanswers systemd[1]: Starting The PHP 5.6 FastCGI Process Manager...
Oct 24 20:21:22 devanswers systemd[1]: Started The PHP 5.6 FastCGI Process Manager.
Now repeat the commands for the next PHP version, in this example, PHP 7.4.

sudo systemctl start php7.4-fpm

Once started, check the status:

sudo systemctl status php7.4-fpm

Output:

● php7.4-fpm.service - The PHP 7.4 FastCGI Process Manager
     Loaded: loaded (/lib/systemd/system/php7.4-fpm.service; enabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-10-24 20:29:01 IST; 10min ago
       Docs: man:php-fpm7.4(8)
   Main PID: 94753 (php-fpm7.4)
     Status: "Processes active: 0, idle: 2, Requests: 0, slow: 0, Traffic: 0req/sec"
      Tasks: 3 (limit: 1131)
     Memory: 28.2M
     CGroup: /system.slice/php7.4-fpm.service
             ├─94753 php-fpm: master process (/etc/php/7.4/fpm/php-fpm.conf)
             ├─94764 php-fpm: pool www
             └─94765 php-fpm: pool www

Oct 24 20:29:48 devanswers systemd[1]: php7.4-fpm.service: Succeeded.
Oct 24 20:29:48 devanswers systemd[1]: Stopped The PHP 7.4 FastCGI Process Manager.
Oct 24 20:29:48 devanswers systemd[1]: Starting The PHP 7.4 FastCGI Process Manager...
Oct 24 20:29:48 devanswers systemd[1]: Started The PHP 7.4 FastCGI Process Manager.
Configure Apache
We need to add some Apache modules using a2enmod.

actions – used for executing CGI scripts based on media type or request method.

fcgid – a high performance alternative to mod_cgi that starts a sufficient number of instances of the CGI program to handle concurrent requests.

alias – provides for the mapping of different parts of the host filesystem in the document tree, and for URL redirection.

proxy_fcgi – allows Apache to forward requests to PHP-FPM.

sudo a2enmod actions fcgid alias proxy_fcgi

Restart Apache.

sudo systemctl restart apache2

You can now use either Virtual Hosts or .htaccess to instruct Apache which version of PHP to use.

Virtual Hosts Method
Open your Apache .conf file and add the <FilesMatch> directive to your Virtual Host.

To view your Virtual Hosts, run:

sudo ls -la /etc/apache2/sites-available/

If you are running multiple domains on your Apache server, you should see multiple .conf files here. If not, you may only have 000-default.conf. In that case, you can edit this file:

sudo nano /etc/apache2/sites-available/000-default.conf

Within the <VirtualHost *:80> container (or <VirtualHost *:433> if you are running SSL), add a FilesMatch directive to instruct the virtual host to run a specific version of PHP.

PHP 5.6 Example

<VirtualHost *:80>
    <FilesMatch \.php> # Apache 2.4.10+ can proxy to unix socket 
        SetHandler "proxy:unix:/var/run/php/php5.6-fpm.sock|fcgi://localhost/" 
    </FilesMatch> 
</VirtualHost> 
Save and exit (press CTRL + X, press Y and then press ENTER)

Make sure to restart Apache after making changes to the Virtual Hosts.

sudo systemctl restart apache2

PHP 7.4 Example /etc/apache2/sites-available/000-default.conf <VirtualHost *:80> <FilesMatch \.php> # Apache 2.4.10+ can proxy to unix socket  SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost/"  </FilesMatch>  </VirtualHost>

Save and exit (press CTRL + X, press Y and then press ENTER)

Make sure to restart Apache after making changes to the Virtual Hosts.

sudo systemctl restart apache2

htaccess Method Instead of the virtual hosts method, you can add the <FilesMatch> directive to your .htaccess file. Before you do, make sure that AllowOverride is enabled, otherwise Apache will ignore .htaccess.
Open the Apache config file.

sudo nano /etc/apache2/apache2.conf

Scroll down the the following section and make sure that AllowOverride is set to All.

/etc/apache2/apache2.conf
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
Save and exit (press CTRL + X, press Y and then press ENTER)

Restart Apache.

sudo systemctl restart apache2

Now you can add the <FilesMatch> directive to your .htaccess file.

PHP 5.6 Example

.htaccess
<FilesMatch \.php> 
    # Apache 2.4.10+ can proxy to unix socket 
    SetHandler "proxy:unix:/var/run/php/php5.6-fpm.sock|fcgi://localhost/" 
</FilesMatch>

PHP 7.4 Example
.htaccess
<FilesMatch \.php> 
    # Apache 2.4.10+ can proxy to unix socket 
    SetHandler "proxy:unix:/var/run/php/php7.4-fpm.sock|fcgi://localhost/" 
</FilesMatch>
Test PHP To see which version of PHP Apache is serving, create a new file called info.php in your web document root.
In this example, we will create a new file in /var/www/html/, though this should be in your own virtual host document root.

sudo nano /var/www/html/info.php

Enter the following PHP code.

/var/www/html/info.php
<?php
phpinfo(); 
?>
Save file and exit. (Press CTRL + X, press Y and then press ENTER)

We can now load this file in the browser by going to http://example.com/info.php or http://your_ip/info.php

Below we can see the PHP info page with the PHP version clearly displayed.

PHP 7 info test page on Apache and Ubuntu 18.04 Bionic Beaver Don’t forget to delete info.php as it contains information that could be useful to hackers.

sudo rm /var/www/html/info.php
