# Magento Develop
Magento 2 project instance for [Magento Docker](https://github.com/davidtay/magento-docker).

## Magento Docker
This project requires that the [Magento Docker](https://github.com/davidtay/magento-docker) 
project has started and that Nginx Proxy, base PHP, MySQL, Elasticsearch and Redis containers are running.

## Create Project
Create your project and copy the etc folder, .env and docker-compose.yml files in this project to your project. 
Suppose this project is in your Documents directory such as ~/Documents/Projects/magento-develop. Then create 
your project directory one level up:

```
mkdir ../mysite
cp -R etc .env docker-compose.yml ../mysite
cd ../mysite
```

Configure your container names, PHP version and virtual host in the `.env` file:

```
;Nginx
NGINX_CONTAINER=mysite_nginx
VIRTUAL_HOST=www.mysite.local
VIRTUAL_PROTO=https
VIRTUAL_PORT=443

;PHP 
PHP_VERSION=7.4
PHP_CONTAINER=mysite_php
LOCAL_PORT=19001
```

Create `www` directory to host your project and start up your nginx/php project instance:

```
mkdir www
docker-compose up -d
```

## Install Magento
Log into your instance `docker exec -it mysite_php bash`. Then install Magento.

```
cd /var/www/html
composer create-project --repository=https://repo.magento.com/magento/project-community-edition .
    Authentication required (repo.magento.com):
      Username: 50cf47429287dr299f82e817a7cd1279
      Password: 
```

then install:

```
cd /var/www/html
php bin/magento setup:install \
--base-url=https://www.mysite.local/ \
--db-host=mysqldb --db-name=mysite \
--db-user=root --db-password=root \
--admin-firstname=John --admin-lastname=Doe \
--admin-email=jdoe@mysite.com --admin-user=jdoe --admin-password=testing123 \
--language=en_US --currency=USD --timezone=America/New_York --use-rewrites=1 \
--elasticsearch-host=es7
```

Run the usual magento commands after installing:

```
php bin/magento setup:upgrade
php bin/magento setup:di:compile
php bin/magento setup:static-content:deploy 
```

## MacOS
You can use nfs to speed up the Mac-Docker filesystem sharing. Edit
`sudo vi /etc/exports`: 

```
/System/Volumes/Data/Users/davidtay/Documents/Projects/mysite/www -alldirs -mapall=501:20 localhost
```

Then restart the nfs daemon: `sudo nfsd restart`

In the docker-compose.yml, uncomment the nsfmount volume and map the PHP and Nginx filesystems:

```
    php: 
      image: magento/php${PHP_VERSION}
      container_name: ${PHP_CONTAINER}
      restart: always
      ports: 
        - "${LOCAL_PORT}:9000"
      volumes:
        - 'nfsmount:/var/www/html'
        - ./etc/php/php.ini:/usr/local/etc/php/conf.d/php.ini
        - ./etc/php/www.conf:/usr/local/etc/php-fpm.d/www.conf
volumes:
    nfsmount:
      driver: local
      driver_opts:
        type: nfs
        o: addr=host.docker.internal,rw,nolock,hard,nointr,nfsvers=3
        device: ":/System/Volumes/Data/${PWD}/www"
```

See https://www.jeffgeerling.com/blog/2020/revisiting-docker-macs-performance-nfs-volumes for more info.