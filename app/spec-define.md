AppSpec 用于定义 Application 的所有依赖包，资源配置，定制化的Script Executor,... 

他的原始数据为 JSON ，如下为一个基于 OpenResty 的 7 层负载均衡 App 的 Spec 定义:
``` json
{
  "kind": "AppSpec",
  "meta": {
    "id": "sysinner-httplb",
    "name": "Http Load Balancer ",
    "version": "18"
  },
  "roles": [
    100,
    101
  ],
  "packages": [
    {
      "name": "openresty",
      "version": "1.13.6.2",
      "release": "4",
      "dist": "el7",
      "arch": "x64"
    },
    {
      "name": "sysinner-httplb",
      "version": "0.9.0",
      "release": "5",
      "dist": "el7",
      "arch": "x64"
    }
  ],
  "executors": [
    {
      "name": "main",
      "exec_start": "if pidof sysinner-httplb; then\n    exit 0\nfi\n\nmkdir -p /opt/openresty/openresty/\nrsync -av {{.inpack_prefix_openresty}}/* /opt/openresty/openresty/\n\nmkdir -p /opt/sysinner/httplb/\nrsync -av {{.inpack_prefix_sysinner_httplb}}/* /opt/sysinner/httplb/\n\n/opt/sysinner/httplb/bin/sysinner-httplb -log_dir=/opt/sysinner/httplb/log -minloglevel=1 -logtolevels=true \u003e /dev/null 2\u003e\u00261 \u0026\n",
      "exec_stop": "killall sysinner-httplb",
      "priority": 8,
      "plan": {
        "on_tick": 60
      }
    }
  ],
  "service_ports": [
    {
      "name": "http",
      "box_port": 8080,
      "host_port": 80
    },
    {
      "name": "https",
      "box_port": 8443,
      "host_port": 443
    }
  ],
  "exp_res": {
    "cpu_min": 1,
    "mem_min": 128,
    "vol_min": 1
  },
  "exp_deploy": {
    "rep_min": 1,
    "rep_max": 1,
    "sys_state": 2
  }
}
```

但通常，开发者使用 inPanel 可视化的开发 AppSpec, 入口 inPanel/App/Spec, 如下

![AppSpec Define](assets/app-spec-entry-w800.png)

