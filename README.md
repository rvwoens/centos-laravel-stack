## simple Centos/Nginx/Php-fpm/node/composer stack for Zero-Downtime deployment
[![GitHub Release](https://img.shields.io/github/v/release/rvwoens/centos-laravel-stack.svg?style=flat)](https://github.com/rvwoens/centos-laravel-stack)
[![Last commit](https://img.shields.io/github/last-commit/rvwoens/centos-laravel-stack)](https://github.com/rvwoens/centos-laravel-stack)
[![License](https://poser.pugx.org/cosninix/cos/license)](https://github.com/rvwoens/centos-laravel-stack)
[![Actions Status](https://github.com/rvwoens/centos-laravel-stack/workflows/CI/badge.svg)](https://github.com/rvwoens/centos-laravel-stack/actions)

### configure a bare centos server with
[![Centos version](https://img.shields.io/badge/CentOS-8-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Centos version](https://img.shields.io/badge/PHP-7.4-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Centos version](https://img.shields.io/badge/Node-12-blue)](https://github.com/rvwoens/centos-laravel-stack)
* CentOS 
* Nginx and virtual hosts
* Php fpm & cli, composer  
* Node, npm, yarn
* Basic security 
* Add laravel projects with Zero-downtime deployment 

The scripts are all bash, and kept very simple so you can tweak them to your needs if you want to.

The Zero-downtime deployment setup makes it very easy to deploy a product version on the server by by calling the ```puller``` script   

### how to start

* bring up a bare centos 8 server somewhere
* ssh as root and run:

```bash
yum -y install tar
curl -s -L https://github.com/rvwoens/centos-laravel-stack/archive/v1.0.42.tar.gz | tar -xz
cd centos-laravel-stack-1.0.42
./setup_full

```

Now you can run complete setups:

- ```setup_full``` (part 01..11)
- Optionally run ```setup_clean``` (part 01..06 only) The clean setup does not install php, mariadb, node, nginx
- Manually only install part of the setup from the parts directory for instance all parts except part ```08_node_npm```

### parts

parts are numbered to show the order in which they should be run.

#### 01 user create
create the default user and allow sudo. Block using passwords and only allow access using your public key for security.

* asks for username
* asks for ssh public key to access this account remotely. Paste the contents of your ~/.ssh/id_rsa.pub

#### 02 set hostname
changes the hostname

#### 03 repos and yums
Add remi and epel repo, updates the system and ```yum``` the basics

#### 04 root password
secures the root user

#### 05 security
firewalls, sshd security and fail2ban - disable password logins and secure it further with ```fail2ban```

#### 06 global settings
color prompts, timezone settings, ntp server, fortune and cowsay (just for fun)

#### 07 php
Set up php 74 including fpm and composer

#### 08 node and npm
Installs node, npm and yarn

#### 09 mysql / mariadb
installs mariadb and sets it up for production

* MariaDB asks some questions during install, like the root password

#### 10 nginx
set up nginx for php-fpm and prepare for multiple virtual hosts with the sites-enabled/sites-available pattern

#### 11 laravel
Actually no laravel project is installed, but everything is prepared for adding a zero-downtime-deployment project using the ```addzhost``` command

#### Finally
logout as root. At this moment the server is ready for project deployments but none is deployed yet.

### Adding new laravel app with Zero Downtime Deployment
Log in as the default (created) user and run
```
    addzhost <<domain>> <<git repository>> <<initial tag>>
```

- adds /var/www/[domain] for the laravel app (make sure to configure DNS to the IP of this server)
- creates a nginx config in /etc/nginx/sites-available and enables it
- adds /var/www/[domain]/releases for each inidividual release
- adds /var/www/[domain]/storage which is linked into each release
- adds /var/www/[domain]/.env which is linked into each release

- generates a ```puller``` script (in /var/www/[domain]) to pull a version (tag) from the repository and release it
- generates a ```rollback``` script to roll back to a previous release (any of the releases available in the releases dir)

- uses ```puller``` to release the initial tag into production

Example:
    ```addzhost myapp.example.com git@gitlab.com:myprojects/myapp.git v1.0.3```
    Now you can browse to myapp.example.com and enjoy your v1.0.3 release (make sure DNS has been set up)

#### puller
Usage: puller [tag]
- Creates a new release in the releases directory and use ```git archive``` to download the release
- create symbolic links of ```/var/www/[domain]/storage``` and ```/var/www/[domain]/.env``` into the release
- runs ```composer install```
- You can create a custom post-deployment script called ```after_deploy``` with your own artisan/composer/npm commands.
a typical production_deploy could look like this 
    ```
    echo "DEPLOY"
    php artisan migrate --force
    yarn install
    yarn run production
    ```
    Make sure ```after_deploy``` is executable.
- Makes sure everything has the right chown and chmod applied
- swaps the /var/www/[domain]/current link towards the new release to make it available with zero downtime

#### rollback
Usage: rollback [tag]
- swaps the /var/www/[domain]/current link to the old release to make it available with zero downtime


### extra utilities in ~/bin
* artisan - make sure you only need to type 'artisan' (on any laravel top directory)
* clear-laravel -  clears all possible caches and restores path permissions
* logtail - tails the latest laravel log for a live logview (from any laravel top directory)






 
