# Archlinux PHP Apache Mysql MongoDb
Fast Archlinux Instalation and configuration web and databases servers (PHP, Apache, Mysql and MongoDb)

(based on this)
https://ostechnix.com/install-apache-mariadb-php-lamp-stack-on-arch-linux-2016/


### Apache Conf and Install

```sh
pacman -Syu
pacman -S apache
vim /etc/httpd/conf/httpd.conf

(comment line)
vim LoadModule unique_id_module modules/mod_unique_id.so

systemctl enable httpd
systemctl restart httpd

systemctl status httpd

--> resp (end line): AH00558: httpd: Could not reliably dete...ge
Hint: Some lines were ellipsized, use -l to show in full.

echo "<html>Test Apache</html>" /srv/http/index.html

(browser test)
127.0.0.1


# Virtual Hosts Config (FollowSymbolicLinks)

sudo chmod o+x /home/ /home/user/ /home/user/folder

ln -s /home/user/folder symblin_kname

vim /etc/httpd/conf/extra/httpd-vhosts.conf

<VirtualHost *:80>
    ServerAdmin webmaster@localhost.com
    DocumentRoot "/srv/http"
    ServerName localhost
    ServerAlias localhost
    <Directory /srv/http>
        Options FollowSymLinks 
        Order Allow,Deny
        Allow from all
        AllowOverride Indexes
    </Directory>
    ErrorLog "/var/log/httpd/localhost-error_log"
    CustomLog "/var/log/httpd/localhost-access_log" common
</VirtualHost>

```

### Mysql (MariaDB) Conf and Install

```sh
pacman -Syu
pacman -S mysql

sudo mysql_install_db --user=mysql --basedir=/usr --datadir=/var/lib/mysql

sudo systemctl enable mysqld
sudo systemctl start mysqld
sudo systemctl status mysqld

resp (end line): Hint: Some lines were ellipsized, use -l to show in full.

sudo mysql_secure_installation


Enter current password for root (enter for none): ## Press Enter
OK, successfully used password, moving on...

New password:##  Enter password
Re-enter new password:  ## Re-enter password
Password updated successfully!
Reloading privilege tables..
 ... Success!
 
Remove anonymous users? [Y/n]## Press Enter
Disallow root login remotely? [Y/n]## Press Enter
Remove test database and access to it? [Y/n]## Press Enter
Reload privilege tables now? [Y/n]## Press Enter
 ... Success!

resp (end line): Thanks for using MariaDB!
```

### PHP Conf and Install

```sh
sudo pacman -S php php-apache php-mongodb php-fpm php-gd
```

### PHP CONF `/etc/httpd/conf/httpd.conf`

```
(coment this line on file httpd.conf)
#LoadModule mpm_event_module modules/mod_mpm_event.so

## copy the lines under at end of file httpd;conf: (for php 8 archlinux and above only)

LoadModule mpm_prefork_module modules/mod_mpm_prefork.so
LoadModule php_module modules/libphp.so
AddHandler php-script php

AddType application/php .php
Include conf/extra/php_module.conf
Include conf/extra/httpd-vhosts.conf

echo "<?php phpinfo(); ?>" /srv/http/info.php

sudo systemctl restart httpd

127.0.0.1/info.php
```

## Install and Conf MongoDB

https://wiki.archlinux.org/title/MongoDB

https://www.php.net/manual/pt_BR/mongodb.installation.pecl.php

(install)

```sh
sudo pacman -S php-mongodb

git clone https://aur.archlinux.org/mongodb-bin.git
makpgk
sudo pacman -U mongo-bin*.zst

git clone https://aur.archlinux.org/mongodb-tools-bin.git
makpgk
sudo pacman -U <package>

git clone https://aur.archlinux.org/mongosh-bin.git
makpgk
sudo pacman -U <package>

git clone https://aur.archlinux.org/mongodb-compass.git
makpgk
sudo pacman -U mongo-compass*.zst
```


## Install phpMyAdmin


phpMyAdmin is a graphical MySQL/MariaDB administration tool that can be used to create, edit and delete databases.

To install it, run:
```
pacman -S phpmyadmin php-mcrypt
```
After installing, edit php.ini file,
```
nano /etc/php/php.ini
```
Make sure the following lines are uncommented.

```
[...]
extension=bz2.so
extension=mcrypt.so
extension=mysqli.so
[...]
```
Save and close the file.

Next, create configuration file for phpMyAdmin,
```
nano /etc/httpd/conf/extra/phpmyadmin.conf
```
Add the following lines:
```
Alias /phpmyadmin "/usr/share/webapps/phpMyAdmin"
 <Directory "/usr/share/webapps/phpMyAdmin">
  DirectoryIndex index.php
  AllowOverride All
  Options FollowSymlinks
  Require all granted
 </Directory>
```
Then, open Apache configuration file,
```
nano /etc/httpd/conf/httpd.conf
```
Add the following line at the end:
```
Include conf/extra/phpmyadmin.conf
```
Save and close the file. Restart httpd service.
```
systemctl restart httpd
```
