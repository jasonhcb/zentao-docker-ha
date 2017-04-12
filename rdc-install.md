# Rdc-zentao-ha 文档

#### 环境要求 :

Docker ,Docker-compose

#### 1 . 安装nfs网络文件系统\(centos\)

    安装
    yum -y install nfs-utils

    # 设置启动项
    systemctl enable nfs-server

    # 启动服务
    systemctl start nfs-server

    # 创建gitlab的文件目录
    mkdir /mnt/nfs/gitlab -p

    # 写入配置文件，`/mnt/nfs/gitlab`为上面创建的目录，`172.29.0.0/16`为要连接到这台文件服务器的客户端的ip地址，这里使用的是掩码的方式。代表255.255
    echo '/mnt/nfs/gitlab   172.29.0.0/16(rw,sync,no_root_squash,no_all_squash)'  >> /etc/exports

    # 重启服务使配置生效
    systemctl restart nfs-server



#### 2 . 构建node1主节点

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





