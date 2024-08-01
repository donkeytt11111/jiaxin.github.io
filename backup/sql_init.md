psql -h 172.16.102.147 -p 32002 -U root

Passw0rd@MY

show timezone;

set time zone 'Asia/Shanghai';

SELECT * FROM pg_timezone_names WHERE name = current_setting('TIMEZONE');

root=# CREATE DATABASE netbox;
CREATE DATABASE
root=# CREATE USER netbox WITH PASSWORD 'Passw0rd@MY';
CREATE ROLE
root=# GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;
GRANT
root=#
检查时区
root=# show timezone;
root=# SELECT * FROM pg_timezone_names WHERE name = current_setting('TIMEZONE');
root=# set time zone 'Asia/Shanghai';
创建用户（并给它一个密码），授予访问权限
root=#create database kea;
root=# create user kea with password  'Passw0rd@MY';
root=# GRANT ALL PRIVILEGES ON DATABASE kea TO kea;
退出
root=# \q

psql -h 172.16.102.118 -p 32002  -d kea -U kea -f dhcpdb_create.pgsql




