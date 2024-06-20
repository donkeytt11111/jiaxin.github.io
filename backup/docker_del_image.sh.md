#!/bin/bash

keyword=$1
version=$2

if [ -z "$keyword" ] || [ -z "$version" ]; then
    echo "Usage: $0 <keyword> <version>"
    exit 1
fi

# 列出所有符合关键词的镜像
images=$(docker image ls --format '{{.Repository}}:{{.Tag}}' | grep "$keyword")

# 循环删除符合条件的镜像
for image in $images
do
    repo=$(echo "$image" | cut -d ':' -f 1)
    tag=$(echo "$image" | cut -d ':' -f 2)

    # 检查镜像版本是否包含指定的版本号
    if [[ "$tag" == *"$version"* ]]; then
        docker image rm "$repo:$tag"
    fi
done

![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/27f074f3-68c8-42a6-9eec-9af23f3f6e77)
