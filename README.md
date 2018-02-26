## simple centos7 server install for laravel
some helper scripts for installing a centos7 server with

* Nginx with virtual hosts
* Php 7.1 fpm & cli  
* Laravel
* Security 
* Deployment by pushing a git repo

The scripts are all bash, and kept very simple so you can tweak them to your needs.

The git-repo deployment makes it very easy to deploy a git laravel version to the (staging/production) server by calling:
```
git push web -f
or
git push web <tag|branchname> -f
```
and the pushed tag/branch is automatically checked out to you server's path and
updated (composer update, dump-autoload, artisan migrate, npm update, npm run dev or gulp etc.)


### how to start

* bring up a centos 7 server somewhere
* ssh as root and run:

```bash
yum -y install git
git clone https://github.com/rvwoens/centos7serverinstall.git
cd  centos7serverinstall
```

Now you can run complete setups:

- setup_clean
- setup_full
- more coming soon..

or you can only install part of the setup from the parts directory

### setup clean

The clean setup does not install php, mariadb, node, nginx

### setup full

Setup full includes php, mariadb, node/npm, nginx, git repo deployment and sets up a multihost environment


### parts

#### 01 user create
create the default user and allow sudo

#### 02 set hostname
changes the hostname

#### 03 global settings
color prompts, timezone settings, ntp server, fortune and cowsay

#### 04 root password
secures the root user

#### 05 security
firewalls, sshd security and fail2ban

#### 06 repos and yums
Add remi and epel repo, updates the system and yums the basics

#### 07 php71
Set up php 71 including fpm and composer

#### 08 node and npm
Installs node and npm

#### 09 mysql / mariadb
installs mariadb and sets it up for production





 
