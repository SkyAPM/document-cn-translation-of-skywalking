# 与 Istio 协作

介绍如何将 Istio 的度量指标发送到 SkyWalking OAP 服务.

## 前提

Istio 已经安装在 kubernetes 集群中. 参考 [Istio 快速启动](https://istio.io/docs/setup/kubernetes/quick-start/)
完成安装.

## 部署 SkyWalking 后台

参考 [K8S 中部署后端](../backend/backend-k8s.md) 一文在 K8S 中安装 OAP Server。

## 设置 Istio 向 OAP 发送度量指标

我们的脚本基于 Istio 1.3.3 编写

### 1. 安装 Istio 指标模板

`kubectl apply -f https://raw.githubusercontent.com/istio/istio/1.3.3/mixer/template/metric/template.yaml`

### 2. 安装 SkyWalking 适配器

`kubectl apply -f skywalkingadapter.yml`

在 [此处](yaml/skywalkingadapter.yml) 查找 `skywalkingadapter.yml`

注意，由于 Istio Mixer 默认时关闭的，我们建议你考虑我们的 [ALS 方案](../envoy/als_setting.md)
