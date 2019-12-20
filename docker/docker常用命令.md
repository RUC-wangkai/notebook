删除容器：
    docker container rm  trusting_newton
清理所有处于终止状态的容器：
    docker container prune
删除镜像：
    docker rmi image-id
批量删除none镜像：
    docker rmi $(docker images | grep "none" | awk '{print $3}') 