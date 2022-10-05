# LEMP Setup on Android

## Installation

Inside your termux app install PHP and Nginx.

```sh
apt update && apt upgrade
apt install php-fpm nginx mariadb
```

At the time of writing, the stack versions are:
- PHP 8.1.11 (fpm-fcgi)
- nginx/1.23.1
- mariadb  Ver 15.1 Distrib 10.9.2-MariaDB

## Configurations

### PHP

Edit the PHP config in `/data/data/com.termux/files/usr/etc/php-fpm.d/www.conf` (or `$PREFIX/etc/php-fpm.d/www.conf`) and set it to listen on a unix socket:
```
listen = /data/data/com.termux/files/usr/var/run/php-fpm.sock
```

Alternatively, you can also set it to listen on a IP address & TCP port like:
```
; 'ip.add.re.ss:port' to listen on a TCP socket
; to a specific IPv4 address on a specific port
listen = 127.0.0.1:9000
; 'port' to listen on a TCP socket to all
; addresses (IPv6 and IPv4-mapped) on a specific
; port
listen = 9000
```

### nginx

The nginx default config directory is located at `/data/data/com.termux/files/usr/etc/nginx` or `$PREFIX/etc/nginx`. Open`nginx.conf` and edit it as follows.

Set error log location as:
```
error_log  /data/data/com.termux/files/usr/var/log/nginx/error.log;
```

Set your root location block as:
```
location / {
    root   /data/data/com.termux/files/usr/share/nginx/html;
    index  index.php index.html index.htm;
    try_files $uri $uri/ /index.html;
}
```

Uncomment and edit your php (`\.php$`) location block to look like:
```
location ~ \.php$ {
    try_files $uri =404;
    root           /data/data/com.termux/files/usr/share/nginx/html;
    fastcgi_pass unix:/data/data/com.termux/files/usr/var/run/php-fpm.sock;
    fastcgi_index  index.php;
    fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
}
```

The original configuration file with proposed changes is included in the repo as `nginx.conf.full`. There's another cleaned up copy with comments & unnecessary configurations removed named as `nginx.conf`.

### Document Root

The document root is a directory from where a web server serves webpages. Its path is mentioned in the `nginx.conf`. The default root is set to `/data/data/com.termux/files/usr/share/nginx/html`.

If you want to change the default document root directory (which you should) to make it easy on yourself to edit php files, change it to a directory in your $HOME, e.g.
```
root   /data/data/com.termux/files/home/htdocs;
```
on three places.
1. root location block (`location / {}`)
2. 50x location block (`location = /50x.html {}`)
3. PHP file location block (`location ~ \.php$ {}`)

For lazy people, just make a soft link to your document root directory in your termux home ($HOME).
```
# place soft link inside termux home
ln -s $PREFIX/share/nginx/html $HOME/htdocs
```

if you have setup the termux storage script, then you can place this soft link (or the document root itself) in your phone's sdcard. This'll simplify your workflow as you would be able to use a good code editor (like Acode etc.) to edit the php files.

### MariaDB

To access root account, you need to login with Termux user name
```
mysql -u $(whoami)
```

The command shown above will also initialize the database with 2 all-privilege accounts (introduced perhaps in MariaDB 10.4.x). The first one is "root" which is inaccessible and the second one with name of your Termux user (check with command id -un or whoami). 

To enable access to root account, you need to login with your Termux user name

```
mysql -u $(whoami)
```

and manually change password for root

```
use mysql;
set password for 'root'@'localhost' = password('YOUR_ROOT_PASSWORD_HERE');
flush privileges;
quit;
```

Verify that you are able to login as 'root' with `mysql -u root -p`. You will need to provide password set in previous step. 

Whenever you want to access MySQL database manually through command line or with some program (web application), you need to start MySQL server:

```
mysqld_safe
```

Then you should be able to connect to database, e.g. wwith`mysql -u root -p`. 


## Services

First start PHP with the command
```
php-fpm
```

Now test your nginx config by the command
```
nginx -t
```

if the test is successful, start nginx service with the command:

```
nginx
```

You can communicate with the nginx service with signals (stop, quit, reopen, reload) like:

```
nginx -s stop
```

Everytime you make a change to nginx config, you have to reload the service:
```
nginx -s reload
```

## Service Automation

termux-services contains a set of scripts for controlling services. Instead of putting commands in ~/.bashrc or ~/.bash_profile, they can be started and stopped with termux-services. It manages a wide variety of back-end services such as apache httpd, lighttpd, nginx, ftpd, sshd, mysqld, postgres, mosquitto, crond, privoxy and more.

To install termux-services, run
```
pkg install termux-services
```
and then *restart termux* so that the service-daemon is started.

To then enable and run a service, run
```
sv-enable <service>
```
If you only want to run it once, run
```
sv up <service>
```
To later stop a service, run:
```
sv down <service>
```
Or to disable it
```
sv-disable <service>
```
A service is disabled if `$PREFIX/var/service/<service>/down` exists, so the `sv-enable` and `sv-disable` scripts touches, or removes, this file.

termux-services uses the programs from runit to control the services. A bunch of example scripts are available from the same site. If you find a script you want to use, or if you write your own, you can use set it up by running:
```
mkdir -p $PREFIX/var/service/<PKG>/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/<PKG>/log/run
```
and then put your run script for the package at `$PREFIX/var/service/<PKG>/run` and make sure that it is runnable.

You can then run
```
sv up <PKG>
```
to start it.

Log files for services are situated in `$PREFIX/var/log/sv/<PKG>/ with the active log file named "current". 

## Test your Code

Finally, the moment of truth! Now that yor services are running, test some code.

Now write a `index.php` file in your document root.

```php
<php?
echo "<h1>Hello Android!</h1>";
```

To visit your first page, open a browser and write in the address bar:

```
localhost:8080
```

If everything went ok, you'll see the output of the echo in your browser. Viola!!!

Happy coding geeks!
