## simple centos7 laravel 5.x nginx php-fpm stack
configure a centos7 server with

* Nginx and virtual hosts
* Php 7.1 fpm & cli  
* Laravel
* Basic security 
* Deployment by pushing a git repo

The scripts are all bash, and kept very simple so you can tweak them to your needs.

The git repository setup makes it very easy to deploy a laravel version to the server by creating a remote repo and push the branch/tag to the server.  

### how to start

* bring up a centos 7 server somewhere
* ssh as root and run:

```bash
yum -y install git
git clone https://github.com/rvwoens/centos7-laravel-stack.git
cd centos7-laravel-stack
./setup_full
```

Now you can run complete setups:

- setup_clean
- setup_full
- setup_docker (coming soon..)

or you can only install part of the setup from the parts directory

### setup clean

The clean setup does not install php, mariadb, node, nginx

### setup full

Setup full includes php, mariadb, node/npm, nginx, git repo deployment and sets up a multihost environment


### parts

parts are numbered to show the order in which they should be called.

#### 01 user create
create the default user and allow sudo

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

#### 07 php71
Set up php 71 including fpm and composer

#### 08 node and npm
Installs node and npm

#### 09 mysql / mariadb
installs mariadb and sets it up for production

#### 10 nginx
set up nginx for php-fpm and multiple virtual hosts

#### 11 git repository

Actually this uses a separate script 'addvhost' (run this as the created user, not as root. Its in its path)
- add /var/www/<domain> for the laravel app
- creates a nginx config in /etc/nginx/sites-available
- creates an empty git repository under ~/git/<domain>
- creates a receive-hook for this repository so it automatically updates the laravel app at /var/www/<domain> after a push

On the development machine just add:
```
git remote add web ssh://<user>@<host>/~/git/<domain>
git push web -f
or
git push web <tag|branchname> -f
```
and the pushed tag/branch is automatically checked out to you server's path and
updated (composer update, dump-autoload, artisan migrate, npm update, npm run dev or gulp etc.)
Check out the file post-receive in the hooks directory at ~/git/<domain>/hooks 





 
