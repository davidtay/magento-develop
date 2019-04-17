# Magento Develop
Development environment for Magento with dockers for M1 and M2

## Start
This repository requires that the [Magento Docker](https://github.com/davidtay/magento-docker) 
project has started and that Nginx Proxy, base PHP, MySQL and Redis are running. 

Copy either the m1 or m2 folder to your project folder. 
Configure your container names, PHP version and virtual host in the `.env` file:

```
cp m2 mysite
cd mysite
vi .env
```


```
; Nginx
NGINX_CONTAINER=mysite_nginx
VIRTUAL_HOST=www.mysite.local
VIRTUAL_PROTO=https
VIRTUAL_PORT=443

; PHP 
PHP_VERSION=7.2
PHP_CONTAINER=mysite_php
LOCAL_PORT=19002
```

Create a `www` directory to install Magento to and then run `docker-compose up -d`:

```
mkdir www
docker-compose up -d
```

## Composer
Go to the [Composer](https://getcomposer.org/download/) download page and get the commands. Then log into your PHP container `docker exec -it --user root mysite_php bash` and run the commands, for example:

```
cd /var/www
php -r "copy('https://getcomposer.org/installer', 'composer-setup.php');"
php -r "if (hash_file('sha384', 'composer-setup.php') === '48e3236262b34d30969dca3c37281b3b4bbe3221bda826ac6a9a62d6444cdb0dcd0615698a5cbe587c3f0fe57a54d8f5') { echo 'Installer verified'; } else { echo 'Installer corrupt'; unlink('composer-setup.php'); } echo PHP_EOL;"
php composer-setup.php --install-dir=/usr/local/bin --filename=composer
php -r "unlink('composer-setup.php');"
```

## Magento 2
Create your Magento 2 project and download the Magento 2 codebase:

```
cd /var/www
composer create-project --repository=https://repo.magento.com/ magento/project-enterprise-edition html
    Authentication required (repo.magento.com):
      Username: 50cf47429287dr299f82e817a7cd1279
      Password: 
```

then run setup:

```
cd /var/www/html
php bin/magento setup:install \
--base-url=https://www.mysite.local/ \
--db-host=mysqldb --db-name=mysite \
--db-user=root --db-password=root \
--admin-firstname=John --admin-lastname=Doe \
--admin-email=jdoe@mysite.com --admin-user=jdoe --admin-password=testing123 \
--language=en_US --currency=USD --timezone=America/New_York --use-rewrites=1
```