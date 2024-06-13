```shell
vi /etc/named.conf

zone "example.com" IN {
    type master;
    file "/home/ljx/example.com.zone"; // 指定域名的区域文件路径
    allow-update { none; };
};
#随意文件夹
sudo vi /etc/named/zones/example.com.zone

$TTL 86400
@       IN      SOA     ns1.example.com. admin.example.com. (
                        2024061301      ; Serial
                        3600            ; Refresh
                        1800            ; Retry
                        604800          ; Expire
                        86400           ; Minimum TTL
                        )

; 定义域名服务器
@       IN      NS      ns1.example.com.

; 定义域名解析记录
ns1     IN      A       192.168.1.100    ; 替换为您的服务器IP地址
www     IN      A       192.168.1.101    ; 替换为您要解析的域名对应的IP地址

systemctl restart named

