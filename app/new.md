#### Create new Application Instance

AppSpec 是 InnerStack 的核心，它将inPack包,Pod,定制脚本,服务端口,配置依赖这些松散的东西合并成一个具有特征，包含逻辑的 AppSpec, 再将它部署到集群中，成为最终的 App Instance.

下面以具体的实例演示如何创建一个应用, [gitea](https://gitea.io/) 是一个go语言开发的git项目托管系统，类似github.com, 可以安装在企业内部用于代码协作,下面以 AppSpec/gitea 为例演示如何安装.

先分析需求:
#### 数据库依赖
gitea 依赖数据库，这里我们选取 MySQL, 而 MySQL 本身也可以是一个 AppSpec, 为了复用 MySQL 的依赖场景，我们单独将其定义为 AppSpec/sysinner-mysql-x1. 

InnerStack 内置配置服务，所以 sysinner-mysql-x1 包含两个 inPack Package:
* mysql57 是 mysql-server 原生的编译二进制包
* sysinner-mysql 是为自动化配置 mysql57 的工具

#### gitea 自身软件
gitea 以 golang 语言开发，可以被编译为一个没有第三方库依赖的独立二进制包，另外它依赖配置文件 custem/conf/app.ini, 我们制作了一个配置模版文件放在指定目录下，通过容器内置的 /home/action/.sysinner/inagent confrender 工具完成配置数据能正确，自动的同步到 app.ini 文件.

最终定义生成 AppSpec/gitea-x1 ，系统支持AppSpec引用依赖，所以再引入 AppSpec/sysinner-mysql-x1, 现在可以开始部署了

#### 开始: 在 AppSpec 中选择 gitea-x1 并 Lanuch

![app-new](assets/app-spec-list-w800.png)


#### Step 1: 为应用设置一个名字 
![app-new](assets/app-new-gitea-n1.png)


#### Step 2: 选择将这个 App 绑定到哪个 Pod 上运行

* 不符合硬件规格的 Pod 将不会出现在这个列表里面
* 后期会增加功能，在这个地方可以选择已有的 Pod, 也可以新建 Pod

![app-new](assets/app-new-gitea-n2.png)


#### Step 3: 确认信息
![app-new](assets/app-new-gitea-n3.png)


#### Step 4: 配置 MySQL

因为引入了依赖 AppSpec/sysinner-mysql-x1 ，所以需要完成对MySQL的配置(address,dbname,user,password), 内置的配置服务信息可以自动同步到容器中，所以用户并不需要关注具体的数值，让系统自动去处理，点击 "Next" 继续下面的操作.

![app-new](assets/app-new-gitea-n4.png)

#### Step 5: 配置 gitea
为gitea设置一个名字。其它大部分 gitea 的配置都基于默认设置，不需要关注，当然如果需要更多配置，你可以继续在 AppSpec 中配置，或者修改 gitea/app.ini 模版. 在这个实例中，我们只设置了必要的数据库链接信息和gitea名字.

![app-new](assets/app-new-gitea-n5.png)

#### Step 6: 提交创建 App Instance
成功后，会自动调转到 Pod 详情，可以在这个页面里观察 AppSpec 在这个 Pod 里面的执行情况, 等待一段时间，没有异常的话，Pod 详情页会出现更多有关这个 AppSpec 的信息，如图:
![app-new](assets/app-new-gitea-n7.png)


#### gitea web access
到此，对于 InnerStack 的操作便完成了，对于 gitea 本身而言，它以 web 方式提供用户UI，有自己独立的用户注册管理系统，第一个注册的用户会成为管理员。

gitea 在 Pod 里面提供 web 服务的端口是 3000, 通过部署以后，对外的端口映射为 192.168.3.92:45002 (在 Pod 详情里面查看), 进入这个地址:

#### gitea 1: 应用成功部署后的首页
![app-new](assets/gitea-n1.png)

#### gitea 2: 注册新用户
![app-new](assets/gitea-n2.png)


#### gitea 3: 登录 gitea
![app-new](assets/gitea-n3.png)


#### gitea 4: 在用户页，点击创建新的代码库
![app-new](assets/gitea-n4.png)


#### gitea 5: 配置
![app-new](assets/gitea-n5.png)


#### gitea 6: 完成一个代码库创建
![app-new](assets/gitea-n6.png)

> 当前实例的 AppSpec 和 inPack Packages 均可以在官方网站查阅，地址: [App Center](https://www.sysinner.com/si/app/), [Pack Center](https://www.sysinner.com/si/inpack/)
