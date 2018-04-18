# How to Re-install macOS High Sierra and the PHP Development Environment.
This document is a guideline for re-installing macOS High Sierra and the PHP
development environment on my MacBook Pro, so it contains some steps that are
specific for my own environment, such as the instructions for backing up and
restoring files. 

However, these steps are irrelevant and can be safely ignored, and the rest of
this document is still very useful for people who have troubles with installing
macOS High Sierra (such as failed to update firmware or to download the full
installer, and etc) or compiling PHP from source (such as the missing symbols
issue).

## Backup Files
Backup the following files:
```bash
~/
/opt/software/
/opt/ezpublish/
/usr/local/bin/service/
/opt/local/etc/varnish/
/opt/local/etc/nginx/
/opt/local/etc/mysql57/
/opt/php/5.6.35/etc/
/opt/php/7.2.4/etc/
```

## Clean Installation of macOS High Sierra
1. Install all latest updates on current Mac OS X, otherwise, instead of the
   full installer, only the macOS High Sierra network installer will be
   downloaded from AppStore.
1. Download macOS High Sierra full installer from AppStore.
1. Create a bootable installation disk with the following command:
```bash
$ sudo /Applications/Install\ macOS\ High\
Sierra.app/Contents/Resources/createinstallmedia --volume /Volumes/<drive-name>
```
4. Reset NVRAM: shutdown Mac, then turn it on and immediately press and hold
   these keys together: <kbd>option</kbd> + <kbd>command</kbd> + <kbd>P</kbd> + 
   <kbd>R</kbd>, finally release the keys after hearing the start up sound for
   the 4th time.

1. Boot with the installation disk by holding <kbd>option</kbd> key when turning
   on the Mac.

1. Follow the installation wizard to complete the installation.

## Enable Root User
Enable root user from command line:
```bash
$ dsenableroot
``` 
or from GUI:
```bash
$ open /System/Library/CoreServices/Applications/Directory\ Utility.app
# Click the lock icon to unlock protection.
# Choose Edit > Enable Root User from menu to enable root user.
```
## Grant Sudo Privileges
```bash
$ su - 
$ vi /etc/sudoers # grant privileges to current user
```
## Disable Root User
Disable root user from command line:
```bash
$ dsenableroot -d
```
or from GUI:
```bash
$ open /System/Library/CoreServices/Applications/Directory\ Utility.app
# Click the lock icon to unlock protection.
# Choose Edit > Disable Root User from menu to enable root user.
```

## Install Xcode
1. Download and install Xcode from AppStore.
1. Install command line tool:
```bash
$ xcode-select --install
```

## Install MacPorts
1. Download and install MacPorts [[1]].
1. Update the ports tree and MacPorts base:
```bash
$ sudo ports selfupdate
```
3. Add the following line in ``bash/var/root/.profile``:
```bash
export PATH=/opt/local/bin:$PATH
```

## Install MySQL
```bash
$ sudo port install mysql57-server
$ sudo port select mysql mysql57
$ sudo /opt/local/lib/mysql57/bin/mysqld --initialize --user=_mysql
# Make a note of the root user password which is auto-generated
$ /opt/local/lib/mysql57/bin/mysqladmin -u root -p password
$ password: [current password] # the auto-generated one
$ new password: [enter] # empty password
$ confirm new password: [enter] # empty password
$ sudo vi /opt/local/etc/mysql57/my.cnf
# Comment out the line that includes the default config file:
# !include /opt/local/etc/mysql57/macports-default.cnf
# Because macports-default.cnf turns on the skip-networking option, MySQL will
# only listen at local socket.
$ sudo port load mysql57-server
```

## Install PHP Dependencies
```bash
$ sudo port install openssl
$ sudo port install curl
$ sudo port install libmcrypt
$ sudo port install gd
$ sudo port select --set python python27
$ sudo port select --set python python27
$ sudo port install readline
$ sudo port install ImageMagick
$ sudo port install icu
$ sudo port install autoconf
$ sudo port install libmemcached
$ sudo mkdir -p /opt/software/php
$ sudo chown -R <your-name>:staff /opt/software
```

## Install PHP 5.6.35

### Download PHP
```bash
$ cd /opt/software/php
$ curl -o php-5.6.35.tgz -L http://cn2.php.net/get/php-5.6.35.tar.gz/from/this/mirror 
$ mkdir 5.6.35
$ tar zxf php-5.6.35 -C 5.6.35 --strip-components=1
$ rm -rf php-5.6.35.tgz
$ cd 5.6.35
$ git init .
$ git add .
$ git commit -m 'Add original files'
```

### Download ext/memcached
```bash
$ cd ext
$ curl -o memcached.tgz -L https://pecl.php.net/get/memcached-2.2.0.tgz
$ mkdir memcached
$ tar zxf memcached.tgz -C memcached --strip-components=1
$ rm -rf memcached.tgz
$ git add memcached
$ git commit -m "Add ext/memcached"
```

### Download ext/imagick
```bash
$ curl -o imagick.tgz -L https://pecl.php.net/get/imagick-3.4.3.tgz 
$ mkdir imagick
$ tar zxf imagick.tgz -C imagick --strip-components=1
$ rm -rf imagick.tgz
$ git add imagick
$ git commit -m "Add ext/imagick"
```

### Configure PHP
```bash
$ git clean -f -d
$ make distclean
$ ./configure \
$ --prefix=/opt/php/5.6.35 \
$ --with-config-file-path=/opt/php/5.6.35/etc \
$ --with-fpm-user=<your-name> \
$ --with-fpm-group=staff \
$ --with-openssl=/opt/local \
$ --with-zlib=/opt/local \
$ --with-curl=/opt/local \
$ --with-mcrypt=/opt/local \
$ --with-xsl=/opt/local \
$ --with-gd \
$ --with-jpeg-dir=/opt/local \
$ --with-png-dir=/opt/local \
$ --with-zlib-dir=/opt/local \
$ --with-xpm-dir=/opt/local \
$ --with-freetype-dir=/opt/local \
$ --with-readline=/opt/local \
$ --enable-mbstring \
$ --enable-exif \
$ --enable-cli \
$ --enable-pcntl \
$ --enable-sockets \
$ --enable-ftp \
$ --enable-soap \
$ --enable-intl \
$ --enable-zip \
$ --enable-maintainer-zts \
$ --with-mysqli \
$ --with-pdo-mysql \
$ --with-iconv=/opt/local 
```

### Fix Broken Libraries
Make the following changes in ``Makefile``:
1. Find the line that defines ``EXTRA_LIBS`` with command ``/^EXTRA_LIBS =`` 

1. Remove ``-lssl``, ``-lcrypto``, ``-liconv``, and ``-lreadline`` from this
   line.

1. Append ``/opt/local/lib/libssl.dylib`` ``/opt/local/lib/libcrypto.dylib``
   ``/opt/local/lib/libiconv.dylib`` ``/opt/local/lib/libreadline.dylib`` to
   this line.

### Install PHP 
```bash
$ make
$ sudo make install
```

### Override PATH Environment Variable
```bash
$ export PATH=/opt/php/5.6.35/bin:$PATH
```

### Install ``ext/opcache``
```bash
$ cd ext/opcache
$ phpize
$ ./configure
$ make
$ sudo make install
```

### Install ``ext/memcached``
```bash
$ cd ext/memcached
$ phpize
$ ./configure --with-libmemcached-dir=/opt/local
$ make
$ sudo make install
```

### Install ``ext/imagick``
```bash
$ cd ext/imagick
$ phpize
$ ./configure --with-imagick=/opt/local
$ make
$ sudo make install
```

### Create ``etc/php.ini``
```bash
$ cd ..
$ sudo cp php.ini-development /opt/php/5.6.35/etc/
```

### Configure ``etc/php.ini``
Make the following changes:
```ini
[PHP]
max_execution_time = 300
max_input_time = 300
memory_limit = 256M
post_max_size = 128M
extension=/opt/php/5.6.35/lib/php/extensions/no-debug-zts-20131226/memcached.so
zend_extension=/opt/php/5.6.35/lib/php/extensions/no-debug-zts-20131226/opcache.so
extension=/opt/php/5.6.35/lib/php/extensions/no-debug-zts-20131226/imagick.so

[Date]
date.timezone = Asia/Shanghai
```

### Create ``etc/php-fpm.ini``
```bash
$ cd /opt/php/5.6.35/etc
$ sudo touch php-fpm.ini
```

### Configure ``etc/php-fpm.ini``
Add the following contents:
```ini
[global]
pid = run/php-fpm.pid
error_log = log/php-fpm.log
include=etc/pool.d/*.ini
```

### Create ``pool.d/www.ini``
```bash
$ mkdir pool.d
$ touch pool.d/www.ini
```

### Configure ``pool.d/www.ini``
Add the following contents:
```ini
[www]
user = <your-name>
group = staff
listen = 127.0.0.1:9000
pm = dynamic
pm.max_children = 10
pm.start_servers = 4
pm.min_spare_servers = 2
pm.max_spare_servers = 6
chdir = /
```

### Create ``net.php.9000.fpm.plist``
```bash
$ sudo touch /Library/LaunchDaemons/net.php.9000.fpm.plist
```

### Configure ``net.php.9000.fpm.plist``
Add the following contents:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Disabled</key>
        <true/>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>net.php.9000.fpm</string>
        <key>ProgramArguments</key>
        <array>
            <string>/opt/php/5.6.35/sbin/php-fpm</string>
            <string>-F</string>
            <string>--fpm-config</string>
            <string>/opt/php/5.6.35/etc/php-fpm.ini</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
```

## Install PHP 7.2.4

### Download PHP
```bash
$ cd /opt/software/php
$ curl -o php-7.2.4.tgz -L http://cn2.php.net/get/php-7.2.4.tar.gz/from/this/mirror
$ mkdir 7.2.4
$ tar zxf php-7.2.4 -C 7.2.4 --strip-components=1
$ rm -rf php-7.2.4.tgz
$ cd 7.2.4
$ git init .
$ git add .
$ git commit -m 'Add original files'
```

### Download ext/memcached
```bash
$ cd ext
$ curl -o memcached.tgz -L https://pecl.php.net/get/memcached-3.0.4.tgz
$ mkdir memcached
$ tar zxf memcached.tgz -C memcached --strip-components=1
$ rm -rf memcached.tgz
$ git add memcached
$ git commit -m "Add ext/memcached"
```

### Download ext/imagick
```bash
$ curl -o imagick.tgz -L https://pecl.php.net/get/imagick-3.4.3.tgz 
$ mkdir imagick
$ tar zxf imagick.tgz -C imagick --strip-components=1
$ rm -rf imagick.tgz
$ git add imagick
$ git commit -m "Add ext/imagick"
```

### Configure PHP
```bash
$ git clean -f -d
$ make distclean
$ ./configure \
$ --prefix=/opt/php/7.2.4 \
$ --with-config-file-path=/opt/php/7.2.4/etc \
$ --with-fpm-user=<your-name> \
$ --with-fpm-group=staff \
$ --with-openssl=/opt/local \
$ --with-zlib=/opt/local \
$ --with-curl=/opt/local \
$ --with-xsl=/opt/local \
$ --with-gd \
$ --with-jpeg-dir=/opt/local \
$ --with-png-dir=/opt/local \
$ --with-zlib-dir=/opt/local \
$ --with-xpm-dir=/opt/local \
$ --with-freetype-dir=/opt/local \
$ --with-readline=/opt/local \
$ --enable-mbstring \
$ --enable-exif \
$ --enable-cli \
$ --enable-pcntl \
$ --enable-sockets \
$ --enable-ftp \
$ --enable-soap \
$ --enable-intl \
$ --enable-zip \
$ --enable-maintainer-zts \
$ --with-mysqli \
$ --with-pdo-mysql \
$ --with-iconv=/opt/local 
```

### Fix Broken Libraries
Make the following changes in ``Makefile``:
1. Find the line that defines ``EXTRA_LIBS`` with command ``/^EXTRA_LIBS =`` 

1. Remove ``-lssl``, ``-lcrypto``, ``-liconv``, and ``-lreadline`` from this
   line.

1. Append ``/opt/local/lib/libssl.dylib`` ``/opt/local/lib/libcrypto.dylib``
   ``/opt/local/lib/libiconv.dylib`` ``/opt/local/lib/libreadline.dylib`` to
   this line.

### Install PHP 
```bash
$ make
$ sudo make install
```

### Override PATH Environment Variable
Append the following line to ``~/.profile`` and ``/var/root/.profile``:
```bash
export PATH=/opt/php/default/bin:$PATH
```

Refresh ``PATH`` environment variable:
```bash
$ source ~/.profile
```

Create PHP default symbolic link:
```bash
$ sudo ln -s /opt/php/7.2.4 /opt/php/default
```

### Install ``ext/opcache``
```bash
$ cd ext/opcache
$ phpize
$ ./configure
$ make
$ sudo make install
```

### Install ``ext/memcached``
```bash
$ cd ext/memcached
$ phpize
$ ./configure --with-libmemcached-dir=/opt/local
$ make
$ sudo make install
```

### Install ``ext/imagick``
```bash
$ cd ext/imagick
$ phpize
$ ./configure --with-imagick=/opt/local
$ make
$ sudo make install
```

### Create ``etc/php.ini``
```bash
$ cd ..
$ sudo cp php.ini-development /opt/php/7.2.4/etc/
```

### Configure ``etc/php.ini``
Make the following changes:
```ini
[PHP]
max_execution_time = 300
max_input_time = 300
memory_limit = 256M
post_max_size = 128M
extension=/opt/php/7.2.4/lib/php/extensions/no-debug-zts-20170718/memcached.so
zend_extension=/opt/php/7.2.4/lib/php/extensions/no-debug-zts-20170718/opcache.so
extension=/opt/php/7.2.4/lib/php/extensions/no-debug-zts-20170718/imagick.so

[Date]
date.timezone = Asia/Shanghai
```

### Create ``etc/php-fpm.ini``
```bash
$ cd /opt/php/5.6.35/etc
$ sudo touch php-fpm.ini
```

### Configure ``etc/php-fpm.ini``
Add the following contents:
```ini
[global]
pid = run/php-fpm.pid
error_log = log/php-fpm.log
include=etc/pool.d/*.ini
```

### Create ``pool.d/www.ini``
```bash
$ mkdir pool.d
$ touch pool.d/www.ini
```

### Configure ``pool.d/www.ini``
Add the following contents:
```ini
[www]
user = <your-name>
group = staff
listen = 127.0.0.1:9002
pm = dynamic
pm.max_children = 10
pm.start_servers = 4
pm.min_spare_servers = 2
pm.max_spare_servers = 6
chdir = /
```

### Create ``net.php.9002.fpm.plist``
```bash
$ sudo touch /Library/LaunchDaemons/net.php.9002.fpm.plist
```

### Configure ``net.php.9002.fpm.plist``
Add the following contents:
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
    <dict>
        <key>Disabled</key>
        <true/>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>net.php.9002.fpm</string>
        <key>ProgramArguments</key>
        <array>
            <string>/opt/php/7.2.4/sbin/php-fpm</string>
            <string>-F</string>
            <string>--fpm-config</string>
            <string>/opt/php/7.2.4/etc/php-fpm.ini</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
    </dict>
</plist>
```

## Install Composer
```bash
$ php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
$ php -r "if (hash_file('SHA384', 'composer-setup.php') === '544e09ee996cdf60ece3804abc52599c22b1f40f4323403c44d44fdfdd586475ca9813a858088ffbc1f233e9b180f061') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
$ php composer-setup.php
$ php -r "unlink('composer-setup.php');" 
$ sudo mv composer.phar /usr/local/bin/composer
```

## Install Nginx

### Install through MacPorts
```bash
$ sudo port install nginx
```

### Configure ``nginx.conf``
```bash
$ cd /opt/local/etc/nginx
$ sudo mv nginx.conf nginx.conf.from.install
$ sudo touch nginx.conf
```
Add the following contents:
```nginx
user  <your-name> staff;
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;
    include sites-enabled/*.conf;
}
```

### Restore other Configuration Files

#### Configure ``sites-available``
```bash
$ sudo mkdir sites-available
$ # Restore all available site conf files to this directory.
```

#### Configure ``sites-enabled``
```bash
$ sudo mkdir sites-enabled
$ # Enable a site by creating a symbolic link of its conf file from the
$ # sites-available directory.
```

#### Restore other Configuration Files
Restore other files, such as ``ez_params.d`` and any site-specific configuration
files.

## Install Varnish

### Install through MacPorts
```bash
$ sudo port install varnish
```

### Configure ``varnish.conf``
```bash
$ sudo mv varnish.conf varnish.conf.default
$ sudo touch varnish.conf
```
Add the following contents:
```ini
VARNISH_CFG="/opt/local/etc/varnish/default.vcl"                                                                         
                                                                                                                         
VARNISHD_OPTS="-j unix,user=nobody                                                                                       
               -a 0.0.0.0:80                                                                                             
               -f $VARNISH_CFG                                                                                           
               -T localhost:6082                                                                                         
               -s malloc,64M"
```

### Restore default.vcl
```bash
$ sudo mv default.vcl default.vcl.default
$ sudo touch default.vcl
```
Restore ``default.vcl`` from backup.

## Restore Databases
```bash
$ mysql -uroot -e 'create database <database-name> character set utf-8 collate utf8_general_ci'
$ mysql -uroot <database-name> < <database-name>.sql
```

## Restore ``~/zerustech/``
Restore ``~/zerustech/`` from backup.

## Restore Web Sites
```bash
$ sudo mkdir -p /opt/ezpublish/sites
# Restore all websites into this directory from backup. 
```

## Install Nodejs
```bash
$ sudo port install nodejs9
$ sudo port install npm5
```

## Install JDK
```bash
$ java --version
```

## Restore git Configuration
Restore ``~/.gitconfig`` and ``~/.git-credentials`` from backup.

## Restore vim-profile
Follow the instructions in ``README.md`` from zerustech/vim-profile [[3]] to
setup vim profile.

## Services

The services, as well as their start/stop scripts, that are related to PHP
development are as follows.

### mysql57-server
```bash
$ sudo port load mysql57-server
$ sudo port unload mysql57-server
```

### nginx
```bash
$ sudo port load nginx
$ sudo port unload nginx
```

### varnish
```bash
$ sudo port load varnish
$ sudo port unload varnish
```

### memcached
```bash
$ sudo port load memcached
$ sudo port unload memcached
```

### php5-fpm
```bash
$ sudo launchctl load -w /Library/LaunchDaemons/net.php.9000.fpm.plist
$ sudo launchctl unload -w /Library/LaunchDaemons/net.php.9000.fpm.plist
```

### php7-fpm
```bash
$ sudo launchctl load -w /Library/LaunchDaemons/net.php.9002.fpm.plist
$ sudo launchctl unload -w /Library/LaunchDaemons/net.php.9002.fpm.plist
```

## Install Adobe CS6
1. Insert the installation DVD into the DVD drive, and create an ISO image as
follows, which will be used for any installations in the future to protect the
DVD from wear and tear:
```bash
$ diskutil list
$ # Find the <disk-name> of the Adobe CS6 dvd.
$ diskutil umount /dev/<disk-name>
$ sudo dd if=/dev/<disk-name> of=/opt/software/purchased/adobe-cs6-en.iso
```
2. Install Adobe CS6 from the ISO image.

1. Download and install the legacy JRE [[4]], which is required by Adobe
   Illustrator.

1. After the installation has been completed, reboot Mac, then start
   ``Photoshop`` and choose ``Help > updates ...`` from menu to check for
   updates. If Mac is not rebooted immediately after the installation, the
   update will fail with a "server not responding" error.

## Install Parallels Desktop 11
1. Download and install Parallels Desktop 11 [[7]].
1. Re-install Parallels Tools
1. Fix the "networking not available" issue:
```bash
$ sudo launchctl stop com.parallels.desktop.launchdaemon
$ sudo launchctl start com.parallels.desktop.launchdaemon
```

## Install other Applications
1. Download and install Thunderbird [[5]].
1. Download and install LibreOffice [[6]].
1. Install Netease Music from AppStore.
1. Download and install Things 2 [[8]].
1. Download and install OmniGraffle 6 [[9]].
1. Install Folx Go from AppStore.
1. Install WeChat from AppStore.
1. Install TypeFu from AppStore.
1. Install PhotoSweeper from AppStore.
1. Download and install Firefox [[10]].
1. Download and install Chrome [[11]].
1. Download and install xquartz [[12]] (optional).
1. Download and install XMind [[13]].
1. Download and install Skype [[14]]. 

## Restore Application Preferences

Restore the preference directories for each application as follows.

### Firefox
```bash
~/Library/Application\ Support/Firefox
~/Library/Preferences/org.mozilla.firefox.plist
```

### Thunderbird
```bash
~/Library/Thunderbird
``` 

### LibreOffice
```bash
/Library/Application\ Support/LibreOffice
``` 

### Things
```bash
~/Library/Preferences/com.culturedcode.things.plist
~/Library/Containers/com.culturedcode.things
~/Library/Cookies/com.culturedcode.things.binarycookies
```

### OmniGraffle
```bash
~/Library/Application\ Support/Omni\ Group/Software
~/Library/Preferences/com.omnigroup.OmniGraffle6.plist
~/Library/Containers/com.omnigroup.OmniGraffle6
~/Library/Containers/com.omnigroup.GraffleLayout
```

### Netease Music
```bash
~/Library/Preferences/com.netease.163music.plist
~/Library/Containers/com.netease.163music
~/Library/Caches/com.netease.163music
~/Library/Cookies/com.netease.163music.binarycookies
```

## References
|A|B|
-|-
\[1\] [MacPorts][1] | \[2\] [Missing Library Symbols while Compiling PHP][2]
\[3\] [zerustech/vim-profile][3] | \[4\] [Legacy JRE for Adobe Illustrator][4]
\[5\] [Thunderbird][5] | \[6\] [LibreOffice][6]
\[7\] [Parallels Desktop 11][7] | \[8\] [Things 2][8]
\[9\] [OmniGraffle 6][9] | \[10\] [Firfox][10]
\[11\] [Chrome][11] | \[12\] [Xquartz][12]
\[13\] [XMind][13]  | \[14\] [Skype][14]

 [1]: https://guide.macports.org/#installing.macports "MacPorts"
 [2]: https://blog.yimingliu.com/2009/02/24/missing-library-symbols-while-compiling-php-528/ "Missing Library Symbols while Compiling PHP"
 [3]: https://github.com/zerustech/vim-profile "zerustech/vim-profile"
 [4]: https://support.apple.com/kb/DL1573?viewlocale=en_US&locale=en_US "Legacy JRE for Adobe Illustrator"
 [5]: https://www.thunderbird.net/en-US/ "Thunderbird"
 [6]: https://www.libreoffice.org/ "LibreOffice"
 [7]: https://www.parallels.com/cn/products/desktop/download/ "Download Parallels Desktop"
 [8]: https://support.culturedcode.com/customer/portal/articles/2803601 "Things2"
 [9]: https://www.omnigroup.com/download/ "OmniGraffle 6"
[10]: https://www.mozilla.org/en-US/firefox/new/ "Firefox"
[11]: https://www.google.ca/chrome/ "Chrome"
[12]: https://www.xquartz.org/ "Xquartz"
[13]: http://www.xmind.net/download/mac/ "XMind"
[14]: http://www.skype.com "Skype" 
