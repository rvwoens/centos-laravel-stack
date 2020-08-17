## simple centos/nginx/php-fpm/node/npm/yarn/composer stack
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
* A git repo for pushing deployments 

The scripts are all bash, and kept very simple so you can tweak them to your needs if you want to.

The git repository setup makes it very easy to deploy a product version to the server by creating a remote repo and push the branch/tag to the server.  

### how to start

* bring up a centos 8 server somewhere
* ssh as root and run:

```bash
yum -y install tar
wget -q -O- https://github.com/rvwoens/centos-laravel-stack/archive/v1.0.32.tar.gz | tar -xz
cd centos-laravel-stack-1.0.32
./setup_full
```

Now you can run complete setups:

- setup_clean (part 01..06 only) The clean setup does not install php, mariadb, node, nginx
- setup_full (part 01..11)

or you can only install part of the setup from the parts directory

### parts

parts are numbered to show the order in which they should be called.

#### 01 user create
create the default user and allow sudo

* asks for username
* asks for ssh public key to access this account remotely. Paste the contents of your ~/.ssh/id_rsa.pub

#### 02 set hostname
changes the hostname

#### 03 repos and yums
Add remi and epel repo, updates the system and yums the basics

#### 04 root password
secures the root user

#### 05 security
firewalls, sshd security and fail2ban

#### 06 global settings
color prompts, timezone settings, ntp server, fortune and cowsay (important!)

#### 07 php
Set up php 74 including fpm and composer

#### 08 node and npm
Installs node and npm

#### 09 mysql / mariadb
installs mariadb and sets it up for production

* MariaDB asks some questions during install

#### 10 nginx
set up nginx for php-fpm and multiple virtual hosts

#### 11 git repository
It's very convenient to use a git repository on your server for deployment of new versions.
* A directory is created at ```~/git``` on the user account (the user as entered in part 01) where all deployment repositories are located.
* The git work directory is located at ```/var/www/[git repo domain]``` so checking out results in a new version deployed publically.
* Extra hooks are provided to automatically check out the tag or branch that is pushed. So you only need to push a release, the rest is automatic.

You can also create a script in your project root directory called ```production_deploy``` that is called after the checkout, for instance to do migrations.

#### Finally
logout as root

#### Adding new virtual host for a domain
Log in as the default (created) user and run

```
    addvhost <<domain>>
```

- adds /var/www/<domain> for the laravel app (make sure to configure DNS to the IP of this server)
- creates a nginx config in /etc/nginx/sites-available and enables it
- creates an empty git repository under ~/git/<domain>
- creates a receive-hook for this repository so it automatically updates the laravel app at /var/www/<domain> after a push

On the development machine just add:
```
git remote add web ssh://<user>@<host>/~/git/<domain>
git push web master
or
git push web <tag|branchname>
```
and the pushed tag/branch is automatically checked out to you server's path and
updated. 

You can create a custom post-deployment script called ```production_deploy``` with your own artisan/composer/npm commands.
a typical production_deploy could look like this:
```
echo "DEPLOY"
echo ">>>>>>>>> artisan DOWN"
php artisan down
echo ">>>>>>>>> composer INSTALL"
composer install
echo ">>>>>>>>> artisan MIGRATE"
php artisan migrate --force
echo ">>>>>>>>> yarn INSTALL"
yarn install
echo ">>>>>>>>> yarn run PRODUCTION"
yarn run production
echo ">>>>>>>>> artisan UP"
php artisan up
```







 
