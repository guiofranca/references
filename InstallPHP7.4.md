# Install PHP 7.4 on Ubuntu 18.04
Add the needed repositories:

```
sudo apt install software-properties-common
sudo add-apt-repository ppa:ondrej/php
sudo apt update
```

Then install PHP accorgind to your needs. The following snippet should be good to go.
```
sudo apt install php7.4-fpm php7.4-common php7.4-mysql php7.4-mongodb php7.4-mbstring php7.4-zip php7.4-curl php7.4-xml php7.4-json php7.4-gd php7.4-redis php7.4-dev
```

# Install SQL Server Driver for PHP 7.4
First, do this magic:
You may change the source list according to your server version. [Available options](https://docs.microsoft.com/pt-br/sql/connect/odbc/linux-mac/installing-the-microsoft-odbc-driver-for-sql-server?view=sql-server-ver15#ubuntu)
```
sudo su
curl https://packages.microsoft.com/keys/microsoft.asc | apt-key add -
curl https://packages.microsoft.com/config/ubuntu/18.04/prod.list > /etc/apt/sources.list.d/mssql-release.list
exit
sudo apt update
```

Then you'll need to install odbc for MS SQL Server, PHP Pear and other tools to compile the driver.

```
sudo ACCEPT_EULA=Y apt install msodbcsql17
sudo apt install php-pear
sudo apt install unixodbc-dev
sudo pecl install sqlsrv
sudo pecl install pdo_sqlsrv
```

After that, create the config files for sqlsrv and pdo_sqlsrv mods.
```
sudo su
printf "; priority=20\nextension=sqlsrv.so\n" > /etc/php/7.4/mods-available/sqlsrv.ini
printf "; priority=30\nextension=pdo_sqlsrv.so\n" > /etc/php/7.4/mods-available/pdo_sqlsrv.ini
exit
```

Finally, enable them and restart PHP-FPM.
```
sudo phpenmod -v 7.4 sqlsrv pdo_sqlsrv
sudo service php7.4-fpm restart
```

You can test if everything is working fine with the commands. They shoud output no errors.
```
php -v
php -m
php -m | grep sqlsrv
```

# Install Oracle database Driver for PHP 7.4
Guide adapted from https://gist.github.com/hewerthomn/81eea2935051eb2500941a9309bca703

Download and unzip Oracle's instant client. Install zip if you don't have it already installed.
```
#sudo apt install zip
mkdir /opt/oracle
cd /opt/oracle
curl https://download.oracle.com/otn_software/linux/instantclient/199000/instantclient-basic-linux.x64-19.9.0.0.0dbru.zip --output client.zip
curl https://download.oracle.com/otn_software/linux/instantclient/199000/instantclient-sdk-linux.x64-19.9.0.0.0dbru.zip --output sdk.zip
unzip client.zip
unzip sdk.zip
```

Link the instant client files:
```
sudo ln -s /opt/oracle/instantclient_19_9/libclntsh.so.12.1 /opt/oracle/instantclient_19_9/libclntsh.so
sudo ln -s /opt/oracle/instantclient_19_9/libocci.so.12.1 /opt/oracle/instantclient_19_9/libocci.so
```

Add to ldconfig:
```
sudo echo "/opt/oracle/instantclient_19_9" >> /etc/ld.so.conf.d/x86_64-linux-gnu.conf
sudo ldconfig
```

Edit and add to the end of environment file `/etc/environment`:
```
sudo nano /etc/environment
LD_LIBRARY_PATH="/opt/oracle/instantclient_19_9"
ORACLE_HOME="/opt/oracle/instantclient_19_9"
```

Prepare for installation and install. When asked for instantclient location input the following: `instantclient,/opt/oracle/instantclient_19_9`
```
sudo apt install php7.4-dev php7.4-pear build-essential libaio1
sudo pecl install oci8-2.2.0
```

Enable:
```
sudo su
echo "extension=oci8.so" >> /etc/php/7.4/fpm/conf.d/20-oci8.ini
echo "extension=oci8.so" >> /etc/php/7.4/cli/conf.d/20-oci8.ini
exit
```

Check for errors with `php -m` and `php -v`. Those commands should output no errors.
Restart PHP-FPM.
```
sudo service php7.4-fpm restart
```
