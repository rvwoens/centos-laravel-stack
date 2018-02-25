## simple centos7 server install for laravel
some helper scripts for installing a centos7 server with

* Nginx with virtual hosts
* Php 7.1 fpm & cli  
* Laravel
* Security 
* Deployment by pushing a git repo


###how to start

* bring up a centos 7 server somewhere (I use linode boxes and Google cloud compute engines)
* ssh as root and run:

```bash
yum -y install git
git clone https://github.com/rvwoens/centos7serverinstall.git
cd  centos7serverinstall
./stage1
```

### stage 1

- add a default user
- set a hostname
- lock passwords and set autorized keys
- allow sudo's
- cosmetic settings
- add neccesary repos
- yum the basics




 
