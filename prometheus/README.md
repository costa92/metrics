# prometheus 
## install
### docker install
安装命令：

使用默认配置文件
```sh
docker run \ 
    -p 9090:9090 \ 
    -name prometheus \
    prom/prometheus
```

使用指定配置文件
```sh
docker run \
    -p 9090:9090 \
    -name prometheus \
    -v /path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
    prom/prometheus
```
 或者指定绑定配置路径

 ```sh
docker run \
    -p 9090:9090 \
    -n prometheus \
    -v /path/to/config:/etc/prometheus \
    prom/prometheus 
 ```

### redis 配置

 新建 config 文件下新 targets/redis/redis.yml


 ```yaml
[
  {
    "targets": [ "172.16.0.112:9121"],
    "labels":  {"env":"test","cluster":"test_one","alias":"test2"}
  }
]

 ```

 修改 promentheus.yaml
 ```yaml
# my global config
global:
  scrape_interval: 15s # Set the scrape interval to every 15 seconds. Default is every 1 minute.
  evaluation_interval: 15s # Evaluate rules every 15 seconds. The default is every 1 minute.
  # scrape_timeout is set to the global default (10s).

# Alertmanager configuration
alerting:
  alertmanagers:
    - static_configs:
        - targets:
          # - alertmanager:9093

# Load rules once and periodically evaluate them according to the global 'evaluation_interval'.
rule_files:
  # - "first_rules.yml"
  # - "second_rules.yml"

# A scrape configuration containing exactly one endpoint to scrape:
# Here it's Prometheus itself.
scrape_configs:
  # The job name is added as a label `job=<job_name>` to any timeseries scraped from this config.
  - job_name: "prometheus"

    # metrics_path defaults to '/metrics'
    # scheme defaults to 'http'.

    static_configs:
      - targets: ["localhost:9090"]
   
  # 新增 redis 配置文件路径
  - job_name: 'redis_exporter'
    file_sd_configs:
    - refresh_interval: 1m
      files:
      - "/etc/prometheus/targets/redis/*.yml"

 ```
**********注意*： prometheus 监控 redis 需要用到 redis_exporter

需要安装 redis_exporter
1. 安装
```sh
$ cd /usr/local/src
$ wget https://github.com/oliver006/redis_exporter/releases/download/v0.21.2/redis_exporter-v0.21.2.linux-amd64.tar.gz
$ mkdir /usr/local/redis_exporter
$ tar xf redis_exporter-v0.21.2.linux-amd64.tar.gz -C /usr/local/redis_exporter/
```
2. 启动
```sh
## 无密码
./redis_exporter redis//127.0.0.1:6379 &
## 有密码
redis_exporter  -redis.addr 127.0.0.1:6379  -redis.password 123456 
```
参考：https://wiki.eryajf.net/pages/2497.html#_2%E3%80%81redis-exporter-%E7%94%A8%E6%B3%95

3. 重启 prometheus 





