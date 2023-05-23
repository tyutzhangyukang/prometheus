# prometheus
通过dcgm-exporter/node-exporter监控主机资源
https://hub.docker.com/r/zhangyukang/prometheus-node-exporter
https://hub.docker.com/r/zhangyukang/nvidia-k8s-dcgm-exporter

创建守护进程service文件 

`vim /etc/systemd/system/docker.node-exporter.service 
[Unit]
Description=Prometheus Node Exporter
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull cache-mgmt001:5001/prometheus/node-exporter
ExecStart=/usr/bin/docker run --rm --network host --cpus=0.5  --pid=host --name %n -v /run/prometheus:/run/prometheus cache-mgmt001:5001/prometheus/node-exporter --collector.textfile.directory="/run/prometheus"

[Install]
WantedBy=multi-user.target`



vim /etc/systemd/system/docker.dcgm-exporter.service 

`[Unit]
Description=NVIDIA DCGM Exporter
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull cache-mgmt001:5001/nvidia/k8s/dcgm-exporter:2.1.8-2.4.0-rc.3-ubi8
ExecStart=/usr/bin/docker run --rm --gpus all --cap-add=SYS_ADMIN --cpus="0.5" --name %n -p 9400:9400 -v "/opt/deepops/nvidia-dcgm-exporter/dcgm-custom-metrics.csv:/etc/dcgm-exporter/default-counters.csv" cache-mgmt001:5001/nvidia/k8s/dcgm-exporter:2.1.8-2.4.0-rc.3-ubi8

[Install]
WantedBy=multi-user.target`


将cvs文件放在/opt/deepops/nvidia-dcgm-exporter/dcgm-custom-metrics.csv底下 https://github.com/tyutzhangyukang/prometheus
创建容器 两个镜像分别来自于： nvidia/k8s/dcgm-exporter:2.1.8-2.4.0-rc.3-ubi8 prometheus/node-exporter:latest




prometheus + grafana节点
vim /etc/systemd/system/docker.prometheus.service
`
[Unit]
Description=Prometheus
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull prom/prometheus
ExecStart=/usr/bin/docker run --rm --network host --name %n -v /etc/prometheus:/etc/prometheus -v deepops_prometheus_metrics:/prometheus prom/prometheus

[Install]
WantedBy=multi-user.target
`

vim /etc/systemd/system/docker.grafana.service
`
[Unit]
Description=Grafana
After=docker.service
Requires=docker.service

[Service]
TimeoutStartSec=0
Restart=always
ExecStartPre=-/usr/bin/docker stop %n
ExecStartPre=-/usr/bin/docker rm %n
ExecStartPre=/usr/bin/docker pull grafana/grafana
ExecStart=/usr/bin/docker run --rm --network host --name %n -v /etc/grafana:/etc/grafana -v /var/lib/grafana:/var/lib/grafana grafana/grafana

[Install]
WantedBy=multi-user.target
`

