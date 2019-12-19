# Prometheus with Thanos sidecar
在 Prometheus 官方 [helm chart](https://github.com/helm/charts/tree/master/stable/prometheus) 下进行修改  
请搭配 thanos-chart 项目食用

### 特性
- Thanos sidecar 默认开启，并创建 grpc Headless Service
- Thanos 对象存储默认关闭
- 多个副本时，需要配置 global.extraLabels 来区分不同副本
- Prometheus Server 的 ConfigMapReloader容器被删除，Thanos sidecar 可以提供配置更新
- Prometheus Server 的 service 默认关闭，不需要开启
- Thanos 开启对象存储时， 通过配置 Values.server.blockDuration 使 Prometheus的min-block-duration和max-block-duration 需要保持一致来禁止 Prometheus 压缩数据
- 支持 Prometheus 配置文件中使用环境变量
-  Thanos sidecar 和 Prometheus 共享 Values.service.env 中设置的环境变量以及额外的挂载卷 (如果有特殊需求可以进行修改该特性)

### 未实现功能
- 未增加 Thanos sidecar http Service, 如果有额外需要，可实现该功能
- 未实现自动生成 Thanos 对象存储的ConfigMap，因为 thanos-chart 也需要该ConfigMap，所以需要开启对象存储时需要先创建ConfigMap


## 安装

```
helm install -n <namespace> -f values.yaml prom_thanos ./prometheus
```
## 更新

```
helm upgrade -n <namespace> -f values.yaml prom_thanos
```
## 删除

```
helm uninstall -n <namespace> prom_thanos
```

### 开启对象存储
保存Thanos 对象存储 ConfigMap, 和 Thanos store 组件共享一个ConfigMap， 可参考 thanos-chart 项目
```
kubectl -n <namespaece> apply -f <thanos-objstore-configmap-name> 
```
##### 可以使用 [fake-s3](https://github.com/jubos/fake-s3) 项目进行 s3 存储测试

需要配置 blockDuration, 使Prometheus 的 min-block-duration, max-block-duration 一致 https://thanos.io/components/sidecar.md
```
blockDuration: 2h
```

values.yaml

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

## 和 stable/prometheus 配置的差异

Parameter | Description | Default
--------- | ----------- | -------
`server.thanos.image.repository` | | `thanosio/thanos`
`server.thanos.image.tag` | | `master-2019-12-14-bec86666`
`server.thanos.image.pullPolicy` | | `IfNotPresent`
`server.thanos.logLevel`| | debug
`server.thanos.extraArgs` | | {}
`server.thanos.resources` | | {}
`server.thanos.objstore.enabled` | | false
`server.thanos.objstore.configFileName` | | ""
`server.thanos.objstore.configMapName` | | ""
`server.thanos.headless.annotations` | | {}
`server.thanos.headless.labels` | | {}
`server.thanos.headless.servicePort` | | 10901
`server.blockDuration` | `设置 Prometheus max block duration 和 min block duration 使其相同大小， 开启 Thanos 对象存储需要Prometheus 关闭数据压缩` | ""
`server.service.enabled` | `是否开启 Prometheus 的 http service, 使用 Thanos 默认关闭` | false
`server.configPath` | `废弃` |
