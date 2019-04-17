# 与 Istio 协作

本文是关于如何将 Istio 的度量指标发送到 SkyWalking OAP 服务的介绍.

## 前提

Istio 已经安装在 kubernetes 集群中. 参考 [Istio 快速启动](https://istio.io/docs/setup/kubernetes/quick-start/)
进行安装.

## 部署 SkyWalking backend

参考 [在 kubernetes 中部署后端](../backend/backend-k8s.md)一文在 kubernetes 集群中安装.

## 设置 Istio 向 OAP 发送度量指标

按照 [设置 Istio 向 OAP 发送度量指标](https://github.com/apache/incubator-skywalking-kubernetes#setup-istio-to-send-metric-to-oap)
中的指南设置 Istio 与 OAP 进行协作.
