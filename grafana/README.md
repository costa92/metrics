# Grafana

## install

### docker install grafana

```sh
docker run -d --name grafana  -p 3000:3000 grafana/grafana grafana
```

docker run config

```sh
docker run -d --name grafana -p 3000:3000  -v /to/path/grafana.ini:/etc/grafana/grafana.ini grafana/grafana grafana
```



新建 执行启动脚本 restart_granfana.sh  

```sh

#!/bin/bash
basedir=$(cd `dirname $0`;pwd)

mkdir -p data # creates a folder for your data
ID=$(id -u) # saves your user id in the ID variable

docker stop grafana
docker rm grafana
docker run \
       -d --name grafana  -p 3000:3000 \
       -e "GF_SERVER_ROOT_URL=http://grafana.server.name" \    
       -e "GF_SECURITY_ADMIN_PASSWORD=newpwd" \
       --user $ID --volume "$PWD/data:/var/lib/grafana" \   // 指定挂载数据文件
       grafana/grafana grafana
```



设置服务的默认域名: GF_SERVER_ROOT_URL

设置admin的密码为:  GF_SECURITY_ADMIN_PASSWORD

### 环境变量配置的默认路径

| 环境变量              | 默认值                    |
| :-------------------- | :------------------------ |
| GF_PATHS_CONFIG       | /etc/grafana/grafana.ini  |
| GF_PATHS_DATA         | /var/lib/grafana          |
| GF_PATHS_HOME         | /usr/share/grafana        |
| GF_PATHS_LOGS         | /var/log/grafana          |
| GF_PATHS_PLUGINS      | /var/lib/grafana/plugins  |
| GF_PATHS_PROVISIONING | /etc/grafana/provisioning |



导入数据源：

![image-20220720151921052](https://file.longqiuhong.com/uploads/picgo/image-20220720151921052.png)

![image-20220720151951271](https://file.longqiuhong.com/uploads/picgo/image-20220720151951271.png)

![image-20220720152028452](https://file.longqiuhong.com/uploads/picgo/image-20220720152028452.png)

![image-20220720152044821](https://file.longqiuhong.com/uploads/picgo/image-20220720152044821.png)
