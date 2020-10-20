## 简介

### 什么是 InnerStack PaaS Engine
受 Google Borg 论文和 Kubernetes 系统的启发, InnerStack 提供轻量，易用，自动化的方式帮助开发者和运营人员部署和管理容器化应用服务.

### Google Borg/Kubernetes 中 Pod 的定义
Pod 是 Google Borg/Kubernetes 的核心概念，Pod 是实现一组特定应用功能的容器组合(可以简单的理解为Pod就是容器), 集群大部分操作都以Pod为对象展开，一个Pod实例包括多个功能:

1. 定义基础运行环境，比如 OS类型, 底层依赖库，系统工具集.
2. 定义容器资源配额和隔离(CPU,Memory,Network IO, Volume IO)，使物理服务器的资源使用高效.
3. 作为资源调度的最小单位，实现副本扩展，宕机恢复，负载均衡等高级功能.
4. 定义 Container Image，所有应用都被打包进Image中，包括基础的运行环境, 中间件, 数据库，业务应用等. 在 Kubernetes 系统中，大部分应用软件的变更、升级通过Pod Image同步实现.

### InnerStack 核心概念 (AppSpec + inPack + Pod)
InnerStack 沿用了部分 Kubernetes-Pod 概念(上面所列的前3项)；另方面，为了让软件包管理更有效，应用管理更灵活，InnerStack 有独特的设计, 包括3个核心组件: AppSpec + inPack + Pod

#### 1. InnerStack AppSpec
AppSpec 是一个配置规范，大致相当于加强版的 Kubernetes-Pod-Manifest, 它是 InnerStack 逻辑核心，为一个线上应用配置其所有依赖的文件和资源，以及定制业务逻辑

1. 定义资源规格，不同应用对 CPU, Memory，Network, Volume 有不同的配额设置. 当然，这个配额也对接计费核算系统.
2. 定义依赖包及其版本 ([inPack](../../view/inpack/)), 如一个 [WordPress CMS](http://wordpress.org/) 依赖 mysql, php, nginx. 
3. 定义导出的对外服务端口，如 http/80, mysql/3306
4. 定义配置依赖, 内置配置管理工具，配置自动分发到Pod，Pod通过配置信息取得自身的配置参数，以及发现其它依赖服务的地址参数
5. 定义容器内部的定制脚本 (Pod/Executor), InnerStack 容器的启动入口为系统自带的 inagent　守护进程，由 inagent 按照配置完成内部服务的启动，计划执行.

> 更详细的 AppSpec 配置规范可以参考 [AppSpec Define](../../view/si/app/spec-define.md), 或者参考官方的 App Center 实例 [App Center](https://www.sysinner.cn/si/app/)

#### 2. InnerStack Pod

* 定义基础运行环境,如 OS类型, 底层依赖库，系统工具集.
* 定义容器资源隔离,如 CPU,Memory,Network IO, Volume IO.
* 作为资源调度对象，实现副本扩展，宕机恢复，负载均衡等等高级功能.

#### 3. InnerStack inPack (Package Manager)

InnerStack 创建了 [inPack](../../view/inpack/) 项目, 用来打包，分发，同步各类运行环境，中间件，应用软件

* 有关 inPack 操作详细，请参考 [inPack文档](../../view/inpack/)
* 可以访问官方网站的 [Pack Center](https://www.sysinner.cn/si/inpack) 查阅当前已经打包的软件项目, 所有软件包的源代码可以访问 [https://github.com/inpack/](https://github.com/inpack) 获取
* inPack Server 索引并存储集群所有包，支持 Channel 归类 (beta, release, ...)


