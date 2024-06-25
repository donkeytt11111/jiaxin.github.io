grep data_volume  /app/harbor/harbor.yml    #根据配置文件查找数据存储目录
data_volume: /data

cd /data/registry     #进入到Harbor的数据目录下

find  docker/   -type  d  -name "current"  | sed  's|docker/registry/v2/repositories/||g;s|/_manifests/tags/|:|g;s|/current||g'  >  images.list

cat images.list
 
curl https://192.168.2.250:443/api/version  -k
{"version":"v2.0"}
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/2390402e-6803-45a1-bf0d-e99f0347395d)



```shell
#!/bin/bash
Harbor_Address=172.16.1.200       #Harbor主机地址
Harbor_User=admin                      #登录Harbor的用户
Harbor_Passwd=Harbor12345              #登录Harbor的用户密码
Images_File=./images.list   # 镜像清单文件
Tar_File=/backup/Harbor-backup/                 #镜像tar包存放路径
set -x
# 获取Harbor中所有的项目（Projects）
Project_List=$(curl -u admin:Harbor12345  -H "Content-Type: application/json" -X GET  https://172.16.1.200/api/v2.0/projects  -k  | python3 -m json.tool | grep name | awk '/"name": /' | awk -F '"' '{print $4}')

for Project in $Project_List;do
   # 循环获取项目下所有的镜像
    Image_Names=$(curl -u admin:Harbor12345 -H "Content-Type: application/json" -X GET https://172.16.1.200/api/v2.0/projects/$Project/repositories -k | python3 -m json.tool | grep name | awk '/"name": /' | awk -F '"' '{print $4}')
    for Image in $Image_Names;do
        # 循环获取镜像的版本（tag)
        Image_Tags=$(curl -u admin:Harbor12345  -H "Content-Type: application/json"   -X GET  https://172.16.1.200/v2/$Image/tags/list  -k |  awk -F '"'  '{print $8,$10,$12}')
       for Tag in $Image_Tags;do
            # 格式化输出镜像信息
            echo "$Harbor_Address/$Image:$Tag"   >> harbor-images-`date '+%Y-%m-%d'`.txt
        done
    done
done
Image_tags=$(uniq $Images_File)
for image_tag in $Image_tags;do
    image_Name=$(echo $image_tag | awk -F/ '{print $3}' |  awk -F: '{print $1}')
    image_Lable=$(echo $image_tag | awk -F/ '{print $3}' |  awk -F: '{print $2}')
    #image_Name=$(echo $image_tag | awk -F ' +' '{print $1}')
    #image_Lable=$(echo $image_tag | awk -F ' +' '{print $2}')
    docker pull 172.16.1.200/$image_tag
    docker tag  172.16.1.200/$image_tag $image_tag
    docker save $image_tag  -o $image_Name.tar
    docker rmi  172.16.1.200/$image_tag
    docker rmi  $image_tag
    mv $image_Name.tar  $Tar_File
done
```
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/cdad1ceb-eb50-42ab-b84e-7e6fe15f9cb1)

👍 
将errorlist文件修改为镜像仓库地址

使用以下脚本完成errorimage拉取

```shell
#!/bin/bash

# 从文件读取镜像列表
while IFS= read -r image; do
    echo "正在拉取镜像: $image"
    docker pull "$image"

    # 正确提取域名后面直到标签前的部分（保留斜杠后的所有内容直到冒号）
    new_repo_name=$(echo "$image" | cut -d/ -f2- | cut -d: -f1)
    new_tag_name=$(echo "$new_repo_name" | awk -F/ '{print $NF}')
    echo "$new_repo_name"
    echo "$new_tag_name"
    version=$(echo "$image" | awk -F: '{print $NF}') # 获取版本号

    # 构建新的标签（包含完整路径和版本号）
    new_tag="${new_repo_name}:${version}"
    
    echo "为镜像打标签: $image -> $new_tag"
    docker tag "$image" "$new_tag"

    echo "导出镜像为tar包: $new_tag"
    docker save "$new_tag" > "/backup/Harbor-backup/${new_tag_name}_${version}.tar"

    # 删除原镜像（根据需要可选择性执行此步骤）
    echo "删除原镜像: $image"
    docker rmi "$image"
    docker rmi "$new_tag"
done < error_images.txt

# 检查是否有任何操作失败
if [ $? -eq 0 ]; then
    echo "所有镜像已成功处理并备份。"
else
    echo "操作过程中出现错误，请检查上面的输出详情。"
fi
```
