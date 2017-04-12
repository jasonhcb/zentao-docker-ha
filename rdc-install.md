# Rdc-zentao-ha 文档

#### 环境要求 :

Docker ,Docker-compose

### 1 . 主节点上：安装nfs网络文件系统

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

### 2 . 配置nfs客户端

注：所有节点都需要安装客户端

    # 创建nfs-client的目录，也是rdc容器的卷要挂载的目录
    mkdir /mnt/docker/rdc -p

    # 写入配置文件，其中`172.29.1.54:/mnt/nfs/rdc`为nfs服务器端ip地址和rdc的文件共享目录，`/mnt/docker/rdc`为上面创建的目录，即rdc的容器要挂载的目录
    echo '172.29.1.54:/mnt/nfs/rdc  /mnt/docker/rdc      nfs auto,nofail,noatime,nolock,intr,tcp,actimeo=1800 0 0'  >> /etc/fstab

    # 挂载目录，使配置生效
    mount -a

    # 查看挂载目录
    df -h

### 3 . 构建HA节点

#### 3.1 构建node1主节点

```
#构建 node1 节点，节点包括以下镜像：
(1) . mysql
(2) . zentao
(3) . nginx
```

下载地址：[docker-compose.yml](./node1/docker-compose.yml)

构建步骤如下：

```
#进入下载目录运行docker-compose

docker-compose up -d
```

#### 3.2 构建node2节点

```
#构建 node2 节点，节点包括以下镜像：
(1) . zentao
```

下载地址：[docker-compose.yml](./node2/docker-compose.yml)

构建步骤如下：

```
#进入下载目录运行docker-compose

docker-compose up -d
```

### 4 . 将备份数据资料导入到对应nfs

从 迁移 服务器 拷贝文件 到 nfs 文件夹

| 服务器文件夹 | nfs文件夹 |
| :--- | :--- |
| zentaopms/www/data | /mnt/docker/rdc/www/data |
| zentaopms/tmp/backup | /mnt/docker/rdc/tmp/backup |
| zentaopms/module/hdc/data | /mnt/docker/rdc/module/hdc/data/ |
| zentaopms/module/bulletin/data | /mnt/docker/rdc/module/bulletin/data |
| zentaopms/config | /mnt/docker/rdc/config |

### 5 . node1节点导入zentao.sql

创建一个名为：zentao  的数据库 , 将事先 导出 的zentao.sql 数据库 导入到数据库中。

注意：编码为 UTF8 - utf8-general-ci

### 6 .node1节点配置Nginx转发

docker-compose 中已经 将 nginx目录映射出来到了 /mnt/docker/rdc/nginx下

将  [default.conf](./node1/default.conf) 配置好 放入 /mnt/docker/rdc/nginx 下

```
upstream rdc {
    ip_hash;
    server 192.168.59.103:80;   #修改为node1 的节点ip
    server 192.168.59.102:80;   #修改为node2 的节点ip
}

server {
    listen       80;
    server_name  rdc.hand-china.com; # 修改为主机 hostname 的名称，比如:rdc.hand-china.com
```

注：重启 此 容器

### 7 .配置zentao连接数据库

打开 /mnt/docker/rdc/config/my.php

```php
# 修改 host,port,name,user,password

$config->db->host        = 'mysql';   //修改成节点mysql  的ip地址
$config->db->port        = '3306';    //修改成节点mysql  的ip端口
$config->db->name        = 'zentao';  
$config->db->user        = 'root';    //修改成节点mysql  的用户名，默认root
$config->db->password    = 'my-secret-pw'; //修改成节点mysql  的用户名，默认my-secret-pw
```



