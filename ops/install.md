## 安装 InnerStack


### 宿主服务器

InnerStack 是一个分布式 PaaS 平台引擎，支持从 1 ~ N 个宿主节点的部署方式，要求如下:

1. x86_64 通用服务器，物理服务器和共有云服务商的云主机都适用。
2. 当前支持 CentOS-8.x/7.x 和 RHEL-8.x/7.x 操作系统，最小化安装。
3. 数据盘需要 XFS 文件系统(/opt 目录)，挂载参数加入 pquota 支持。


以腾讯云 CVM 标准云主机为例，我们在后台新建一个云主机实例,如下图:

![pic](ops/assets/tc.order-1.cmp.png)

注意上图黄色框内信息:

1. 确保操作系统版本 CentOS 8
2. 独立的数据盘
3. 第一个节点默认为管理节点，需要公网 IP（如果企业私有网络可忽略此项目）

下一步开通主机，SSH 登录

### 系统环境

更新系统到最新版本:

``` shell
yum update -y

# ... ...

cat /etc/redhat-release

# 输出
CentOS Linux release 8.2.2004 (Core)
```

### 数据盘配置

确认数据设备的有效性

``` shell
[root@VM-0-8-centos ~]# fdisk -l
Disk /dev/vda: 50 GiB, 53687091200 bytes, 104857600 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x89ee0607

Device     Boot Start       End   Sectors Size Id Type
/dev/vda1  *     2048 104857566 104855519  50G 83 Linux


Disk /dev/vdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
```

如上 ```Disk /dev/vdb``` 这个设备就是需要配置的数据盘.


分区:

``` shell
# 分区
parted -s /dev/vdb mklabel gpt
parted -s /dev/vdb mkpart primary xfs 0% 100%

# 确认
fdisk -l

# 以下为输出信息，这里隐藏了非 /dev/vdb 设备的信息
Disk /dev/vdb: 100 GiB, 107374182400 bytes, 209715200 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: A7669389-4181-4425-B5B5-91E7792E3C09

Device     Start       End   Sectors  Size Type
/dev/vdb1   2048 209713151 209711104  100G Linux filesystem
```


格式化

``` shell
# 格式化
mkfs.xfs /dev/vdb1

# 以下为输出
meta-data=/dev/vdb1              isize=512    agcount=4, agsize=6553472 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=1, sparse=1, rmapbt=0
         =                       reflink=1
data     =                       bsize=4096   blocks=26213888, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0, ftype=1
log      =internal log           bsize=4096   blocks=12799, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
```

挂载

``` shell
# 参数设置
echo "/dev/vdb1  /opt  xfs  defaults,pquota     1 2" >> /etc/fstab

# 执行
mount -a

# 确认
df -h

# 以下为输出
Filesystem      Size  Used Avail Use% Mounted on
devtmpfs        902M     0  902M   0% /dev
tmpfs           915M   24K  915M   1% /dev/shm
tmpfs           915M  400K  915M   1% /run
tmpfs           915M     0  915M   0% /sys/fs/cgroup
/dev/vda1        50G  1.9G   46G   4% /
tmpfs           183M     0  183M   0% /run/user/0
/dev/vdb1       100G  747M  100G   1% /opt
```

注:

InnerStack 中对 /opt 目录的设置是必要条件，其关键数据的保存路径分别是:

1. /opt/sysinner/innerstack/var/db_zone/ 核心数据库, inpack 包文件存储
2. /opt/sysinner/innerstack/var/log/ 系统运行日志
3. /opt/sysinner/ipm/ 节点解压 inapck 包文件目录
4. /opt/docker/ 容器 OS 镜像文件
5. /opt/sysinner/pods/ 容器内的用户数据


如果宿主服务器有多块数据盘，InnerStack 支持扩展 pods 编排到多个盘符，它有一个固定的挂载模式:

* /data/hdd_([0-9a-f]{8}) 挂载 HDD 类型数据盘
* /data/ssd_([0-9a-f]{8}) 挂载 SSD 类型数据盘


确认数据盘符有效性后，继续下面操作。


### 安装 innerstack


设置 yum 源，将下面文本整体复制并在目标节点执行:

``` shell
cat <<< '[docker-ce-stable]
name=Docker CE Stable - $basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/$releasever/$basearch/stable
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg

[sysinner]
name=SysInner - InnerStack beta
baseurl=https://www.sysinner.com/repo/el/$releasever/x86_64/beta
        https://www.sysinner.cn/repo/el/$releasever/x86_64/beta
enabled=1
gpgcheck=0

' > /etc/yum.repos.d/sysinner.repo
```


安装，启动服务

``` shell
yum install -y sysinner-innerstack && systemctl enable innerstack && systemctl start innerstack
```


没有异常的话，可以通过日志文件查看运行状况

``` shell

tail -f /opt/sysinner/innerstack/var/log/innerstackd.log.INFO

# 输出
Running on machine: VM-0-10-centos
Binary: Built with gc go1.15.2 for linux/amd64
Log line format: [DIWEF] yyyy-mm-dd hh:ii:ss.uuuuuu file:line] msg
I 2020-10-16 21:05:40.809607 hmsg.go:301] hmsg/mail-manager template set 2
I 2020-10-16 21:05:40.809622 hmsg.go:301] hmsg/mail-manager template set 2
W 2020-10-16 21:05:40.810111 main.go:81] waiting initialization
I 2020-10-16 21:05:40.810201 service.go:241] lessgo/httpsrv: listening on unix//opt/sysinner/innerstack/var/server.sock
W 2020-10-16 21:05:43.810237 main.go:81] waiting initialization
W 2020-10-16 21:05:46.810364 main.go:81] waiting initialization
W 2020-10-16 21:05:49.810478 main.go:81] waiting initialization
```

也可以通过本地命令 ```innerstack info``` 查看本地环境参数信息


### 初始配置

如上，系统已经正常启动并等待初始配置，第1个 InnerStack 宿主节点需要执行 zone-init 初始化操作，如下:

``` shell
innerstack zone-init --host-addr 172.21.0.6:9529 --wan-ip 49.233.25.33 --http-port 9530 --zone-id z1 --cell-id g1

zone successfully initialized
  inPanel Management
       url: http://49.233.25.33:9530/in/
      user: sysadmin
  password: fe1c81e9351aa340d8f0110531c9e5aa
```

设置成功后，系统会输出 inPanel 管理入口地址和登录信息，请妥善保存，并在浏览中打开 URL 登录

参数说明

* --host-addr 为该节点内网IP和端口
* --wan-ip 为该节点对应的公网IP，对于企业内网机房这项配置可以忽略
* --http-port 对外提供基于 http 的 API, WebUI 端口
* --zone-id 机房唯一别名
* --cell-id 同一机房内服务器分组唯一别名

你还可以通过 innerstack zone-init --help 获取更多帮助信息.

这个命令只在第一次建立 InnerStack 集群时使用一次，后续安装的 innerstack 节点则通过 innerstack host-join 命令加入宿主节点，详细参数可通过 ```innerstack host-join --help``` 获取帮助信息. 


## 登录管理面板

登录系统后，会看到一个空的 Pod 列表页:

![pic](ops/assets/ops-install-pod-list.cmp.png)


可以创建一个 "空 Pod" 测试功能.

![pic](ops/assets/ops-install-pod-new.cmp.png)


创建成功后，进入 Pod 详情页:

![pic](ops/assets/ops-install-pod-entry.cmp.png)


注: 新装节点，后台任务会自动下载并同步基础 g3/g2 等容器镜像 (一共 2 GB大小)，如果镜像没有拉取完毕，此时的 Pod 会一直等待较长时间才能启动完成，这个时间取决于你的网络状况。

> 如果你有本地 docker register 服务可通过 /etc/docker/daemon.json 编辑修改，重启 docker 加快镜像拉取效率。

通过 Pod 页下方 Replicas 日志观察，一切正常后可以通过 Pod 详情页的 "Remote Access" 录入 SSH 公钥，等待容器自动重启后，系统绑定端口信息 (如下图 Service Port Mapping):

![pic](ops/assets/ops-install-pod-ssh-entry.cmp.png)

通过本地终端登录远程 Pod 中，查看基本信息:

``` shell
[guest@localhost ~]$ ssh action@49.233.25.33 -p 18676

[action@sysinner ~]$ df -h
Filesystem      Size  Used Avail Use% Mounted on
overlay          10G  8.0K   10G   1% /
tmpfs            64M     0   64M   0% /dev
tmpfs           915M     0  915M   0% /sys/fs/cgroup
shm              64M     0   64M   0% /dev/shm
/dev/vdb1        10G   14M   10G   1% /opt
tmpfs           915M     0  915M   0% /proc/acpi
tmpfs           915M     0  915M   0% /proc/scsi
tmpfs           915M     0  915M   0% /sys/firmware

[action@sysinner ~]$ uptime
 08:32:54 up 0 min,  0 users,  load average: 0.00, 0.10, 0.22
 
[action@sysinner ~]$ free
              total        used        free      shared  buff/cache   available
Mem:         524288       11464      510184           0        2640      512824
Swap:             0           0           0
[action@sysinner ~]$ 
```

到此，节点安装、配置完成，你可以继续下面文档了解更多:

* [InnerStack 用户手册](https://www.sysinner.cn/gdoc/view/si/)
* [AppSpec 应用构建实践](https://www.sysinner.cn/gdoc/view/app-guide/)
* [inPack 包管理](https://www.sysinner.cn/gdoc/view/inpack/)


> 注: InnerStack PaaS Engine 当前处于 BETA 阶段, 如有问题欢迎联系交流:
>
> * 邮件 evorui#gmail.com
> * 腾讯微信 @ruilog
> * 新浪微博 @ruilog



