```shell
yum install -y epel-release && yum update -y && yum install -y jq

#!/bin/bash

# 定义常量
BACKUP_FILE="/etc/yum.repos.d/CentOS-Base.repo.backup"
REPO_FILE="/etc/yum.repos.d/CentOS-Base.repo"

# 备份当前的yum源配置文件
echo "备份当前的yum源配置文件..."
cp $REPO_FILE $BACKUP_FILE

# 清空 CentOS-Base.repo 文件
echo "清空 CentOS-Base.repo 文件..."
> $REPO_FILE

# 写入新的配置内容，使用多个稳定的镜像地址
echo "写入新的配置内容..."
cat << EOF > $REPO_FILE
#
# CentOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for CentOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the 
# remarked out baseurl= line instead.
#
#

[base]
name=CentOS-$releasever - Base
# 使用多个稳定的镜像地址
baseurl=http://mirrors.cloud.tencent.com/centos/\$releasever/os/\$basearch/
baseurl=http://mirrors.163.com/centos/\$releasever/os/\$basearch/
baseurl=http://mirrors.huaweicloud.com/centos/\$releasever/os/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

# released updates 
[updates]
name=CentOS-$releasever - Updates
# 使用多个稳定的镜像地址
baseurl=http://mirrors.cloud.tencent.com/centos/\$releasever/updates/\$basearch/
baseurl=http://mirrors.163.com/centos/\$releasever/updates/\$basearch/
baseurl=http://mirrors.huaweicloud.com/centos/\$releasever/updates/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-CentOS-7

# additional packages that may be useful
[extras]
name=CentOS-$releasever - Extras
# 使用多个稳定的镜像地址
baseurl=http://mirrors.cloud.tencent.com/centos/\$releasever/extras/\$basearch/
baseurl=http://mirrors.163.com/centos/\$releasever/extras/\$basearch/
baseurl=http://mirrors.huaweicloud.com/centos/\$releasever/extras/\$basearch/
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Centos-7

# additional packages that extend functionality of existing packages
[centosplus]
name=CentOS-$releasever - Plus
# 使用多个稳定的镜像地址
baseurl=http://mirrors.cloud.tencent.com/centos/\$releasever/centosplus/\$basearch/
baseurl=http://mirrors.163.com/centos/\$releasever/centosplus/\$basearch/
baseurl=http://mirrors.huaweicloud.com/centos/\$releasever/centosplus/\$basearch/
gpgcheck=1
enabled=0
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-Centos-7
EOF

# 清除缓存并重新生成缓存
echo "清除缓存并重新生成缓存..."
yum clean all && yum makecache

# 输出完成信息
echo "配置文件更新完成。"