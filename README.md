# PHP Laravel Developing Team Environment Build Guide (an Example)

This is an example on how to build PHP Laravel team developing environment and roughly the process. 

- GitHub based
- Developing / testing machines can be on premise Linux hosts or AWS EC2 instances
- PHP Laravel developing / testing ready

## How it works
- Developer clones / forks base / current version PHP Laravel source codes from GitHub main develop branch to his / her developing machine or GitHub account
- Developer develops on his / her developing machine
- Developer pushs his / her updates to his / her own GitHub repo branch for testing
- Developer tests his / her updates by following the process drafted in this guide (the testing process can be easily scripted)
- Developer reviews testing result and makes further updates or raises pull / merge request
- Developer repeats same developing cycle

## GitHub resources
- GitHub team account or main develop repo and branch (use 'https://github.com/fen9li/simple-laravel-app' as an example in this guide)  
- GitHub repos and branch forked from main develop repo by individual developers

## Usage
### Prepare standard developing / testing linux host
Setup standard developing linux host, which is PHP Laravel developing and GitHub ready. The linux host can be an on-premise one or an aws ec2 instance; Can be a physical one or virtual. 

- The standard linux host

```sh
~]# yum -y update
... ...
~]# uname -a
Linux sinatra.fen9.li 3.10.0-514.26.2.el7.x86_64 #1 SMP Tue Jul 4 15:04:05 UTC 2017 x86_64 x86_64 x86_64 GNU/Linux
~]# cat /etc/os-release
NAME="CentOS Linux"
VERSION="7 (Core)"
ID="centos"
ID_LIKE="rhel fedora"
VERSION_ID="7"
PRETTY_NAME="CentOS Linux 7 (Core)"
ANSI_COLOR="0;31"
CPE_NAME="cpe:/o:centos:centos:7"
HOME_URL="https://www.centos.org/"
BUG_REPORT_URL="https://bugs.centos.org/"

CENTOS_MANTISBT_PROJECT="CentOS-7"
CENTOS_MANTISBT_PROJECT_VERSION="7"
REDHAT_SUPPORT_PRODUCT="centos"
REDHAT_SUPPORT_PRODUCT_VERSION="7"

~]# 
```

- Configure firewall
- Install and configure git
- Install httpd
- Install php 7.x +
- Install and configure composer
- Install other necessary software packages

```sh
~]# firewall-cmd --permanent --add-service={http,https}
...
~]# firewall-cmd --reload
...
~]# yum -y install git httpd tree policycoreutils-python
...
~]# rpm -Uvh https://dl.fedoraproject.org/pub/epel/epel-release-latest-7.noarch.rpm
...
~]# rpm -Uvh https://mirror.webtatic.com/yum/el7/webtatic-release.rpm
...
~]# yum -y install php70w php70w-common php70w-gd php70w-phar php70w-xml php70w-cli php70w-mbstring php70w-tokenizer php70w-openssl php70w-pdo
...
~]# php --version
PHP 7.0.23 (cli) (built: Sep 16 2017 12:47:01) ( NTS )
Copyright (c) 1997-2017 The PHP Group
Zend Engine v3.0.0, Copyright (c) 1998-2017 Zend Technologies
~]#
...
~]# curl -sS https://getcomposer.org/installer | php
...
~]# mv composer.phar /usr/bin/composer
...
~]# composer --version
Composer version 1.5.2 2017-09-11 16:59:25
~]# exit
~]$ git config --global user.name "Xxxx Xx"
~]$ git config --global user.email xxx@xxx.xxx
```

### Developing
- Create local base developing directory and configure access permissions (configure as 777 in this example, which can be tighten as per individual requirements).
- Git clone / fork base or current version of source codes from GitHub developers team main developing branch.
```sh
~]# mkdir -m 777 /laradev
~]# exit
~]$ cd laradev
laradev]$ git clone -b develop https://github.com/fen9li/simple-laravel-app
...
laradev]$
```

### Develop / update 
Developer makes his / her updates and pushs to his / her own GitHub individual branch.

### Test new codes
- Create local test base directory and configure access permissions
- Git clone new draft source code to local test base directory
 ```sh
 laratest]$ git clone -b xxx https://github.com/xxxxxx/simple-laravel-app
...
laratest]$
laratest]$ cd /laratest/simple-laravel-app/
simple-laravel-app]$
simple-laravel-app]$ tree -L 1
.
├── app
├── artisan
├── bootstrap
├── composer.json
├── composer.lock
├── config
├── database
├── package.json
├── phpunit.xml
├── public
├── readme.md
├── resources
├── routes
├── server.php
├── storage
├── tests
├── vendor
└── webpack.mix.js

10 directories, 8 files
simple-laravel-app]$

simple-laravel-app]$ composer install
…
simple-laravel-app]$ cp .env.example .env
simple-laravel-app]$ php artisan key:generate
Application key [base64:Q9sIu6DleLrn1Qzu9s7Jgaw8ZgULPYTDhwRKUIJH6fk=] set successfully.
simple-laravel-app]$

simple-laravel-app]$ pwd
/laratest/simple-laravel-app
simple-laravel-app]$
 ```
> Make a note on 'pwd' command output. This is the base directory which will be used later.

- Configure test base directory selinux settings 
```sh
~]# laraBaseDir=/laratest/simple-laravel-app
~]# echo $laraBaseDir
/laratest/simple-laravel-app
~]#
~]# semanage fcontext -a -t httpd_sys_content_t "$laraBaseDir(/.*)?"
~]# restorecon -R "$laraBaseDir"
~]# semanage fcontext -a -t httpd_sys_rw_content_t "$laraBaseDir/storage(/.*)?"
~]# restorecon -R "$laraBaseDir/storage"
~]# semanage fcontext -a -t httpd_sys_rw_content_t "$laraBaseDir/bootstrap/cache(/.*)?"
~]# restorecon -R "$laraBaseDir/bootstrap/cache"
~]#
```

- Configure /etc/httpd/conf/httpd.conf and start / enable / restart httpd service
```sh
~]# cp /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.orig
~]# vi /etc/httpd/conf/httpd.conf
~]# diff /etc/httpd/conf/httpd.conf /etc/httpd/conf/httpd.conf.orig
119,120c119
< # DocumentRoot "/var/www/html"
< DocumentRoot "/laratest/simple-laravel-app/public"
---
> DocumentRoot "/var/www/html"
132,133c131
< # <Directory "/var/www/html">
< <Directory "/laratest/simple-laravel-app/public">
---
> <Directory "/var/www/html">
~]#
~]# httpd -t
Syntax OK
~]# systemctl start httpd
~]# systemctl enable httpd
...
~]#
```

- Configure files / directories access permissions
```sh
~]# chown -R apache:apache /laratest/simple-laravel-app
~]# chmod -R 755 /laratest/simple-laravel-app/storage
~]#
```

- Enter http://[developing-host-ip-address]/index.php in web browser url bar to test
- Double check by running Laravel artisan commands (the result may differ from individual source code versions)
```sh
simple-laravel-app]$ php artisan route:list
+--------+----------+----------+------+---------+--------------+
| Domain | Method   | URI      | Name | Action  | Middleware   |
+--------+----------+----------+------+---------+--------------+
|        | GET|HEAD | /        |      | Closure | web          |
|        | GET|HEAD | api/user |      | Closure | api,auth:api |
+--------+----------+----------+------+---------+--------------+
simple-laravel-app]$ 
```

- Explore Laravel logs and / or httpd logs when necessary
```sh
simple-laravel-app]$ ls -l storage/logs/
total 4
-rwxr-xr-x. 1 apache apache 4060 Oct  2 13:12 laravel.log
simple-laravel-app]$
...
[root@lara201 ~]# ls -la /var/log/httpd/
total 12
drwx------. 2 root root   41 Oct  1 23:59 .
drwxr-xr-x. 9 root root 4096 Oct  2 13:05 ..
-rw-r--r--. 1 root root  438 Oct  2 13:21 access_log
-rw-r--r--. 1 root root 1635 Oct  2 13:20 error_log
[root@lara201 ~]#
```

## Make further developing / updating, raise pull / merge request and start new updating cycle
Developer can do further developing / updating or raise pull / merge request based on testing result, and then start new developing cycle.

# License
MIT
