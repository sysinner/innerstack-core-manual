### Create new Pod Instance

Pod是容器集合. 大部分PaaS系统以技术栈封装各类容器时环境(如 Java, Python, PHP), 在这基础上再集成各类应用软件，最终的交付实体是包括全部环境和依赖包的容器镜像; 而在 Sysinner 中容器仅限于系统级的运行环境，其它所有的运行时环境，数据库，应用程序都统一由 [inPack](../../../view/inpack/) 打包管理, 所以 Sysinner Pod 类似传统意义上的操作系统，一个接近零损耗的OS环境.

#### Workflow Essentials

* 对于**开发者**: 首先创建一个新的 Pod, 其需要的 Java, Python, MySQL, Redis, VIM 等等都在后期通过 AppSpec 部署绑定到这个 Pod 的方式完成，**按需添加, 任意组合**. 简单说来，对于开发人员的工作流类似:

> 1. Create Pod
> 2. Deploy AppSpec: Java + MySQL + VIM
> 3. Develop + Build (if needed)

* 对于**运维**，或者普通用户: 流程更为简单，只需要在 AppSpec 中选择指定的应用，按照提示逐步操作即可 (Pod 在 AppSpec 部署的过程中按照提示选择自动创建)

> 详细操作请参考 [Create new App Instance](app/new.md)

#### 创建一个空的 Pod Instance

如图，在 inPanel - Pods 中点击 **Create Pod Instance**

![pic](ops/assets/ops-install-pod-list.cmp.png)

如图, 创建过程需要考虑 Pod 对资源的使用需求，点击提交即可.

![pic](ops/assets/ops-install-pod-new.cmp.png)

创建过程可能遇到的问题:

* 创建失败，提示 Charge Failed: 系统内置计费模块，而该用户账户余额不足
* 创建成功，但长期处于 ZoneMaster/Scheduler/Pending 状态: 集群没有足够的硬件资源，系统处理等待调度中

#### 关于计费模块

企业系统中，对应用运行成本的量化记录很重要，InnerStack 内置了资源使用计费模块 (基于 [hooto IAM](https://github.com/hooto/iam/) 实现).

* 计费模块用于帮助开发和运维人员估计应用运行成本
* 扣费的主体为应用的所有者，管理员在用户管理后台为所属人员添加资源配额.
* 采用按月预扣费冻结的方式(Prepay)，如果期间执行了销毁Pod的操作, 系统会按实际使用时间计费(Payout)，并退回多余冻结金额，详细的数据请在会员中心的 Accout 里面查看

