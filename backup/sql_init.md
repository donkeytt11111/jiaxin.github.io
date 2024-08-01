psql -h 172.16.102.147 -p 32002 -U root

Passw0rd@MY

show timezone;

set time zone 'Asia/Shanghai';

SELECT * FROM pg_timezone_names WHERE name = current_setting('TIMEZONE');


