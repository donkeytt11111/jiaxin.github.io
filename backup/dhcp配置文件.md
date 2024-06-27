```shell
ddns-update-style none;
subnet 172.16.8.0 netmask 255.255.255.0
{
range 172.16.8.201 172.16.8.240;
option routers 172.16.8.1;
default-lease-time 600;
max-lease-time 7200;
#filename"ipxe.efi";
#next-server 172.16.8.2;
}
