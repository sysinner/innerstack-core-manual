### App Detail

在 inPanel/Applications 列表中，管理已经创建的 App Instances, 如:

* 查看当前App实例的 AppSpec 配置 (AppSpec 具有版本属性)
* 如果 AppSpec 有更新，可以执行 Deploy 操作，升级当前的 App Instance
* 执行 Setup 可以对当前的 App Instance 进行 Start, Stop, Destory 操作
* 因为 App Instance 绑定在 Pod 环境中运行，所以对 App 监控统计请到对应的 Pod 实例详情里查看
* TODO 后期计划开发一个通用, 可灵活配置，自动化的监控框架，来实现对 App 内部业务特征的监控

![app-entry](app/assets/app-list.cmp.png)

