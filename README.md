# Magento Develop
Docker development environment for Magento 2

## Start
This project requires that the [Magento Docker](https://github.com/davidtay/magento-docker) 
project has started and that Nginx Proxy, base PHP, MySQL, Elasticsearch and Redis are running. 

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

Start up your nginx/php instance.

```
docker-compose up -d
```


## Magento 2
Create your Magento 2 project and download the Magento 2 codebase:

```
cd /var/www/html
composer create-project --repository=https://repo.magento.com/magento/project-community-edition .
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