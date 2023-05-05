[TOC]

### Log

Grafana + Loki + Promtail 采用物理安装方式 (yum安装有标准的systemctl 管理方式与配置文件)

#### Grafana

~~~bash
$ yum install -y https://dl.grafana.com/oss/release/grafana-9.5.1-1.aarch64.rpm
~~~

配置文件 /etc/grafana/grafana.ini

#### Loki + Promtail

~~~bash
https://github.com/grafana/loki/releases

$ wget https://github.com/grafana/loki/releases/download/v2.8.2/loki-2.8.2.aarch64.rpm
$ wget https://github.com/grafana/loki/releases/download/v2.8.2/promtail-2.8.2.aarch64.rpm
~~~

promtail配置 /etc/promtail/config

~~~yaml
server:
  http_listen_port: 9080
  grpc_listen_port: 0

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://localhost:3100/loki/api/v1/push
# 修改或添加job_name即可
scrape_configs:
- job_name: all-apps
  static_configs:
  - targets:
      - localhost
    labels:
      job: apps
      __path__: /var/log/apps/*log
~~~

