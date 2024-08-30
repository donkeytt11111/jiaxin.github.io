psql -h 172.16.102.145 -p 32002 -U root

Passw0rd@MY

show timezone;

SELECT * FROM pg_timezone_names WHERE name = current_setting('TIMEZONE');

CREATE DATABASE netbox;
CREATE USER netbox WITH PASSWORD 'Passw0rd@MY';

GRANT ALL PRIVILEGES ON DATABASE netbox TO netbox;

SELECT * FROM pg_timezone_names WHERE name = current_setting('TIMEZONE');

create database kea;

create user kea with password  'Passw0rd@MY';

GRANT ALL PRIVILEGES ON DATABASE kea TO kea;

ALTER DATABASE netbox SET timezone TO 'Asia/Shanghai';

ALTER DATABASE kea SET timezone TO 'Asia/Shanghai';

ALTER DATABASE root SET timezone TO 'Asia/Shanghai';

`````````````````````````````````````````````````````````````````````````````````````````````````

psql -h 172.16.102.145 -p 32002 -U  netbox


psql -h 172.16.102.145 -p 32002  -d kea -U kea -f ./dhcpdb_create.pgsql

ALTER DATABASE kea SET timezone TO 'Asia/Shanghai';

kubectl rollout restart statefulset -n cncp-system






