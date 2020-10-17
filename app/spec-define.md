AppSpec 用于定义 Application 的所有依赖包，资源配置，定制化的Script Executor,... 

他的原始数据格式基于 [TOML](https://github.com/toml-lang/toml)，如下为一个基于 Gitea 源码管理应用的 AppSpec 定义:

``` toml
kind = "AppSpec"
roles = [101, 100]
type_tags = ["devops", "enterprise"]

[meta]
  id = "gitea-x1"
  name = "Gitea 源码自托管系统"
  version = "1.0"

[[depends]]
  id = "sysinner-mysql-x1"
  name = "MySQL x1"
  version = "1.0"

[[packages]]
  name = "gitea"
  version = "1.12"

[[executors]]
  name = "gitea-main"
  exec_start = """
DAEMON=/opt/gitea/gitea/gitea
DAEMON_ARGS="web"
NAME=gitea

if pidof $NAME; then
    exit 0
fi

if [ ! -d "/opt/gitea/gitea" ]; then
  mkdir -p /opt/gitea/gitea
fi
rsync -av {{.inpack_prefix_gitea}}/* /opt/gitea/gitea/

/home/action/.sysinner/inagent config-render --app-spec gitea-x1 --in /opt/gitea/gitea/misc/app.ini --out /opt/gitea/gitea/custom/conf/app.ini
/home/action/.sysinner/inagent config-merge --app-spec gitea-x1 --config /opt/gitea/gitea/custom/conf/app.ini --with-config-field cfg/gitea/app_ini

cd /opt/gitea/gitea/
$DAEMON $DAEMON_ARGS >> /home/action/var/log/gitea.log 2>&1 &
"""
  exec_stop = "killall gitea"

  [executors.plan]
    on_tick = 60

[[service_ports]]
  name = "http"
  box_port = 3000

[[service_ports]]
  name = "gitssh"
  box_port = 3022

[configurator]
  name = "cfg/gitea"

  [[configurator.fields]]
    name = "app_name"
    title = "应用名称"
    type = 1
    default = "Gitea"

  [[configurator.fields]]
    name = "app_ini"
    title = "ini 增量配置"
    type = 304
    default = "[i18n]\nLANGS = en-US,zh-CN\nNAMES = English,中文"

[exp_res]
  cpu_min = 2
  mem_min = 128
  vol_min = 5

[exp_deploy]
  rep_min = 1
  rep_max = 1
  sys_state = 1
  network_mode = 1
```

但通常，开发者使用 inPanel 可视化的开发 AppSpec, 入口 inPanel/App/Spec, 如下

![AppSpec Define](app/assets/app-spec-edit-v.cmp.png)

