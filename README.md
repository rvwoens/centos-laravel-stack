# simple Centos Laravel stack for Zero-Downtime deployment
[![GitHub Release](https://img.shields.io/badge/release-3.0.19-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Last commit](https://img.shields.io/github/last-commit/rvwoens/centos-laravel-stack)](https://github.com/rvwoens/centos-laravel-stack)
[![License](https://poser.pugx.org/cosninix/cos/license)](https://github.com/rvwoens/centos-laravel-stack)
[![Actions Status](https://github.com/rvwoens/centos-laravel-stack/workflows/CI/badge.svg)](https://github.com/rvwoens/centos-laravel-stack/actions)

## Installs a fresh server with
[![Centos version](https://img.shields.io/badge/centos-8%209%20stream-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![PHP version](https://img.shields.io/badge/PHP-8.0%208.1%208.2-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![NGINX version](https://img.shields.io/badge/Nginx-1.20.1-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Node version](https://img.shields.io/badge/Node-16-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Redis version](https://img.shields.io/badge/Redis-6.2-blue)](https://github.com/rvwoens/centos-laravel-stack)
[![Laravel version](https://img.shields.io/badge/Laravel-v5%20v6%20v7%20v8%20v9%20v10-blue)](https://github.com/rvwoens/centos-laravel-stack)
* CentOS
* Nginx and virtual hosts
* Php fpm & cli, composer
* Nvm, Node, npm, yarn
* Redis
* Security and Hardening
* Add (multiple) laravel projects (any laravel version) with Zero-Downtime deployment

The scripts are all bash, and kept very simple so you can tweak them to your needs if you want to.

Very easy project deployment by calling the project's  ```puller``` script

## how to install

* bring up a bare centos server somewhere (centOS 8, 9 and stream supported)
* ssh as root and run:

```bash
yum -y install tar
curl -s -L https://github.com/rvwoens/centos-laravel-stack/archive/v3.0.19.tar.gz | tar -xz
cd centos-laravel-stack-3.0.19
./setup_full
```

That's it (estimated 30 minutes to run, and stops for some input). Now you can add laravel apps:

## Add a new laravel app with Zero Downtime Deployment
Log in as the default (created) user and run
```
    addzhost <<domain>> <<git repository>> <<initial tag>>
```

### Note:
You need access privileges from this server on the repository.
- For github or gitlab, Go to settings->ssh keys and add the contents of ```~/.ssh/id_rsa.pub``` to a new ssh key.
- As an alternative you can use the readonly ```https://github.com/<user>/<repo>.git``` (default) repo url, but in this case the puller will just use a git clone

### Directory structure
The zero downtime deployment creates the following directory structure for each app:
```
/var/www/app-domain-name
\_  current   - symlink to the current active release. Nginx serves from here
\_  puller    - puller script to deploy a new release (generated by addzhost)
\_  rollback  - rollback script to go back to an old release (generated by addzhost)
\_  releases  - all releases are kept here
| \_ 1.0.0
| \_ 1.0.1 
| \_ etc
\_  storage   - main laravel storage directory symlinked into all releases
\_ .env       - Main laravel .env symlinked into all releases
```

### Example:

```$ addzhost myapp.example.com git@gitlab.com:myprojects/myapp.git v1.0.3```        
Now you can browse to myapp.example.com and enjoy your v1.0.3 release (make sure DNS has been set up)


- adds ```/var/www/myapp.example.com``` for the laravel app (make sure to configure DNS to the IP of this server)
- creates a nginx config in ```/etc/nginx/sites-available``` and enables it
- creates ```/var/www/myapp.example.com/releases``` dir for each inidividual release
- creates ```/var/www/myapp.example.com/storage``` dir which is linked into each release
- creates (stub) ```/var/www/myapp.example.com/.env``` file which is linked into each release. Tweak this to your needs.


- generates a ```puller``` script (in /var/www/myapp.example.com) to pull a version (tag) from the repository and release it
- generates a ```rollback``` script to roll back to a previous release (any of the releases available in the releases dir)
- uses ```puller``` to release the initial tag into production. if it fails you can rollback everything and try again.

### puller 
Usage: ```$ /var/www/[domain]/puller [tag]```

Puller is generated by addzhost on the top project directory: ```/var/www/[domain]/puller```

- Creates a new release in the releases directory and uses ```git archive``` to download the release
- create symbolic links of ```/var/www/[domain]/storage``` and ```/var/www/[domain]/.env``` into the release
- runs ```composer install```
- You can create a custom post-deployment script in the root of your repository called ```after_deploy``` with your own artisan/composer/npm commands.
  a typical after_deploy could look like this
    ```
    echo "DEPLOY"
    php artisan migrate --force
    yarn install
    yarn run production
    ```
- Makes sure everything has the right chown and chmod applied
- swaps the ```/var/www/[domain]/current``` link towards the new release to make it available with zero downtime

If any of the steps failed ```puller``` aborts before making the release current.

You can call ```puller``` from your local development machine for further integration:

```
server=[user]@[host]
dir=[deploy directory]
tag=`git describe --abbrev=0 --tags`    # latest tag
echo "$dir/puller $tag;" | ssh $server
```


### rollback (generated by addzhost)
Usage: rollback [tag]

Rollback is generated by addzhost on the top project directory: ```/var/www/[domain]/rollback```

- swaps the /var/www/[domain]/current link to the old release to make it available with zero downtime

## Local support

The local_support directory contains a few bash scripts for local deployment.
Used when you need to deploy against multiple instances of your app on multiple servers,
for instance a staging, a beta test and a production machine.

### pusher (local machine)
 you can use the ```pusher``` tool to deploy to a number of instances on any server using a ```pusher.conf``` file.
The ```pusher.conf``` file is expected on the base path (root) of the laravel app.

By default it pushes a new version (last tag obtained from the local git) to the instance 
by using the ```newversion``` tool, however you can also specify a new tag yourself.

For conveniance, a ```config/version.php``` file is written/updated in your laravel project
(and pushed to git) before deploying so you can show the deployed version somewhere in your app.

Pusher is found in the local_support directory together with an example pusher.conf file. 
Pusher calls the appropriate ```puller``` script on the target machine as defined in the ```pusher.conf``` file.

copy the pusher script to a path somewhere so it can be used by multiple projects.

Examples: (assume current dir is the root of a laravel project on a local machine)
```
$ cat pusher.conf
staging     example@exampleserver.com               /var/www/staging-of-myapp
prod        example@productionserver-example.com    /var/www/myapp
betatest    my@exampleserver.com                    /var/www/betatest
```
```
$ pusher betatest
... determine the lastest git tag (for instance 1.3.42)
... increments the version to 1.3.43 and sets a git tag, 
... git push and write config/version.php with 1.3.43
... ssh into server my@exampleserver.com 
... run puller 1.3.43 on the /var/www/betatest/releases directory
```

```
$ pusher prod 1.2.5
... creates a version 1.2.5 and sets a git tag, 
... git push and write config/version.php with 1.2.5
... ssh into server example@productionserver-example.com
... run puller 1.2.5 on the /var/www/myapp directory
```

### newversion (local machine)
Simple bash script that increments a version number.
Use the -x option to avoid incrementing the second version element.

used by the ```pusher``` script to automatically increment the version number
Examples:
```
$ newversion 1.3.4 
1.3.5
$ newversion 1.3.9
1.4.0
$ newversion 1.3.09 
1.3.10 
$ newversion -x 1.3.9 
1.3.10 
```

## Overview of individual installation parts run by ```setup_full```

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
Set up php including fpm and composer

#### 08 node and npm
Optionally Installs node, npm and yarn

#### 09 mysql / mariadb
Optionally installs mariadb and sets it up for production

* MariaDB asks some questions during install, like the root password to be used

#### 10 nginx
set up nginx for php-fpm and prepare for multiple virtual hosts with the sites-enabled/sites-available pattern

#### 11 laravel
Optionally it can install a default laravel project with zero-downtime-deployment project using the ```addzhost``` command

#### 12 redis
Optionally install redis

#### Finally
logout as root. At this moment the server is ready for project deployments.


### extra utilities in ~/bin
* artisan - shortcut to ```php artisan```
* clear-laravel -  clears all possible caches and restores path permissions
* logtail - tails the latest laravel log for a live logview (from any laravel top directory)
* dbRemote2local - copy a remote mysql database to a local database
* release-cleanup - removes old releases, run in the release dir of a project
* 

 