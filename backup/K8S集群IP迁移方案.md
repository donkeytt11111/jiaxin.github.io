![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/df4eac8d-85f4-4e83-bc69-c4b2f34ef6e0)
👍 先修改master3
👍 :%s/172.16.8.70/172.16.102.70/g
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/44e9920b-3a94-45bf-bc96-d3d7ccc9ff9f)
👍 新增
master2 master1 同理
👍 master1注意network下面的IP要改回之前的参照下图(因为全局替换了)
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/ca265ed9-9a8b-498d-9066-ae19f96467de)
👍 将 /root/.tallos 和kube 改为新节点master1的IP
👍 kubectl edit configmap  cluster-config -n kube-system 将IP改为新地址IP
👍 talosctl reboot -n ms1,ms2,ms3(IP)
👍 重启完成后，将网关修改新的网关，旧IP删除
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/814dcc35-22fd-429f-bd60-ac0b7417fe18)
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/817a5bf1-9c84-434c-8d4e-caa9e35205ce)
