ubuntu16.04:
参考https://yeasy.gitbooks.io/docker_practice/install/ubuntu.html

1.卸载旧版本
    sudo apt-get remove docker \
                docker-engine \
                docker.io
2.添加使用 HTTPS 传输的软件包以及 CA 证书
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    software-properties-common

3.添加软件源的 GPG 密钥(官网访问不到，使用国内源)
curl -fsSL https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu/gpg | sudo apt-key add -

4.向 source.list 中添加 Docker 软件源
sudo add-apt-repository \
    "deb [arch=amd64] https://mirrors.ustc.edu.cn/docker-ce/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

5.安装Docker CE
sudo apt-get update
sudo apt-get install docker-ce

6.测试是否安装成功
sudo docker run hello-world

建立docker用户组
1.建立 docker 组：
sudo groupadd docker

2.将当前用户加入 docker 组：
sudo usermod -a6 docker $USER
退出当前终端并重新登录，进行如下测试。
sudo systemctl restart docker

如果提示get ......dial unix /var/run/docker.sock权限不够，则修改/var/run/docker.sock权限
sudo chmod a+rw /var/run/docker.sock

