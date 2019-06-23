### PodSpec 方案配置

在 Pod 创建过程中，需要为其分配合理的硬件资源，硬件资源因为升级换待，以及应用对资源的使用特征，会产生差异，PodSpecPlan 就是用于归类这类需求。比如通性型,CPU密集型,内存密集型,IO密集型等，对PodSpecPlan的管理由运维人员根据经验和自身业务特征定义，系统没有任何限制.

![ops-pod](assets/ops-pod-specplan-list.png)

如下图所示, 正在配置一个名为 General g1 的方案,　将不同性能和特征的服务器分组 (Zone/Cluster), PodSpecPlan 会绑定到指定的集群分组，内置的资源调度器 Scheduler 将按照　PodSpecPlan　的配置将 Pod 调度到指定的集群. 

![ops-pod](assets/ops-pod-specplan-entry.png)

