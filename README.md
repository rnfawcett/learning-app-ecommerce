# Introduction

This is a sample e-commerce application built for learning purposes.

Here's how to deploy it on CentOS systems:

## Deploy Pre-Requisites

1. Install FirewallD

```
sudo yum install -y firewalld
sudo service firewalld start
sudo systemctl enable firewalld
```

* Check

```
sudo firewalld status
- {check for "active (running)"}
firewall-cmd --list-all
- {look for "ssh" on the "services:" output line
```

## Deploy and Configure Database

2. Install MariaDB

```
sudo yum install -y mariadb-server
sudo vi /etc/my.cnf
- {only if any change required}
sudo service mariadb start
sudo systemctl enable mariadb
```

* Check

```
sudo service mariadb status
- {check for "active (running)"}
```

3. Configure firewall for Database

```
sudo firewall-cmd --permanent --zone=public --add-port=3306/tcp
sudo firewall-cmd --reload
```

* Check

```
sudo firewall-cmd --list-all
- {check for "3306/tcp" on the "ports:" line}
```

4. Configure Database

```
$ mysql
MariaDB > CREATE DATABASE ecomdb;
MariaDB > CREATE USER 'ecomuser'@'localhost' IDENTIFIED BY 'ecompassword';
MariaDB > GRANT ALL PRIVILEGES ON *.* TO 'ecomuser'@'localhost';
MariaDB > FLUSH PRIVILEGES;
```

> ON a multi-node setup remember to provide the IP address of the web server here: `'ecomuser'@'web-server-ip'`

* Check

```
$ mysql
MariaDB > show database;
- {look for "ecomdb"}
```

5. Load Product Inventory Information to database

Create the db-load-script.sql

```
cat > db-load-script.sql <<-EOF
USE ecomdb;
CREATE TABLE products (id mediumint(8) unsigned NOT NULL auto_increment,Name varchar(255) default NULL,Price varchar(255) default NULL, ImageUrl varchar(255) default NULL,PRIMARY KEY (id)) AUTO_INCREMENT=1;

INSERT INTO products (Name,Price,ImageUrl) VALUES ("Laptop","100","c-1.png"),("Drone","200","c-2.png"),("VR","300","c-3.png"),("Tablet","50","c-5.png"),("Watch","90","c-6.png"),("Phone Covers","20","c-7.png"),("Phone","80","c-8.png"),("Laptop","150","c-4.png");

EOF
```

Run sql script

```

mysql < db-load-script.sql
```

* Check

```
$ mysql
show databases:
use ecomdb;
select * from products;
- {look for products' data}
```


## Deploy and Configure Web

1. Install required packages

```
sudo yum install -y httpd php php-mysql
sudo firewall-cmd --permanent --zone=public --add-port=80/tcp
sudo firewall-cmd --reload
```

* Check

```
sudo firewall-cmd --list-all
- {look for "80/tcp" on the "ports:" line}
```

2. Configure httpd

Change `DirectoryIndex index.html` to `DirectoryIndex index.php` to make the php page the default page

```
sudo sed -i 's/index.html/index.php/g' /etc/httpd/conf/httpd.conf
```

3. Start httpd

```
sudo service httpd start
sudo systemctl enable httpd
```

* Check

```
sudo service httpd status
- {check for "active (running)"}
```

4. Download code

```
sudo yum install -y git
- {only when "git --version" doesn't reply with some version #}
git clone https://github.com/kodekloudhub/learning-app-ecommerce.git /var/www/html/
```

5. Update index.php

Update [index.php](https://github.com/kodekloudhub/learning-app-ecommerce/blob/13b6e9ddc867eff30368c7e4f013164a85e2dccb/index.php#L107) file to connect to the right database server. In this case `localhost` since the database is on the same server.

```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php

              <?php
                        $link = mysqli_connect('172.20.1.101', 'ecomuser', 'ecompassword', 'ecomdb');
                        if ($link) {
                        $res = mysqli_query($link, "select * from products;");
                        while ($row = mysqli_fetch_assoc($res)) { ?>
```

> ON a multi-node setup remember to provide the IP address of the database server here.
```
sudo sed -i 's/172.20.1.101/localhost/g' /var/www/html/index.php
```

6. Test

```
curl http://localhost
```

* Check

```
- {look for "Product List" in the curl output}
```
