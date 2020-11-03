# MongoDB on Kubernetes

## 实验内容

通过业界常用的两种MongoDB Operator（MongoDB Enterprise Operator 以及 Percona Kubernetes Operator）创建并管理MongoDB集群，完成MongoDB集群创建、容量调整、集群使用、数据备份、集群监控等任务，同时并对比两种Operator的优劣势。

## 实验一
### 利用MongoDB Enterprise Operator管理MongoDB集群

### MongoDB Enterprise Operator描述
MongoDB Enterprise Operator可用于在Kubernetes平台上管理MongoDB集群的生命周期，包括集群的创建、容量的调整、服务通信、数据备份、快照恢复等任务。MongoDB Enterprise Operator基于标准Kubernetes API开发而成，具有平台无关性，可以安装在各种云厂商以及软件发行商的提供的Kubernetes平台上。其基于标准Kubernetes API开发而成，并提供CRD，开发运维人员仅需要按照MongoDB Enterprise Operator提供的的CRD进行定义并提交任务即可，MongoDB Enterprise Operator会自动地创建Kubernetes Statefulset/PVC/ConfigMap等资源自动地完成MongoDB集群的创建和配置。

### 实验步骤

- 步骤一：安装MongoDB Enterprise Operator
