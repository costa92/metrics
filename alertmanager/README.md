# alertmanager 安装

### docker 部署

```sh
docker run -d --restart=always \
--name=alertmanager \
-p 9093:9093 \
-v $PWD/alertmanager.yml:/etc/alertmanager/alertmanager.yml \
-v $PWD/template:/etc/alertmanager/template \
prom/alertmanager:latest
```

```sh
sudo docker run -d --name=alertmanager -p 9093:9093 -v $PWD/alertmanager.yml:/etc/alertmanager/alertmanager.yml -v $PWD/template:/etc/alertmanager/template prom/alertmanager:latest
```



alertmanager.yml  配置文档：

```yaml
global:
  resolve_timeout: 5m
  smtp_from: 'xxxx@qq.com' # 发件人
  smtp_smarthost: 'smtp.qq.com:465' # 邮箱服务器的 POP3/SMTP 主机配置 smtp.qq.com 端口为 465 或 587
  smtp_auth_username: 'xxxx@qq.com' # 用户名
  smtp_auth_password: 'xxxxx' # 授权码 不是 QQ 密码
  smtp_require_tls: false
  smtp_hello: 'qq.com'
templates:
  - '/etc/alertmanager/template/email.tmpl'
route:
  group_by: ['alertname'] # 告警分组
  group_wait: 5s # 在组内等待所配置的时间，如果同组内，5 秒内出现相同报警，在一个组内出现。
  group_interval: 5m # 如果组内内容不变化，合并为一条警报信息，5 分钟后发送。
  repeat_interval: 5m # 发送告警间隔时间 s/m/h，如果指定时间内没有修复，则重新发送告警
  receiver: 'email' # 优先使用 email 发送
  routes: #子路由，使用 email 发送
  - receiver: email
    match_re:
      serverity: email
receivers:
- name: 'email'
  email_configs:
  - to: 'xxxxx@163.com' # 如果想发送多个人就以 ',' 做分割
    send_resolved: true
- name: 'wechat'
  wechat_configs:
  - corp_id: 'xxxxxxxxxxxxx' #企业 ID
    api_url: 'https://qyapi.weixin.qq.com/cgi-bin/' # 企业微信 api 接口 统一定义
    to_party: '2'  # 通知组 ID
    agent_id: '1000002' # 新建应用的 agent_id
    api_secret: 'xxxxxxxxxxxxxx' # 生成的 secret
    send_resolved: true
inhibit_rules:
  - source_match:
      severity: 'critical'
    target_match:
      severity: 'warning'
    equal: ['alertname', 'dev', 'instance']
```

template 模板：

参考：https://github.com/prometheus/alertmanager/tree/main/template

微信模板：wechat.tmpl

````tmpl
{{ define "wechat.default.message" }}
{{ range $i, $alert :=.Alerts }}
【系统报警】
告警状态：{{   .Status }}
告警级别：{{ $alert.Labels.severity }}
告警应用：{{ $alert.Annotations.summary }}
告警详情：{{ $alert.Annotations.description }}
触发阀值：{{ $alert.Annotations.value }}
告警主机：{{ $alert.Labels.instance }}
告警时间：{{ $alert.StartsAt.Format "2006-01-02 15:04:05" }}
{{ end }}
{{ end }}
````



###  Prometheus 添加 Alertmanager

```yaml
# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
      - targets: ["172.16.0.112:9093"]
          # - alertmanager:9093
```

### Prometheus 配置报警规则

```yaml
# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  - "memory_over.yml"
  # - "first_rules.yml"
  # - "second_rules.yml"

```

memory_over.yml

```yaml
groups:
- name: example
  rules:
  - alert: NodeMemoryUsage
    expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes+node_memory_Buffers_bytes+node_memory_Cached_bytes )) / node_memory_MemTotal_bytes * 100 > 50
    for: 1m
    labels:
      user: caizh
    annotations:
      summary: "{{$labels.instance}}: High Memory usage detected"
      description: "{{$labels.instance}}: Memory usage is above 80% (current value is:{{ $value }})"
```

其他的配置：

编写主机监控规则文件，rules/host_sys.yml

```yaml
groups:
- name: Host
 rules:
 - alert: HostMemory Usage
   expr: (node_memory_MemTotal_bytes - (node_memory_MemFree_bytes + node_memory_Buffers_bytes + node_memory_Cached_bytes)) / node_memory_MemTotal_bytes * 100 >  90
   for: 1m
   labels:
     name: Memory
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: "宿主机内存使用率超过90%."
     value: "{{ $value }}"
 - alert: HostCPU Usage
   expr: sum(avg without (cpu)(irate(node_cpu_seconds_total{mode!='idle'}[5m]))) by (instance,appname) > 0.8
   for: 1m
   labels:
     name: CPU
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: "宿主机CPU使用率超过80%."
     value: "{{ $value }}"
 - alert: HostLoad
   expr: node_load5 > 20
   for: 1m
   labels:
     name: Load
     severity: Warning
   annotations:
     summary: "{{ $labels.appname }} "
     description: " 主机负载5分钟超过20."
     value: "{{ $value }}"
 - alert: HostFilesystem Usage
   expr: (node_filesystem_size_bytes-node_filesystem_free_bytes)/node_filesystem_size_bytes*100>80
   for: 1m
   labels:
     name: Disk
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: " 宿主机 [ {{ $labels.mountpoint }} ]分区使用超过80%."
     value: "{{ $value }}%"
 - alert: HostDiskio writes
   expr: irate(node_disk_writes_completed_total{job=~"Host"}[1m]) > 10
   for: 1m
   labels:
     name: Diskio
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: " 宿主机 [{{ $labels.device }}]磁盘1分钟平均写入IO负载较高."
     value: "{{ $value }}iops"
 - alert: HostDiskio reads
   expr: irate(node_disk_reads_completed_total{job=~"Host"}[1m]) > 10
   for: 1m
   labels:
     name: Diskio
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: " 宿机 [{{ $labels.device }}]磁盘1分钟平均读取IO负载较高."
     value: "{{ $value }}iops"
 - alert: HostNetwork_receive
   expr: irate(node_network_receive_bytes_total{device!~"lo|bond[0-9]|cbr[0-9]|veth.*|virbr.*|ovs-system"}[5m]) / 1048576  > 10
   for: 1m
   labels:
     name: Network_receive
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: " 宿主机 [{{ $labels.device }}] 网卡5分钟平均接收流量超过10Mbps."
     value: "{{ $value }}3Mbps"
 - alert: hostNetwork_transmit
   expr: irate(node_network_transmit_bytes_total{device!~"lo|bond[0-9]|cbr[0-9]|veth.*|virbr.*|ovs-system"}[5m]) / 1048576  > 10
   for: 1m
   labels:
     name: Network_transmit
     severity: Warning
   annotations:
     summary: " {{ $labels.appname }} "
     description: " 宿主机 [{{ $labels.device }}] 网卡5分钟内平均发送流量超过10Mbps."
     value: "{{ $value }}3Mbps"
```

编写容器监控规则文件，rules/container_sys.yml

```yaml
groups:
- name: Container
  rules:
  - alert: ContainerCPU
    expr: (sum by(name,instance) (rate(container_cpu_usage_seconds_total{image!=""}[5m]))*100) > 200
    for: 1m
    labels:
      name: CPU_Usage
      severity: Warning
    annotations:
      summary: "{{ $labels.name }} "
      description: " 容器CPU使用超200%."
      value: "{{ $value }}%"
  - alert: Memory Usage
    expr: (container_memory_usage_bytes{name=~".+"} - container_memory_cache{name=~".+"})  / container_spec_memory_limit_bytes{name=~".+"}   * 100 > 200
    for: 1m
    labels:
      name: Memory
      severity: Warning
    annotations:
      summary: "{{ $labels.name }} "
      description: " 容器内存使用超过200%."
      value: "{{ $value }}%"
  - alert: Network_receive
    expr: irate(container_network_receive_bytes_total{name=~".+",interface=~"eth.+"}[5m]) / 1048576  > 10
    for: 1m
    labels:
      name: Network_receive
      severity: Warning
    annotations:
      summary: "{{ $labels.name }} "
      description: "容器 [{{ $labels.device }}] 网卡5分钟平均接收流量超过10Mbps."
      value: "{{ $value }}Mbps"
  - alert: Network_transmit
    expr: irate(container_network_transmit_bytes_total{name=~".+",interface=~"eth.+"}[5m]) / 1048576  > 10
    for: 1m
    labels:
      name: Network_transmit
      severity: Warning
    annotations:
      summary: "{{ $labels.name }} "
      description: "容器 [{{ $labels.device }}] 网卡5分钟平均发送流量超过10Mbps."
      value: "{{ $value }}Mbps"
```

编写redis监控规则文件，redis_check.yml

```yaml
groups:
- name: redisdown
  rules:
  - alert: RedisDown
    expr: redis_up == 0
    for: 1m
    labels:
      name: instance
      severity: Critical
    annotations:
      summary: " {{ $labels.alias }}"
      description: " 服务停止运行 "
      value: "{{ $value }}"
  - alert: Redis linked too many clients
    expr: redis_connected_clients / redis_config_maxclients * 100 > 80
    for: 1m
    labels:
      name: instance
      severity: Warning
    annotations:
      summary: " {{ $labels.alias }}"
      description: " Redis连接数超过最大连接数的80%. "
      value: "{{ $value }}"
  - alert: Redis linked
    expr: redis_connected_clients / redis_config_maxclients * 100 > 80
    for: 1m
    labels:
      name: instance
      severity: Warning
    annotations:
      summary: " {{ $labels.alias }}"
      description: " Redis连接数超过最大连接数的80%. "
      value: "{{ $value }}"
```

编写服务停止监控规则，rules/service_down.yml

```yaml
- alert: ProcessDown
    expr: namedprocess_namegroup_num_procs  == 0
    for: 1m
    labels:
      name: instance
      severity: Critical
    annotations:
      summary: " {{ $labels.appname }}"
      description: " 进程停止运行 "
      value: "{{ $value }}"
  - alert: Grafana down
    expr: absent(container_last_seen{name=~"grafana.+"} ) == 1
    for: 1m
    labels:
      name: grafana
      severity: Critical
    annotations:
      summary: "Grafana"
      description: "Grafana容器停止运行"
      value: "{{ $value }}"
```



Grafana 导入模板

```text
主机监控展示看板Node-exporter导入 8919 模板
容器监控展示看板cadvisor-exporter导入193 模板
应用监控展示看板jmx-exporter导入8563 模板
Redis监控展示看板Redis-exporter导入2751 模板
进程监控展示看板Process-exporter导入249 模板
```
