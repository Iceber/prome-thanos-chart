# Prometheus with Thanos sidecar
在 Prometheus 官方 helm chart 下进行修改

### 特性
- Thanos sidecar 默认开启，并创建 grpc Headless Service
- Thanos 对象存储默认关闭
- 多个副本时，需要配置 global.extraLabels 来区分不同副本
- Prometheus Server 的 ConfigMapReloader容器被删除，Thanos sidecar 可以提供配置更新
- Prometheus Server 的 service 默认关闭，不需要开启
- Thanos 开启对象存储时， 通过配置 Values.server.blockDuration 使 Prometheus的min-block-duration和max-block-duration 需要保持一致来禁止 Prometheus 压缩数据
- 支持 Prometheus 配置文件中使用环境变量
-  Thanos sidecar 和 Prometheus 共享 Values.service.env 中设置的环境变量以及额外的挂载卷 (如果有特殊需求可以进行修改该特性)


## 安装

```
helm install -n <namespace> -f values.yml prom_thanos ./prometheus
```
## 更新

```
helm upgrade -n <namespace> -f values.yml prom_thanos
```
## 删除

```
helm uninstall -n <namespace> prom_thanos
```

values.yml

```
rbac:
    create: true

alertmanager:
    enabled: false

pushgateway:
    enabled: false

nodeExporter:
    enabled: false

kubeStateMetrics:
    enabled: false

server:
  persistentVolume:
      enabled: false
  replicaCount: 2
  statefulSet:
      enabled: true

  blockDuration: 5m
  thanos:
      repository: dockerhub.azk8s.cn/thanosio/thanos
      objstore:
          enabled: true
          configFileName: s3.yml
          configMapName: thanos-objstore-s3-config

  global:
      scrape_interval: 15s
      scrape_timeout: 4s
      external_labels:
          prometheus_group: 0
          prometheus_replica: $(HOSTNAME)
```
