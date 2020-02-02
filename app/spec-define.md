AppSpec 用于定义 Application 的所有依赖包，资源配置，定制化的Script Executor,... 

他的原始数据格式基于 [TOML](https://github.com/toml-lang/toml)，如下为一个基于 OpenResty 的 7 层负载均衡 App 的 Spec 定义:
``` toml
kind = "AppSpec"
roles = [101, 100]

[meta]
  id = "sysinner-httplb"
  name = "Http Load Balancer "
  version = "21"

[[packages]]
  name = "openresty"
  version = "1.15.8.2"
  release = "1"
  dist = "el7"
  arch = "x64"

[[packages]]
  name = "sysinner-httplb"
  version = "0.9.1"
  release = "2"
  dist = "el7"
  arch = "x64"

[[executors]]
  name = "main"
  exec_start = """if pidof sysinner-httplb; then
    exit 0
fi

mkdir -p /opt/openresty/openresty/
rsync -av {{.inpack_prefix_openresty}}/* /opt/openresty/openresty/

mkdir -p /opt/sysinner/httplb/
rsync -av {{.inpack_prefix_sysinner_httplb}}/* /opt/sysinner/httplb/

/opt/sysinner/httplb/bin/sysinner-httplb -log_dir=/opt/sysinner/httplb/log -minloglevel=1 -logtolevels=true > /dev/null 2>&1 &
"""
  exec_stop = "killall sysinner-httplb"
  priority = 8
  [executors.plan]
    on_tick = 60

[[service_ports]]
  name = "http"
  box_port = 8080
  host_port = 80

[exp_res]
  cpu_min = 1
  mem_min = 128
  vol_min = 1

[exp_deploy]
  rep_min = 1
  rep_max = 1
  sys_state = 1
  network_mode = 1
```

但通常，开发者使用 inPanel 可视化的开发 AppSpec, 入口 inPanel/App/Spec, 如下

![AppSpec Define](assets/app-spec-entry-w800.png)

