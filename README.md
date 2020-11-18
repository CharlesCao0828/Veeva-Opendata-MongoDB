# MongoDB on Kubernetes

## Lab1.实验内容
### 通过Helm安装类型为Standalone的Mongodb
### 实验步骤
- 步骤一：添加镜像仓库
  ```
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  helm repo add bitnami https://charts.bitnami.com/bitnami
  ```
- 步骤二：安装类型为standalone的MongoDB数据库
  ```
  helm install mongo-standalone bitnami/mongodb --set architecture=standalone,auth.rootPassword=Mongo@12345,persistence.size=20Gi
  ```
- 步骤三：查看安装状态与结果
  ```
  ## 查看pvc创建
  kubectl get pvc

  ## 查看Mongodb节点
  kubectl get pod

  ## 查看kubernetes service
  kubectl get svc
  ```
- 步骤四： 创建MongoDB客户端，并登录MongoDB集群。
  ```
  ## 创建MongoDB客户端
  kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il

  ## 登录MongoDB集群
  mongo admin --host mongo-standalone-mongodb -u root -p Mongo@12345

  ## 创建数据库
  use online-shop

  ## 创建集合并插入文档
  db.inventory.insertMany([
    { item: "journal", qty: 25, status: "A", size: { h: 14, w: 21, uom: "cm" }, tags: [ "blank", "red" ] },
    { item: "notebook", qty: 50, status: "A", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank" ] },
    { item: "paper", qty: 10, status: "D", size: { h: 8.5, w: 11, uom: "in" }, tags: [ "red", "blank", "plain" ] },
    { item: "planner", qty: 0, status: "D", size: { h: 22.85, w: 30, uom: "cm" }, tags: [ "blank", "red" ] },
    { item: "postcard", qty: 45, status: "A", size: { h: 10, w: 15.25, uom: "cm" }, tags: [ "blue" ] }
  ]);

  ## 查询文档
  db.inventory.find({})
  ```
- 步骤五： 删除MongoDB standalone实例。
  ```
  ## 删除helm安装版本
  helm ls
  helm uninstall mongo-standalone

  ## 查询pod
  kubectl get pod

  ## 查询pvc
  kubectl get pvc
  ```

******
## Lab2.实验内容
### 通过Helm安装类型为Replicaset的MongoDB
### 实验步骤
- 步骤一：通过Helm安装MongoDB Replicaset集群
  ```
  helm install mongo-replicaset bitnami/mongodb --set architecture=replicaset,auth.rootPassword=Mongo@12345,persistence.size=20Gi,replicaCount=2
  ```
- 步骤二：查看Kubernetes资源与MongoDB集群创建状态
  ```
  ## 查看pvc创建
  kubectl get pvc

  ## 查看Mongodb节点
  kubectl get pod

  ## 查看kubernetes service
  kubectl get svc
  ```
- 步骤三：待集群节点全部正常运行后， 创建MongoDB客户端，并登录MongoDB集群。
  ```
  ## 创建MongoDB客户端
  kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il

  ## 登录MongoDB集群
  mongo admin --host "mongo-replicaset-mongodb-0.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-1.mongo-replicaset-mongodb-headless.default.svc.cluster.local," --authenticationDatabase admin -u root -p Mongo@12345

  ## 查看MongoDB Replicaset 状态
  rs.status()

  ## 查看slave节点数量
  rs.printSlaveReplicationInfo()
  ```
- 步骤四：调整MongoDB集群Replicas数量，对集群进行扩缩容
  ```
  ## 退出mongo shell客户端
  exit 

  ## 获取MONGODB_REPLICA_SET_KEY
  kubectl get secret --namespace default mongo-replicaset-mongodb -o jsonpath="{.data.mongodb-replica-set-key}" | base64 --decode
  
  ## 更新helm，将MongoDB Replicaset的数量扩展为3个
  helm upgrade  mongo-replicaset bitnami/mongodb --set architecture=replicaset,auth.rootPassword=Mongo@12345,persistence.size=20Gi,replicaCount=3,auth.replicaSetKey=<MONGODB_REPLICA_SET_KEY>
  ```
- 步骤五：等待数分钟后查看结果
  ```
  ## 创建MongoDB客户端
  kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il
  
  ## 登录MongoDB集群
  mongo admin --host "mongo-replicaset-mongodb-0.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-1.mongo-replicaset-mongodb-headless.default.svc.cluster.local,mongo-replicaset-mongodb-2.mongo-replicaset-mongodb-headless.default.svc.cluster.local," --authenticationDatabase admin -u root -p Mongo@12345

  ## 查看MongoDB Replicaset 状态
  rs.status()

  ## 查看MongoDB Slave节点数量
  rs.printSlaveReplicationInfo()

  ## 删除mongo-replicaset
  helm uninstall mongo-replicaset
  ```


******
## Lab3.实验内容
### 通过Helm安装类型为Sharded的MongoDB集群
### 实验步骤
- 步骤一：通过Helm安装类型为Sharded的MongoDB集群
  ```
  helm install mongo-sharded bitnami/mongodb-sharded --set shards=1,mongos.replicas=1,mongodbRootPassword=Mongo@12345,shardsvr.persistence.size=20
  ```
- 步骤二：查看Kubernetes资源与MongoDB集群创建状态
  ```
  ## 查看pvc创建
  kubectl get pvc

  ## 查看Mongodb节点
  kubectl get pod

  ## 查看kubernetes service
  kubectl get svc
  ```
- 步骤三：待集群节点全部正常运行后， 创建MongoDB客户端，并登录MongoDB集群。
  ```
  ## 创建MongoDB客户端
  kubectl run -i --rm --tty percona-client --image=percona/percona-server-mongodb:4.2.8-8 --restart=Never -- bash -il

  ## 登录MongoDB集群
  mongo admin --host mongo-sharded-mongodb-sharded --authenticationDatabase admin  -u root -p Mongo@12345
  ```
- 步骤四：查看MongoDB集群shard信息
  ```
  ## 在MongoDB shell里执行
  sh.status()
  ```
- 步骤五：尝试更改MongoDB Shard的Replica数量，提升MongoDB集群的高可用性。
  ```
  kubectl get secret mongo-sharded-mongodb-sharded -o jsonpath="{.data.mongodb-replica-set-key}" | base64 --decode
  helm upgrade mongo-sharded bitnami/mongodb-sharded --set shards=2,shardsvr.dataNode.replicas=2,mongodbRootPassword=Mongo@12345,shardsvr.persistence.size=20,replicaSetKey=<MONGODB_REPLICA_SET_KEY>
  ```
- 步骤六：查看Kubernetes资源，确认MongoDB新的Shard节点与存储卷被创建。
  ```
  ## 查看pvc创建
  kubectl get pvc

  ## 查看Mongodb节点
  kubectl get pod

  ## 查看kubernetes service
  kubectl get svc
  ```
- 步骤七：查看新创建的MongoDB节点日志。
  ```
  kubectl logs mongo-sharded-mongodb-sharded-shard0-data-1

  ## 登录MongoDB集群
  mongo admin --host mongo-sharded-mongodb-sharded --authenticationDatabase admin  -u root -p Mongo@12345

  ## 查看MongoDB集群shard状态
  sh.status()
  ```

******
## Lab4.实验内容

通过业界常用的两种MongoDB Operator（MongoDB Enterprise Operator 以及 Percona Kubernetes Operator）创建并管理MongoDB集群，完成MongoDB集群创建、容量调整、集群使用、数据备份、集群监控等任务，同时并对比两种Operator的优劣势。

### 利用MongoDB Enterprise Operator管理MongoDB集群

### MongoDB Enterprise Operator描述
[MongoDB Enterprise Operator](https://docs.mongodb.com/kubernetes-operator/master/)可用于在Kubernetes平台上管理MongoDB集群的生命周期，包括集群的创建、容量的调整、服务通信、数据备份、快照恢复等任务。MongoDB Enterprise Operator基于标准Kubernetes API开发而成，具有平台无关性，可以安装在各种云厂商以及软件发行商的提供的Kubernetes平台上。其基于标准Kubernetes API开发而成，并提供CRD，开发运维人员仅需要按照MongoDB Enterprise Operator提供的的CRD进行定义并提交任务即可，MongoDB Enterprise Operator会自动地创建Kubernetes Statefulset/PVC/ConfigMap等资源自动地完成MongoDB集群的创建和配置。

### 实验前提
- 完成EKS集群的创建
- 客户端已经完成kubectl工具的安装和配置
- 客户端已经完成helm工具的安装

### 实验步骤
- 步骤一：安装MongoDB Enterprise Operator
  - 1.1 克隆MongoDB Enterprise Operator gihub链接.
  ```
  git clone https://github.com/mongodb/mongodb-enterprise-kubernetes.git
  ```
  - 1.2 创建Kubernetes Namespace，MongoDB Operator与MongoDB集群都将运行在此命名空间下。
  ```
  kubectl create namespace mongodb
  ```
  - 1.3 通过helm工具安装MongoDB Enterprise Operator
  ```
  ## 切换路径
  cd mongodb-enterprise-kubernetes/

  ## 通过helm安装MongoDB Enterprise Operator
  helm install mongodb helm_chart \
     --values helm_chart/values.yaml
  ```
  - 1.4 运行命令查看MongoDB Enterprise Operator安装结果，MongoDB Enterprise Operator会以pod的形式运行在mongodb namespace下。
  ```
   kubectl get pod -n mongodb 
  ```
- 步骤二：部署OpsManager实例
  - 2.1 设置kubectl context信息，让kubectl默认工作在mongodb命名空间下。
  ```
  kubectl config set-context $(kubectl config current-context) --namespace=mongodb
  ```
  - 2.2 创建Kubernetes Secret，新创建的两个Secret用于存放OpsManager用户名和密码，以及OpsManager Applicaton Database密码。当OpsManager成功创建后，用户可用该OpsManager的用户名和密码登录并OpsManager，可用OpsManager Application Database的密码对OpsManager的Databse进行维护。
  ```
  ## 存放OpsManager用户名密码
  kubectl create secret generic om-admin-secret \
  --from-literal=Username=admin \
  --from-literal=Password=Admin@123 \
  --from-literal=FirstName=San\
  --from-literal=LastName=Zhang

  ## 存放OpsManager Applicaton Database
  kubectl create secret generic om-db-user-secret \
  --from-literal=password=Admin@12345
  ```
  - 2.3 通过OpsManager yaml配置文件创建OpsManager实例。若需要进一步定制化配置OpsManager，请查看[详细配置](https://docs.mongodb.com/kubernetes-operator/master/reference/k8s-operator-om-specification/)。
  ```
  kubectl apply -f kubernetes/mongodb-ops-manager.yml
  ```
  - 2.4 查看OpsManager部署状态。
  ```
  kubectl get om mongodb-ops-manager -o yaml
  kubectl get pod
  ```
  - 2.5 查询OpsManager url，访问OpsManager界面。默认情况下，MongoDB Enterprise Operator会自动为OpsManager创建类型为LoadBalancer的Kubernetes Service，用户可通过该Service对应的ELB URL访问OpsManager图形界面。利用2.2步创建的OpsManager用户名和密码登录OpsManager，查看OpsManager支持的功能。
  ```
  kubectl get svc |grep mongodb-ops-manager-svc-ext 
  ```

- 步骤三：部署MongoDB Database
  - 3.1 登录OpsManager创建API KEY，API KEY的作用范围可以是global、organization或者project。通过API KEY，第三方应用程序可以在与OpsManager进行交互，同时也会收到相应的权限控制。
  - 3.2 利用API KEY创建Kubernetes的Secret，在创建MongoDB数据库之前，MongoDB Operator会引用该Secret，向OpsManager注册即将创建的MongoDB数据库集群信息，待MongoDB集群创建成功后，MongoDB集群的信息会自动显示在OpsManager的界面上。
  - 3.3 通过MongoDB yaml文件创建MongoDB集群，MongoDB Enterprise Operator支持以[Standalone](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-standalone/)、[Replica Set](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-replica-set/)、[Sharded Cluster](https://docs.mongodb.com/kubernetes-operator/master/tutorial/deploy-replica-set/)三种模式部署集群，用户可以在MongoDB yaml文件中定义相应的类型。除了集群类型，用户也可以在yaml文件中定义其它与集群相关的信息，如MongoDB版本、集群数量、存储卷大小、启动参数等信息，若需要对集群进行进一步定制化配置，请参考[详细信息](https://docs.mongodb.com/kubernetes-operator/master/reference/k8s-operator-specification/#replica-set-settings)。本步骤会创建一个类型为Replica Set，节点数量为三的MongoDB集群。
  - 3.4 查看MongoDB集群部署的状态

- 步骤四：MongoDB数据备份与快照恢复
  - 4.1 创建S3 Bucket
  - 4.2 获取AWS IAM accessKey与secretKey的信息，并利用该信息创建Kubernetes Secret，MongoDB OpsManager与Backup Agent会通过该Secret值与AWS S3交互，并将MongoDB集群数据备份的快照存放在S3存储桶上。
  - 4.3 启用MongoDB备份功能，备份功能的启用与参数的定义可以在OpsManager yaml文件中进行定义，用户可在该文件中定义MongoDB备份的存储介质以及目标集群等信息，如需更加详细地定义备份任务，请参考详细配置。
  - 4.4 连接MongoDB集群，并在集群上创建数据集
  - 4.5 登录OpsManager，为MongoDB集群创建备份任务、置备份策略并对集群创建快照。
  - 4.6 连接MongoDB集群，并删除MongoDB数据。
  - 4.7 登录OpsManager，恢复MongoDB集群快照。
  - 4.8 连接MongoDB集群，验证MongoDB集群数据是否恢复。
  
  
