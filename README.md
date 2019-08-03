# Install-Firefly-iii-on-freenas
#steps taken to install Firefly-iii in Freenas 11.2 jail

#Source: https://www.samueldowling.com/2018/12/08/install-nextcloud-on-freenas-iocage-jail-with-hardened-security/
#FAMP - Freenas/Apache24/MariaDB setup

#Create databases for data and db for firefly

#Create mysql user and group

#set permissions for db for mysql

#ssh into freenas and create iocage jail

    sudo iocage create -n firefly-iii -r 11.2-RELEASE ip4_addr="vnet0|INTERNAL IP ADDR/24" defaultrouter="ROUTER IP ADDR" vnet="on" allow_raw_sockets="1" boot="on"

#stop jail and add storages - at least db - in gui?

#create folder in jail if not there

#start firefly-iii

    sudo iocage start firefly-iii

###DID NOT WORK#optimize metadata? - 

          sudo zfs set primarycache=metadata /mnt/Tank3/firefly-iii/db

          --cannot open '/mnt/Tank3/firefly-iii/db': leading slash in name

#go into console for firefly-iii

    sudo iocage console firefly-iii

#optional- change to latest releases

#install apache24 and mariadb

    pkg install nano apache24 mariadb102-server
  
#enable services and start

    sysrc apache24_enable=yes

    service apache24 start

    sysrc mysql_enable=yes

    service mysql-server start

#check they are running
  
    service apache24 status
   
    service mysql-server status
  
#set up mysql

    mysql_secure_installation

####TRIED AFTER INSTALL WITH COMPOSER- FAILS SEE LINE 68 #configure database for firefly-iii?

       mysql -u root -p
  
       CREATE DATABASE fireflyiii;
       CREATE USER 'fireflyiii_admin'@'localhost' IDENTIFIED BY 'YOUR PASSWORD'
       GRANT ALL ON fireflyiii.* to fireflyiii_admin@localhost;
    ------NOT WORKING-----can't find any matching row in the user table
       FLUSH PRIVILEGES;
       exit

#Source: https://docs.firefly-iii.org/en/latest/installation/server.html
  #https://kifarunix.com/install-apache-mysql-php-famp-stack-on-freebsd-12/
#Install PHP and firefly-iii

#install php72 and needed extensions

    pkg install php72 php72-mysqli mod_php72 php72-mbstring php72-zlib php72-curl php72-gd php72-json php72-bcmath php72-intl php72-ldap php72-xml php72-simplexml php72-fileinfo php72-filter php72-hash php72-iconv php72-openssl php72-pdo php72-tokenizer php72-zip php72-pdo_mysql php72-phar php72-pdo_pgsql php72-pgsql php72-pdo_sqlite 

#install composer

    curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer
  
#cd to www directory

    cd /usr/local/www
  
#install firefly-iii

    composer create-project grumpydictator/firefly-iii --no-dev --prefer-dist firefly-iii 4.7.17.3
  
  php artisan complains about something-----
  
  > Illuminate\Foundation\ComposerScripts::postAutoloadDump
  > @php artisan firefly:instructions install
  Script @php artisan firefly:instructions install handling the post-install-cmd event returned with error code 255

----continue anyways------

#change permissions

          sudo chown -R www-data:www-data firefly-iii - NOT USED_ NO USER www-data
    chmod -R 775 firefly-iii/storage

#edit .env file?

    cd firefly-iii
  
    nano .env

#initialize the database

    php artisan migrate:refresh --seed

    php artisan firefly:upgrade-database

    php artisan firefly:verify

    php artisan passport:install

DONE?

INTERNAL IP ADDR/firefly-iii/
------NOT FOUND---------

  nano /usr/local/etc/apache24/httpd.conf

  <IfModule dir_module>
     DirectoryIndex index.php index.html
  </IfModule>
  
#add to end

  <FilesMatch "\.php$">
    SetHandler application/x-httpd-php
  </FilesMatch>
  <FilesMatch "\.phps$"> 
    SetHandler application/x-httpd-php-source
  </FilesMatch>
  
#restart apache24

Still no worky- goto 63 trying to set up db- fail on that too

Will try ubuntu-server vm and follow this guide:
https://gist.github.com/philthynz/ec04833a8e39c7f7d1b0d33cb4197a95


