## 远程访问 Pod 实例


## 登录管理面板

登录 inPanel 后，在 Pod 管理列表页中选择登录的 Pod, 进入该 Pod 详情页:

![pic](ops/assets/ops-install-pod-entry.cmp.png)


Pod 详情页目录中点击 "Remote Access" 录入 SSH 公钥，等待容器自动重启后，系统绑定端口信息 (如下图 Service Port Mapping):


![pic](ops/assets/ops-install-pod-ssh-entry.cmp.png)

通过本地终端登录远程 Pod 中便可登录远程 Pod 实例:


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

Tips: InnerStack 容器环境默认全局用户ID为 action, 不具有 root 权限，如果需要重启容器操作，可以通过如下命令达成 killall inagent

