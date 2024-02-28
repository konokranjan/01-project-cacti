# 01-project-cacti

1
dnf install -y epel-release

2
dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

3
yum install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm

4
yum install -y net-snmp net-snmp-utils net-snmp-libs rrdtool

5
yum install -y mariadb-server mariadb

6
yum install -y php php-xml php-session php-sockets php-ldap php-gd php-json php-mysqlnd php-gmp php-mbstring php-posix php-snmp php-intl

7
systemctl start httpd
systemctl start snmpd
systemctl start mariadb

8
systemctl enable httpd
systemctl enable snmpd
systemctl enable mariadb

9
vi /etc/my.cnf.d/mariadb-server.cnf

10 Add variables in the [mysqld] section.
collation-server=utf8mb4_unicode_ci
character-set-server=utf8mb4
max_heap_table_size=32M
tmp_table_size=32M
join_buffer_size=64M
# 25% Of Total System Memory
innodb_buffer_pool_size=1GB
# pool_size/128 for less than 1GB of memory
innodb_buffer_pool_instances=10
innodb_flush_log_at_timeout=3
innodb_read_io_threads=32
innodb_write_io_threads=16
innodb_io_capacity=5000
innodb_io_capacity_max=10000

11
systemctl restart mariadb

12
mysql -u root -p
create database cacti;
GRANT ALL ON cacti.* TO cactiuser@localhost IDENTIFIED BY 'admin';
flush privileges;
exit

13
mysql -u root -p mysql < /usr/share/mariadb/mysql_test_data_timezone.sql
user password : cactipassword

14
mysql -u root -p
GRANT SELECT ON mysql.time_zone_name TO cactiuser@localhost;
flush privileges;
exit

15
yum install -y cacti

16
mysql cacti < /usr/share/doc/cacti/cacti.sql -u cactiuser -p
user password : cactipassword

17
vi /usr/share/cacti/include/config.php
/*
* Make sure these values reflect your actual database/host/user/password
*/
$database_type = 'mysql';
$database_default = 'cacti';
$database_hostname = 'localhost';
$database_username = 'cactiuser';
$database_password = 'cactipassword';
$database_port = '3306';

18
vi /etc/cron.d/cacti

19 Uncomment the below line.
*/5 * * * *     apache  /usr/bin/php /usr/share/cacti/poller.php > /dev/null 2>&1

20
vi /etc/httpd/conf.d/cacti.conf

-----------------------
Alias /cacti    /usr/share/cacti
<Directory /usr/share/cacti/>
<IfModule mod_authz_core.c>
# httpd 2.4
Require all granted
</IfModule>
<IfModule !mod_authz_core.c>
# httpd 2.2
Order deny,allow
Deny from all
Allow from localhost
</IfModule>
</Directory>
----------------------
21
vi /etc/php.ini
date.timezone = Asia/Dhaka
memory_limit = 512M
max_execution_time = 60

22
systemctl restart httpd
systemctl restart php-fpm

23
firewall-cmd --permanent --add-service=http
firewall-cmd --reload

24
setenforce 0

login your server ip/cacti
