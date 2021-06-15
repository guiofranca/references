# Install MariaDB on Ubuntu 18.04

First update the repositories to include MariaDB's repo:

```
sudo apt install software-properties-common
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mirror.ufscar.br/mariadb/repo/10.5/ubuntu bionic main'
sudo apt update
```

Then install it:

```
sudo apt install mariadb-server
```

Secure the instalation according to your needs. The default root password is empty.
```
sudo mysql_secure_installation
```

You're good to go!

# Creating Users and Schemas

To create a new schema, login to your mysql (MariaDB) and run the following codes:

```
mysql -u root -p
create database mydatabase character set utf8mb4 collate utf8mb4_unicode_ci;
create user 'myuser'@'localhost' identified by 'somepassword';
grant all privileges on mydatabase.* to 'myuser'@'localhost';
flush privileges;
exit
```

Aditionally, if you want access the database from anywhere, do the following:
```
create user 'myuser'@'%' identified by 'somepassword';
grant all privileges on mydatabase.* to 'myuser'@'%';
flush privileges;
exit
```

And, if you need to access the database from outside network, config the server on `/etc/mysql/mariadb.conf.d/50-server.cnf` and uncomment or add the line:
```
bind-address = 0.0.0.0
```

Finally, restart the server:
```
sudo service mariadb restart
```
