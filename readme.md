Install Multiple Versions of PHP on Ubuntu via PPA
The easiest way to install multiple versions of PHP is by using the PPA from Ondřej Surý, who is a Debian developer. To add this PPA, run the following commands in the terminal. The software-properties-common package is needed if you want to install software from PPA. It’s installed automatically on Ubuntu desktop, but might be missing on your Ubuntu server.

> sudo apt install software-properties-common

> sudo add-apt-repository ppa:ondrej/php

> sudo apt update

Now you can install PHP8.2 on Ubuntu by executing the following command.

> sudo  apt install php8.2 php8.2-fpm 

And install some common PHP8.2 extensions.

> sudo apt install php8.2-mysql php8.2-mbstring php8.2-xml php8.2-gd php8.2-curl php8.2-zip php8.2-intl

Now you can install PHP8.3 on Ubuntu by executing the following command.

> sudo apt install php8.3 php8.3-fpm

And install some common PHP8.3 extensions.

> sudo apt install php8.2-mysql php8.3-mbstring php8.3-xml php8.3-gd php8.3-curl php8.3-zip php8.3-intl

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
