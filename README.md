# Magento Develop
Development environment for Magento with dockers for M1 and M2

## Depends
This repository requires that the Magento Docker project has started with Nginx Proxy, base PHP, MySQL and Redis and running. 
You will need to create your own PHP container based on the base PHP image `magento/php`.

## Start
Select either the m1 or m2 folder. Configure your container names and virtual host name in the `.env` file:
```
; Nginx
NGINX_CONTAINER=mysite_nginx
VIRTUAL_HOST=local.mysite.com
LETSENCRYPT_HOST=local.mysite.com
LETSENCRYPT_EMAIL=admin@mysite.com

; PHP 
PHP_CONTAINER=mysite_php
LOCAL_PORT=19001 ; host port that points to your PHP container
```
In the `etc/nginx/conf.d/default.conf`, add your server name (i.e. "local.mysite.com").
Create a `www` directory to install Magento to. Run `docker-compose up -d`.

## Composer
Go here https://getcomposer.org/download/ and get the commands. Then log into your PHP container `docker exec -it --user root mysite_php bash` and run the commands, for example:
```
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```

## Magento 2
Install Magento 2:
```
cd /var/www
composer create-project --repository=https://repo.magento.com/ magento/project-enterprise-edition html
    Authentication required (repo.magento.com):
      Username: 50cf47429287dr299f82e817a7cd1279
      Password: 
```
After installation, run setup:
```
cd html
php bin/magento setup:install \
--base-url=http://local.mysite.com/ \
--db-host=mysqldb --db-name=mysite \
--db-user=root --db-password=root \
--admin-firstname=John --admin-lastname=Doe \
--admin-email=jdoe@mysite.com --admin-user=jdoe --admin-password=testing123 \
--language=en_US --currency=USD --timezone=America/New_York --use-rewrites=1
```