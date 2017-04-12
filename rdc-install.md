# Rdc-zentao-ha 文档

#### 环境要求 :

Docker ,Docker-compose

#### 1 . 主节点上：安装nfs网络文件系统

本机系统：centos为例

    安装
    yum -y install nfs-utils

    # 设置启动项
    systemctl enable nfs-server

    # 启动服务
    systemctl start nfs-server

    # 创建gitlab的文件目录
    mkdir /mnt/nfs/rdc -p

    # 写入配置文件，`/mnt/nfs/rdc`为上面创建的目录，`172.29.0.0/16`为要连接到这台文件服务器的客户端的ip地址，这里使用的是掩码的方式。代表255.255
    echo '/mnt/nfs/rdc   172.29.0.0/16(rw,sync,no_root_squash,no_all_squash)'  >> /etc/exports

    # 重启服务使配置生效
    systemctl restart nfs-server

#### 2 . 配置nfs客户端

注：在所有节点都需要安装客户端

    # 创建nfs-client的目录，也是rdc容器的卷要挂载的目录
    mkdir /mnt/docker/rdc -p

    # 写入配置文件，其中`172.29.1.54:/mnt/nfs/rdc`为nfs服务器端ip地址和rdc的文件共享目录，`/mnt/docker/rdc`为上面创建的目录，即rdc的容器要挂载的目录
    echo '172.29.1.54:/mnt/nfs/rdc  /mnt/docker/rdc      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0'  >> /etc/fstab

    # 挂载目录，使配置生效
    mount -a

    # 查看挂载目录
    df -h

#### 3 . 构建node1主节点

```
#构建 node1 节点，节点包括以下镜像：
(1) . mysql
(2) . zentao
(3) . nginx
```

下载地址：[docker-compose.yml](./node1/docker-compose.yml)

构建步骤如下：

```

```



