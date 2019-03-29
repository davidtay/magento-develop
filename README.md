# Magento Develop
Development environment for Magento with dockers for M1 and M2

## Depends
This repository requires that the Magento Docker project has started with Nginx Proxy, base PHP, MySQL and Redis and running. You will need to create your own PHP container based on the base PHP container.
`docker commit php mysite/php`

## Start
Go into the m1 or m2 folder and run `docker-compose up -d`.

