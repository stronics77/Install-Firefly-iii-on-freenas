# Install-Firefly-iii-on-freenas
#steps taken to install Firefly-iii in Freenas 11.2 jail 

This works on ubuntu-server18 vm with bhyve:

https://old.reddit.com/r/FireflyIII/comments/8thxuu/fireflyiii_on_ubuntu_server_1604lts_nginx_php72/

So... let's try one more time with nginx on FN 11.2

source: https://www.cyberciti.biz/faq/freebsd-install-nginx-webserver/

pkg install nano nginx php72 php72-extensions php72-ldap php72-gd php72-bcmath php72-intl php72-curl php72-mbstring php72-zip php72-openssl php72-zlib php72-fileinfo php72-pdo_mysql mariadb102-server

sysrc *_enable=YES
- for all services, nginx, mysql-server

start services

set up db

mysql_secure_installation

        mysql_secure_installation
        mysql -u root -p 
        CREATE DATABASE firefly; 
        GRANT ALL ON firefly.* TO firefly@localhost IDENTIFIED BY 'your_password'; 
        \q



DONT THINK I NEED THIS_ MAYBE TRY WITHOUT
nano /usr/local/etc/php/99-custom.ini

display_errors=Off
safe_mode=Off
safe_mode_exec_dir=
safe_mode_allowed_env_vars=PHP_
expose_php=Off
log_errors=On
error_log=/var/log/nginx/php.scripts.log
register_globals=Off
cgi.force_redirect=0
file_uploads=On
allow_url_fopen=On
sql.safe_mode=Off
disable_functions=show_source, system, shell_exec, passthru, proc_nice, exec
max_execution_time=60
memory_limit=60M
upload_max_filesize=2M
post_max_size=2M
cgi.fix_pathinfo=0
sendmail_path=/usr/sbin/sendmail -email -t

cp -v /usr/local/etc/php.ini-production /usr/local/etc/php.ini


NGINX CONFIG: (put behind a proxy at some point???) - only lan no wan access atm

server {
    listen 80 default_server;
    listen [::]:80 default_server;
    root /firefly-iii/public;
    index index.php index.html index.htm index.nginx-debian.html;
    server_name _;
    location / {
        try_files $uri $uri/ /index.php?$query_string;
        autoindex on;
        sendfile off;
    }
      location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/run/php/php7.2-fpm.sock;
      }
      location ~ /\.ht {
      deny all;
      }
      
    **does not like the include snippets/fastcgi-php.conf; line- no directory snippets found.- need to look into


curl -sS https://getcomposer.org/installer | php -- --install-dir=/usr/local/bin --filename=composer

composer create-project grumpydictator/firefly-iii --no-dev --prefer-dist firefly-iii 4.7.17.3

chown -R www:www firefly-iii

php artisan migrate:refresh --seed

php artisan passport:install

An error occurred. ----RATS----


