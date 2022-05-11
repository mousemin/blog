---
title: prometheus配置nginx监控
date: 2021-09-29T11:09:56+08:00
Description:
Tags: 
    - promethenus
    - nginx
Categories:
    - 工具
DisableComments: false
---

由于没有能力自己写,去[github](https://github.com)找到两个方案
- [nginx-module-vts](https://github.com/vozlt/nginx-module-vts)
- [nginx-lua-prometheus](https://github.com/knyar/nginx-lua-prometheus)

## nginx-module-vts方案

- [nginx-1.16.1](http://nginx.org/download/nginx-1.16.1.tar.gz)
- [nginx-module-vts-0.1.18](https://github.com/vozlt/nginx-module-vts/archive/refs/tags/v0.1.18.tar.gz)

### 安装

编译`nginx`将`nginx-module-vts`模块编译进去
```bash
./configure --add-module=nginx-module-vts && make && make install
```

### 配置
`nginx.conf`添加 `vhost_traffic_status_zone` 示例如下
```conf
http {
    vhost_traffic_status_zone;
    ...
    server {
        ...
        /metrics {
            vhost_traffic_status_display;
            vhost_traffic_status_display_format prometheus;
        }
    }
}
```
>更多使用方式, 请参考[帮助文档](https://github.com/vozlt/nginx-module-vts/blob/master/README.md)

访问 `http://xxxxx:xxx/metrics` 示例

![prometheus-nginx](https://cdn.mousemin.com/img/2021-09-11-7e7f26acfd71363d6fea1605cab61b9a.png)



## nginx-lua-prometheus方案

由于使用的是 `nginx-1.16` , [lua-nginx-module](https://github.com/openresty/lua-nginx-module)没有明确支持 `nginx-1.16` ,暂未使用