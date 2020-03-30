# Introduction

This is a sample e-commerce application built for learning purposes.

Here's how to deploy it on CentOS systems:

## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum install firewalld
sudo service firewalld start
sudo systemctl enable firewalld # enable it so it starts when the systems starts
```
check if it is up and running
```
systemctl status firewalld
```
check the settings for the firewallD
```
firewalld --list-all
```

## Deploy and Configure Database

1. Install MariaDB (community forked version of mysql, everything is almost the same apaprt from the name)

```
sudo yum install mariadb-server
sudo vi /etc/my.cnf # no change, we leave the default settings
sudo service mariadb start
sudo systemctl enable mariadb
```
check if it is up and running
```
systemctl status mariadb
```
2. Configure firewall for Database (default mysql/mariadb port: 3306)

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload  # without reload the changes do not take effect
```
check if the port is set correctly
```
firewall-cmd --list-all
```

3. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > show databases;    # to check if it is created
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
MariaDB > exit
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

4. Load Product Inventory Information to database

```
mysql < db-load-script.sql
```


## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysql
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo vi /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo service httpd start
sudo systemctl enable httpd
```

4. Download code

```
sudo yum install -y git
git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

5. Update index.php

Update [index.php](https://github.com/kodekloudhub/learning-app-ecommerce/blob/13b6e9ddc867eff30368c7e4f013164a85e2dccb/index.php#L107) file to connect to the right database server. In this case `localhost` since the database is on the same server.

```
              <?php
                        $link = mysqli_connect('172.20.1.101', 'ecomuser', 'ecompassword', 'ecomdb');
                        if ($link) {
                        $res = mysqli_query($link, "select * from products;");
                        while ($row = mysqli_fetch_assoc($res)) { ?>
```

> ON a multi-node setup remember to provide the IP address of the database server here.

6. Test

```
curl http://localhost
```
