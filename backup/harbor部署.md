wget https://github.com/goharbor/harbor/releases/download/v2.11.0/harbor-offline-installer-v2.11.0.tgz
解压压缩包
复制compose文件
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/fcc2629c-8bd9-4889-97b3-c4c5cf1e88a0)
#修改配置文件中的端口和hostname
vim harbor.yml
👍 
# 生成私钥
openssl genrsa -out 172.16.8.3.key 2048
# 生成CSR（证书签署请求）
openssl req -new -key 172.16.8.3.key -out 172.16.8.3.csr
# 用私钥生成自签名证书
openssl x509 -req -days 365 -in 172.16.8.3.csr -signkey 172.16.8.3.key -out 172.16.8.3.crt

💯 #新版修复
openssl x509 -req -days 365 -in 172.16.8.3.csr -CA 172.16.8.3.crt -CAkey 172.16.8.3.key -set_serial 01 -out 172.16.8.3.crt -extfile <(echo -e "subjectAltName = IP:172.16.8.3")
👍 
#修改配置文件
https:
  enabled: true
  port: 443
  certificate: /home/test/harbor/harbor/172.16.8.3.crt
  private_key: /home/test/harbor/harbor/172.16.8.3.key
👍 
#执行脚本
./prepre
👍 
#执行安装脚本
install
#配置登录模块
sudo tee /etc/docker/daemon.json <<-'EOF'
> {
> "insecure-registries":["172.16.8.3"]
> }
> EOF
