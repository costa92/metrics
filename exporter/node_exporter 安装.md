#  安装 node_exporter

## docker 安装

```sh
docker run -d --name=node -p 9100:9100 prom/node-exporter:latest
```

docker 运行配置的信息参考： [github](https://github.com/prometheus/node_exporter)

## node exporter版本的二进制包:

**mac 安装**

```sh
curl -OL https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.darwin-amd64.tar.gz
tar -xzf node_exporter-1.3.1.darwin-amd64.tar.gz
```

运行node exporter:

```sh
cd node_exporter-0.15.2.darwin-amd64
cp node_exporter-0.15.2.darwin-amd64/node_exporter /usr/local/bin/
node_exporter
```

启动成功后，可以看到以下输出：

```sh
INFO[0000] Listening on :9100                            source="node_exporter.go:76"
```



**linux 安装**(centos)

```sh
curl -OL https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar -xzf node_exporter-1.3.1.linux-amd64.tar.gz
mv node_exporter-0.18.1.linux-amd64 /data/node_exporter
chown prometheus:prometheus -R /data/node_exporter
```

封装service

```sh
[Unit]
Description=Prometheus Node Exporter
After=network.target
[Service]
ExecStart=/data/node_exporter/node_exporter
User=prometheus
[Install]
WantedBy=multi-user.target
```

置开机自启动

```sh
systemctl daemon-reload
systemctl enable node-exporter
systemctl start node-exporter
```

 查看端口 

```sh
ss -tunlp|grep node
```



## 修改 prometheus 配置文件

```yaml
    static_configs:
      - targets: ["localhost:9090"]

  - job_name: "node"
    scrape_interval: 10s
    metrics_path: "/metrics"
    static_configs:
      - targets: ["172.16.0.112:9100"]
```

如下监控指标：

​	node_boot_time：系统启动时间

​	node_cpu：系统CPU使用量

​	node*disk**：磁盘IO

​	node*filesystem**：文件系统用量

​	node_load1：系统负载

​	node*memeory**：内存使用量

​	node*network**：网络带宽

​	node_time：当前系统时间

​	go_*：node exporter中go相关指标

​	process_*：node exporter自身进程相关运行指标

**重新启动Prometheus Server**

访问http://localhost:9090，进入到Prometheus Server。如果输入“up”并且点击执行按钮以后，可以看到如下结果：

![image-20220722123621896](https://file.longqiuhong.com/uploads/picgo/image-20220722123621896.png)

点击 菜单栏中的 status --> Tagrets 就会显示下面的数据

![image-20220722123428217](https://file.longqiuhong.com/uploads/picgo/image-20220722123428217.png)

##  Grafana 导入  Dashboard 模板

通过 https://grafana.com/dashboards 网站，可以找到大量可直接使用的Dashboard：比如我这里选择了热门模板（ID：8919），展示效果如下：

![image-20220722124325787](https://file.longqiuhong.com/uploads/picgo/image-20220722124325787.png)

![image-20220722124344748](https://file.longqiuhong.com/uploads/picgo/image-20220722124344748.png)

Dashboard 访问结果

![image-20220722124415834](https://file.longqiuhong.com/uploads/picgo/image-20220722124415834.png)

或使用 https://grafana.com/grafana/dashboards/12486 模板

![image-20220722140016993](https://file.longqiuhong.com/uploads/picgo/image-20220722140016993.png)

