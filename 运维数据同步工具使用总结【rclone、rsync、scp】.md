>  原文地址 [blog.csdn.net](https://blog.csdn.net/AttaGain/article/details/149826930)

#### 数据同步工具使用总结【rclone、rsync、scp】

- [一、数据处理背景](#一数据处理背景)
- [二、数据处理方法对比](#二数据处理方法对比)
  - [1、数据关系梳理](#1数据关系梳理)
  - [2、不同工具处理方法](#2不同工具处理方法)
  - [3、经验总结](#3经验总结)
- [三、工具扩展知识](#三工具扩展知识)
  - [1、rclone 工具介绍](#1rclone-工具介绍)
    - [（1）、rclone 概述](#1rclone-概述)
    - [（2）、安装工具及配置](#2安装工具及配置)
  - [（4）、应用场景](#4应用场景)
    - [从一个集群服务迁移到另一个集群服务](#从一个集群服务迁移到另一个集群服务)
      - [本地文件迁移到云上服务](#本地文件迁移到云上服务)
      - [本地文件同步到云上服务](#本地文件同步到云上服务)
  - [2、rsync 工具介绍](#2rsync-工具介绍)
    - [（1）、rsync（Remote Sync，远程同步）](#1rsyncremote-sync远程同步)
    - [（2）rsync 同步源](#2rsync-同步源)
    - [（3）、配置 rsync 源](#3配置-rsync-源)
      - [配置源的两种表达方式](#配置源的两种表达方式)
      - [免交互格式](#免交互格式)
  - [3、scp 工具介绍](#3scp-工具介绍)

一、数据处理背景
--------

*   作为数据工作者，日常工作中常见工作有：数据同步，网络数据抓取，系统数据入库，日志数据搜集，还有数据分析、挖掘等等。数据数据同步工作，本地数据文件的复制，本地服务器间数据同步，本地服务器到网络存储（网盘、NAS、对象存储等）的数据同步，网络存储之间的数据同步，大数据非结构性数据同步等。
*   本文主要是希望将十几个 T 的数据，从 A 服务器同步到 B 服务器，数据源是由 MinIO 实现的对象存储服务提供（部署在 C 服务器），mount 挂载在 A 服务器制定路径（由于服务器访问安全问题，访问策略上如此定义）。

二、数据处理方法对比
----------

### 1、数据关系梳理

*   数据源由 MinIO 服务提供，挂载在 A 服务器制定路径。在数据同步过程我们无需关注 A 服务器和 C 服务器之间的访问关系。在数据同步效率上可能带来的瓶颈由：a、A 服务器和 C 服务器之间的网络带宽；b、C 服务器提供 MinIO 服务的并发能力。
*   A 服务器和 B 服务器可通过 SSH 访问，但无法访问 C 服务器。  
    B 服务器 ssh 访问 A 服务器，端口默认 22  
    用户名：root 密码：1qaz@wsx  
    A 服务器 IP:192.168.1.100; 数据路径：/data/bigdata  
    B 服务器数据路径：/data/bigdata; 数据路径：/data/bigdata

### 2、不同工具处理方法

*   scp 复制
    *   简单直接，比较适合小文件传输，没有复杂的配置。
    *   该命令和本地 cp 命令参数大致相同，区别在于跨服务器复制。
    *   命令格式：sudo scp [属性参数] 源数据路径 目标数据路径
    *   如果源数据路径为远程服务器，则意味着数据从远程服务器复制到本地；如果源数据路径为本地，则意味着数据由本地复制到远程服务器。
    *   登录 B 服务器，执行如下脚本：

```
sudo scp -r root@192.168.1.100:/data/bigdata /data/
```

上述命令实现数据从 A 服务器复制到 B 服务器。该命令在执行大文件复制时，可能因为网络环境因素导致数据复制终端，不支持断点续传。

*   rsync 同步
    *   支持断点续传，同步执行失败，可以再次执行，检查数据状态，继续完成前次未同步数据
    *   Linux 系统包自带工具，需要单独安装，命令参数相对复杂，日常简单的数据同步工作基本满足需求
    *   数据传输效率较 scp 高些，结合 inotify-tools 可实现数据实时同步; parallel 可以实现多线程并行处理
    *   命令格式：sudo rsync [属性] 源数据路径 目标数据路径
    *   如果源数据路径为远程服务器，则意味着数据从远程服务器复制到本地；如果源数据路径为本地，则意味着数据由本地复制到远程服务器。
    *   登录 B 服务器，执行如下脚本：

```
sudo rsync -av root@192.168.1.100:/data/bigdata /data/bigdata
```

上述命令实现数据从 A 服务器到 B 服务器数据同步。适合比较大的文件同步，支持断点续传。需要注意的是，源数据路径和目标数据路径均需指定明确存在的路径，然后将路径下所有数据资源同步到目标路径下，不会创建根路径名称。

*   rclone 同步
    *   支持断点续传，可实现服务器间的数据同步，复制工作，功能非常强大
    *   需要自主安装的第三方工具包。支持本地数据同步功能，具有与 unix 命令 rsync、cp、mv、mount、ls、ncdu、tree、rm 和 cat 相当的强大云功能
    *   支持大文件的快速传输，默认支持多线程，数据同步效率大约是 rsync 的三倍
    *   命令格式：sudo rclone 源数据路径 目标数据路径 [属性]
    *   如果源数据路径为远程服务器，则意味着数据从远程服务器复制到本地；如果源数据路径为本地，则意味着数据由本地复制到远程服务器。
    *   登录 B 服务器，首先需要配置 rclone 配置文件：

```
sudo rcllone config
```

*   按照向导提示，配置远程服务器的参数信息。证书部分如果均为空，则按照用户名，密码方式认证，生成参数结果如下：  
    配置参数也可以复制如下信息，仅仅修改标识符，服务器 B 的 IP，user, port, 其他保持不变。

```
[svra]                      # 标识符，很重要，可以随意起
type = sftp
host = 192.168.1.100        # 改为服务器B的ip
user = root                 # 默认root用户
port = 22                   # 默认22端口，如果是其他端口请修改
# key_file = ~/.ssh/rclone-merged  #证书认证，可以删除该行，仅仅用户名密码认证方式
shell_type = unix
md5sum_command = md5sum     # 向导生成时可以选择为空
sha1sum_command = sha1sum   # 向导生成时可以选择为空
```

*   执行如下命令，实现数据同步

```
sudo rclone sync svra:/data/bigdata /data/bigdata -u -v -P --transfers=20 --ignore-errors --buffer-size=128M  --drive-acknowledge-abuse
```

上述命令实现数据从 A 服务器到 B 服务器数据同步。适合比较大的文件同步，支持断点续传。需要注意的是，源数据路径和目标数据路径均需指定明确存在的路径，然后将路径下所有数据资源同步到目标路径下，不会创建根路径名称。

### 3、经验总结

*   scp 小白救星，但大文件是噩梦。优势​​：命令简单、加密传输、系统预装；致命缺陷​​：​​断点续传 = 0​​！传输 10GB 文件若中断，必须重头再来；​​适用场景​​：单文件＜1GB、临时备份、内网低风险环境
*   rsync：企业级神器，增量同步碾压全场。核心理由​​：仅传输差异部分，​​节省带宽 70%+​；–partial：断点续传；bwlimit=5000：限速 5MB/s 避免挤爆业务；​​实测数据​​：同步 100GB 变化文件，​​scp 需 1.5 小时 → rsync 仅 18 分钟
*   rclone：强大的数据同步工具，云时代数据搬运神奇。支持几乎所有云存储，网盘，本地服务器之间的数据同步。重点需要维护好配置文件中各个存储资源信息

三、工具扩展知识
--------

### 1、rclone 工具介绍

#### （1）、rclone 概述

Rclone 是一款的命令行工具，支持在不同对象存储、网盘间同步、上传、下载数据。  
官网网址：https://rclone.org/  
Github 项目：https://github.com/ncw/rclone  
最近有一个不幸的消息是：Amazon 禁止了 rclone 在他家存储上使用，好忧伤。  
新闻地址：https://forum.rclone.org/t/rclone-has-been-banned-from-amazon-drive/2314  
新闻地址：https://www.lowendtalk.com/discussion/115117/rclone-banned-from-amazon-drive  
支持的主流对象存储有：  
　　　　Google Drive  
　　　　Amazon S3  
　　　　Openstack Swift / Rackspace cloud files / Memset Memstore  
　　　　Dropbox  
　　　　Google Cloud Storage  
　　　　Amazon Drive  
　　　　Microsoft One Drive  
　　　　Hubic  
　　　　Backblaze B2  
　　　　Yandex Disk

The local filesystem

#### （2）、安装工具及配置

*   工具安装  
    以 Ubuntu 系统为例，执行在线安装命令，亦可下载安装包，离线安装。

```
sudo -v ; curl https://rclone.org/install.sh | sudo bash
```

*   资源配置  
    执行下面命令，按照向导选择对应资源配置类型，生成配置文件

```
sudo rclone config
Current remotes:
n) New remote
s) Set configuration password
q) Quit config
n/s/q> q
```

*   生成配置文件类型举例：
    
    *   服务器资源
    
    ```
    [svra]                      # 标识符，很重要，可以随意起
    type = sftp
    host = 192.168.1.100        # 改为服务器B的ip
    user = root                 # 默认root用户
    port = 22                   # 默认22端口，如果是其他端口请修改
    # key_file = ~/.ssh/rclone-merged  #证书认证，可以删除该行，仅仅用户名密码认证方式
    shell_type = unix
    md5sum_command = md5sum     # 向导生成时可以选择为空
    sha1sum_command = sha1sum   # 向导生成时可以选择为空
    ```
    
    *   S3 协议类型
    
    ```
    # cat /root/.config/rclone/rclone.conf 
     [src] #s数据源服务
     type = s3
     provider = Minio
     env_auth = false
     access_key_id = xxxxxx
     secret_access_key = xxxxxx
     region = cn-east-1
     endpoint = http://10.0.176.163:22829
     location_constraint = 
     server_side_encryption = 
      
     [des] #目标数据服务服务
     type = s3
     provider = Minio
     env_auth = false
     access_key_id = xxxxxx
     secret_access_key = xxxxxx
     region = cn-east-1
     endpoint = http://10.0.176.163:20445
     location_constraint = 
     server_side_encryption =
    ```
    
*   服务期间免密登录设置（按需）
    
    *   生成密钥对  
        在客户端上执行命令，生成密钥对。然后，合并秘钥，最终会在~/.ssh / 目录下生成 rclone.pub，rclone，以及 rclone-merged 三个文件  
        注意：ssh 生成的秘钥文件，保存在当前用户目录下的. ssh 目录下，即：~/.ssh/
    
    ```
    ssh-keygen -q -t rsa -b 4096 -C "rclone key" -N "" -f ~/.ssh/rclone     #静默生成rclone密钥对
    cd ~/.ssh/
    cat rclone* > rclone-merged   # 将密钥对合并，否则会连接失败
    ```
    
    *   复制公钥到远程访问服务器
        *   命令方式复制  
            假设服务器的 ip 是 192.168.1.100，ssh 端口是 22，使用以下命令，然后输入服务器的密码即可。
    
    ```
    ssh-copy-id -i ~/.ssh/rclone.pub -f -p 22 root@192.168.1.100   #自行修改为你自己的
    ```
    
    *   手动方式复制  
        在客户端上打开 rclone.pub，复制里面的内容，在服务器上~/.ssh / 的目录下，新建 authorized_keys 文件, 粘贴内容到 authorized_keys
*   挂载远程服务到本地路径
    
    *   使用 rclone mount 命令，将远程服务挂载到本地
    
    ```
    sudo rclone mount svra:/data/bigdata    /data/bigdata --allow-other  --allow-non-empty --umask 0002
    ll /data/bigdata
    ```
    
*   后台运行 rclone
    
    ```
    nohup sudo rclone sync svra:/data/bigdata /data/bigdata -u -v -P --transfers=20 --ignore-errors --buffer-size=128M  --drive-acknowledge-abuse  sync.log 2 > &1 &
    ```
    
    其他例子：
    
    ```
    #复制最近7天的消息 --max-age 7d ,支持d、h、m、s，将minio:test的最近7天数据复制到minio1:test，并将日志写入copyrclone.log
    #不带--max-age 7d则是复制所有数据
    nohup rclone -P copy --max-age 7d --no-traverse minio:test minio1:test > copyrclone.log 2>&1 &
    
    #保留最近30天的数据，30天前的全部删除 --min-age 30d ,支持d、h、m、s，删除minio:test 30天前的数据，并将日志写入derclone.log
    #不带--mix-age 30d则是删除所有数据
    #相反--max-age 30d则是删除最近30天的数据
    nohup rclone -P delete --min-age 30d --no-traverse minio:test > derclone.log 2>&1 &
    ```
    

```
### （3）、命令介绍
- Rclone将一个目录树从一个存储系统同步到另一个。

  它的语法是这样的
  
  语法：[选项] 子命令 <参数> <参数…>
- 常见子命令
  查看一个远端目录：
  
  rclone ls remote:path
  
  拷贝一个本地目录到远端的目录：
  
  rclone copy /local/path remote:path
  
  将本地目录同步到远端的目录：
  
  rclone sync --interactive /local/path remote:path

  rclone最常用的命令如下，完整的命令列表可以通过help或者是rclone提供的文档查看：
  ```bash
  rclone config - Enter an interactive configuration session.
   rclone copy - Copy files from source to dest, skipping already copied.
   rclone sync - Make source and dest identical, modifying destination only.
   rclone bisync - Bidirectional synchronization between two paths.
   rclone move - Move files from source to dest.
   rclone delete - Remove the contents of path.
   rclone purge - Remove the path and all of its contents.
   rclone mkdir - Make the path if it doesn't already exist.
   rclone rmdir - Remove the path.
   rclone rmdirs - Remove any empty directories under the path.
   rclone check - Check if the files in the source and destination match.
   rclone ls - List all the objects in the path with size and path.
   rclone lsd - List all directories/containers/buckets in the path.
   rclone lsl - List all the objects in the path with size, modification time and path.
   rclone md5sum - Produce an md5sum file for all the objects in the path.
   rclone sha1sum - Produce a sha1sum file for all the objects in the path.
   rclone size - Return the total size and number of objects in remote:path.
   rclone version - Show the version number.
   rclone cleanup - Clean up the remote if possible.
   rclone dedupe - Interactively find duplicate files and delete/rename them.
   rclone authorize - Remote authorization.
   rclone cat - Concatenate any files and send them to stdout.
   rclone copyto - Copy files from source to dest, skipping already copied.
   rclone genautocomplete - Output shell completion scripts for rclone.
   rclone gendocs - Output markdown docs for rclone to the directory supplied.
   rclone listremotes - List all the remotes in the config file.
   rclone mount - Mount the remote as a mountpoint.
   rclone moveto - Move file or directory from source to dest.
   rclone obscure - Obscure password for use in the rclone.conf
   rclone cryptcheck - Check the integrity of an encrypted remote.
   rclone about - Get quota information from the remote.
  ```

>官网 https://rclone.org/

### （4）、应用场景
#### 从一个集群服务迁移到另一个集群服务
从一个集群将对应的文件迁移到另一个集群的应用是比较常见的一种应用情况，下面以对象服务的迁移为例进行简单的介绍：

首先我们需要对两个服务都进行配置，第一个集群是test1，第二个集群是test2，现在需要将test1集群中的桶bucket1迁移到test2中。

可以使用如下的命令：
```bash
rclone copy test://bucket1 test2://bucket1
```

如果在 test2 集群上没有这个桶 bucket1 的话，我们可以选择先手动在 test2 上创建对应的桶 bucket1，也可以选择添加参数 --create-empty-src-dirs 来自动创建：

```
rclone --create-empty-src-dirs copy test1://bucket1 test2://bucket2
```

如果想要实时监控进度的话，可以使用参数—progress；

##### 本地文件迁移到云上服务

本地存在文件需要上云的情况下，也可以使用 rclone 来完成这个任务：

```
rclone copy $localpath $remote://$bucket/$prefix
```

可以用这个参数结构执行命令来完成任务，也可以根据实际情况添加相应的参数。

##### 本地文件同步到云上服务

本地文件在拷贝过一次后，或者是首次直接迁移到云端，也可以使用 sync 命令来完成：

```
rclone sync $src_path  $dest_path
```

eg: rclone sync /mnt/test test://bucket1/test  
sync 与 copy 最大的区别在于 sync 命令是以 “同步” 为目的，也就是说，如果存在一个文件，dest 端有而 src 端没有的话，会将 dest 端的文件清理掉。

### 2、rsync 工具介绍

#### （1）、rsync（Remote Sync，远程同步）

一款开源的快速备份工具  
支持本地复制  
也可以在不同主机（例如：其他 SSH、rsync 主机）之间镜像同步整个目录树，支持增量备份，并保持钳接和权限。  
采用优化的同步算法，传输前执行压缩,，因此非常适用于异地备份、镜像服务器等应用。

#### （2）rsync 同步源

在远程同步任务中，负责发起 rsync 司步操作的客户机称为发起端，而负责响应来自客户机的 rsync 同步操作的服务器称为同步源 (备份源)。在同步过程中，同步源负责提供文件的原始位置，发起端应对该位置具有读取权限。  
例：  
A 服务器同步 B 服务器的数据，B 服务器就是备份源  
反过来，B 服务器同步 A 服务器的数据，那么 A 服务器就是备份源  
![](https://i-blog.csdnimg.cn/direct/c4a618f0fe7b4d83baa52ace40f3028c.png)

#### （3）、配置 rsync 源

*   基本思路
    
    *   建立 rsyncd.conf 配置文件、独立的 rsync 账号文件
        *   配置文件 rsyncd.conf  
            需手动配置，语法类似于 Samba 配置  
            认证配置 auth users、secrets file，不加则为匿名
        *   rsync 账号文件  
            采用 “用户名：密码” 的格式记录，每行一个用户记录  
            独立的账号数据，不依赖系统账号
    *   启用 rsync 服务  
        通过 --daemon 独自提供服务：rsync --daemon  
        可以通过执行 kill $(cat /var/run/rsyncd.pid) 关闭服务
*   rsync 命令
    

```
#命令的用法
rsync [选项] 原始位置 目标位置

#----------常用选项--------------------------
-r：递归模式，包含目录及子目录中的所有文件。
-l：对于符号链接文件仍然复制为符号链接文件。
-v：显示同步过程的详细（verbose）信息。
-z：在传输文件时进行压缩（compress）。
-a：归档模式，保留文件的权限、属性等信息，等同于组合选项“-rlptgoD”。
-p：保留文件的权限标记。
-t：保留文件的时间标记。
-g：保留文件的属组标记（仅超级用户使用）。
-o：保留文件的属主标记（仅超级用户使用）。
-H：保留硬连接文件。
-A：保留 ACL 属性信息。
-D：保留设备文件及其他特殊文件。
--delete：删除目标位置有而原始位置没有的文件,即删除差异文件，保留一致性。
--checksum：根据校验和（而不是文件大小、修改时间）来决定是否跳过文件。
--password-file=file：从file中得到密码，用于免交互处理，file文件的权限要是600
```

##### 配置源的两种表达方式

将指定的资源下载到本地 / root 目录下进行备份。  
格式一：  
用户名 @主机地址:: 共享模块名  
例如：  
backuper@192.168.163.10::wwwroot /opt

```
格式二：
rsync://用户名@主机地址/共享模块名
例如：
rsync://backuper@192.168.163.10/wwwroot /opt
```

##### 免交互格式

```
echo "密码" > /etc/密码文件
chmod 600 /etc/密码文件

#设置周期性任务
crontab -e
30 22 * * * /usr/bin/rsync -az --delete --password-file=/etc/密码文件 backuper@192.168.163.10::wwwroot /opt

systemctl restart crond
systemctl enable crond
-----------------------------------
rsync进程数量 rsync支持多线程
https://blog.51cto.com/u_16213679/10351277
```

### 3、scp 工具介绍

系统内核命令，基本能力不再赘述。