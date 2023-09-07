Install Multiple Versions of PHP on Ubuntu via PPA
The easiest way to install multiple versions of PHP is by using the PPA from Ondřej Surý, who is a Debian developer. To add this PPA, run the following commands in the terminal. The software-properties-common package is needed if you want to install software from PPA. It’s installed automatically on Ubuntu desktop, but might be missing on your Ubuntu server.

> sudo apt install software-properties-common

> sudo add-apt-repository ppa:ondrej/php

> sudo apt update

Now you can install PHP8.1 on Ubuntu by executing the following command.

> sudo  apt install php8.1 php8.1-fpm 

And install some common PHP8.1 extensions.

> sudo apt install php8.1-mysql php8.1-mbstring php8.1-xml php8.1-gd php8.1-curl

To install PHP7.4 on Ubuntu , run

> sudo apt install php7.4 php7.4-fpm

Install some common PHP7.4 extensions.

> sudo apt install php7.4-mysql php7.4-mbstring php7.4-xml php7.4-gd php7.4-curl

Switching PHP Version in Apache Virtual Host
By default, Apache uses one PHP version across all virtual hosts.
If you want to use a different PHP version in a particular virtual host, 
you will need to disable Apache PHP module and run PHP code via PHP-FPM. Check if mod_php is installed.

> dpkg -l | grep libapache2-mod-php


If it’s installed, you need to disable it.

>sudo a2dismod php7.4


also, disable the prefork MPM module.

> sudo a2dismod mpm_prefork

run the following command to enable three modules in order to use PHP-FPM, regardless of whether mod_php is installed on your server.

>sudo a2enmod mpm_event proxy_fcgi setenvif

add the following line in your virtual host between <VirtualHost> tags and then restart Apache.

> Include /etc/apache2/conf-available/php8.0-fpm.conf

>`DocumentRoot "/var/www/landingpage.local/public"`
> 
>`ServerName landingpage.local`
> 
>`<Directory "/var/www/landingpage.local/public">`
> 
>`AllowOverride All`
> 
>`Order Allow,Deny`
> 
>`Allow from All`
>
>`</Directory>`
>
>`#Include /etc/apache2/conf-available/php7.4-fpm.conf`
> 
>`#Include /etc/apache2/conf-available/php8.1-fpm.conf`
>
>`Include /etc/apache2/conf-available/php8.2-fpm.conf`
>
>`ErrorLog ${APACHE_LOG_DIR}/error.log`
>
>`CustomLog ${APACHE_LOG_DIR}/access.log combined`