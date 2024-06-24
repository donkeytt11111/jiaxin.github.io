```shell
#!/bin/bash

# 定义目标仓库前缀
TARGET_IP="172.16.8.3"

# 获取所有本地镜像的信息
IMAGES=$(docker images --format "{{.Repository}}:{{.Tag}}")

# 遍历所有本地镜像
for IMAGE in $IMAGES
do
    # 获取镜像的原始仓库和标签
    REPO=$(echo $IMAGE | cut -d':' -f1)
    TAG=$(echo $IMAGE | cut -d':' -f2)

    # 构造新的仓库地址
    NEW_REPO="$TARGET_IP/$REPO"

    # 构造新的完整标签
    NEW_IMAGE="$NEW_REPO:$TAG"

    echo "Tagging image $IMAGE as $NEW_IMAGE"
    docker tag $IMAGE $NEW_IMAGE

    if [ $? -eq 0 ]; then
        echo "Successfully tagged $IMAGE as $NEW_IMAGE"

        echo "Pushing image $NEW_IMAGE to remote repository..."
        docker push $NEW_IMAGE

        if [ $? -eq 0 ]; then
            echo "Successfully pushed $NEW_IMAGE"
        else
            echo "Failed to push $NEW_IMAGE"
        fi
    else
        echo "Failed to tag $IMAGE"
    fi
done
```
![image](https://github.com/donkeytt11111/jiaxin.github.io/assets/167744103/cb4089e5-c5b0-4687-999d-d58a8c97050b)
