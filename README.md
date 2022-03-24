# 指导手册

## 1. Service-Center注册中心

### 1.1 SC的下载

下载地址：

https://github.com/apache/servicecomb-service-center/releases

下载最新的 2.1.0 版本（根据自身OS选择合适的软件包），下载后可直接运行。

```sh
# 第一步
start-service-center.sh (Linux/Mac) / start-service-center.bat(Windows)
# 第二步
start-frontend.sh(Linux/Mac) / start-frontend.bat(Windows)
```

 前段访问页面：http://127.0.0.1:30103/

![](./img/front.png)

> 这边使用内置的 db 所以可直接使用

---

### 1.2  etcd 高可用集群搭建

这边可参考官方文档：

https://etcd.io/docs/v3.5/op-guide/clustering/

https://etcd.io/docs/v3.5/op-guide/security/

https://blog.51cto.com/mageedu/2699744

### 1.3 二进制编译

一、前提：配置好go sdk，下载地址：https://go.dev/dl/

二、下载代码库

> https://github.com/apache/servicecomb-service-center.git

三、解决依赖

`go module` 是go语言从1.11版本之后官方推出的版本管理工具。

proxy代理设置：https://goproxy.cn/

```go
# Download the modules
GO111MODULE=on 
go mod download
```

四、编译

```go
go build -o service-center github.com/apache/servicecomb-service-center/cmd/scserver
```

> GO可以跨平台编译，重点关注
>
> GOOS=windows //程序构建环境目前OS
>
> GOARCH=amd64 //程序构建环境的目标计算架构

### 1.4 API信息

地址：https://github.com/apache/servicecomb-service-center/blob/master/docs/openapi/v4.yaml

### 1.5 更多资料

可以查看使用手册以及官网的readme：

1.https://service-center.readthedocs.io/en/latest/

2.https://github.com/apache/servicecomb-service-center

## 2. Sermant

### 1.1 Sermant的下载

https://github.com/huaweicloud/Sermant/releases

![](./img/sermant-v0.1.0.png)

### 1.2 编译构建

- 编译机器需具备[git](https://git-scm.com/downloads) ,[jdk 8或11](https://www.oracle.com/java/technologies/downloads/) ,[maven](https://maven.apache.org/download.cgi) 环境
- 执行`git clone -b develop https://github.com/huaweicloud/Sermant.git` 克隆最新源码
- 执行`cd Sermant`进入源码目录
- 执行`mvn clean package -Dmaven.test.skip -Pexample` 编译示例项目

![](./img/mvn-clean-package.png)

### 1.3 注册中心插件介绍文档

https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/document.md

详细信息见官方文档:https://github.com/huaweicloud/Sermant

## 3. 演示

### 3.1 安装概述

#### 3.1.1 部署拓扑

![](./img/deploy.png)

* sc 为注册中心，本组件为无状态服务，可根据系统规模部署多个
* etcd 高可用集群至少需要 3 个节点

按照上述的部署拓扑需要 8 台vm。

#### 3.1.2 安装流程

![](./img/process.png)

### 3.2 安装前准备

#### 3.2.1 准备环境

我这边是个人机子安装了虚拟机。由于我这边个人机子资源有限，只申请了 6 台虚拟机。

OS: Centos 7

![](./img/vm.png)

| vm-ip          | 域       |
| -------------- | -------- |
| 192.168.81.128 | region-1 |
| 192.168.81.129 | region-1 |
| 192.168.81.130 | region-1 |
| 192.168.81.131 | region-2 |
| 192.168.81.132 | region-2 |
| 192.168.81.133 | region-2 |

#### 3.2.2 获取软件包

> etcd 根据要求获取相应版本建议3xx，我这边使用了最新的版本。

* SC：https://github.com/apache/servicecomb-service-center/releases
* ETCD：https://github.com/etcd-io/etcd/releases

![](./img/sc-linux.png)

![](./img/etcd-linux.png)


### 3.3 安装操作

#### 3.3.1 安装 etcd

> https://etcd.io/docs/v3.5/op-guide/clustering/
>
> 这边需要**注意**：如果你机子单独有固态盘的话，可以把 etcd 安装在固态上，提高性能。

**region-1**

```sh
前提：包已经上传到虚拟机的制定目录，并解压

进入3 台 etcd 所在目录，执行（这一块可以由systemd去维护）,需要创建下数据以及日志存放的位置。

./etcd --name=etcd-01 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.128:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.128:2379 \
--listen-peer-urls=http://192.168.81.128:2380 \
--initial-advertise-peer-urls=http://192.168.81.128:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.128:2380,etcd-02=http://192.168.81.129:2380,etcd-03=http://192.168.81.130:2380 \
--initial-cluster-state new 


./etcd --name=etcd-02 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.129:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.129:2379 \
--listen-peer-urls=http://192.168.81.129:2380 \
--initial-advertise-peer-urls=http://192.168.81.129:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.128:2380,etcd-02=http://192.168.81.129:2380,etcd-03=http://192.168.81.130:2380 \
--initial-cluster-state new 

./etcd --name=etcd-03 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.130:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.130:2379 \
--listen-peer-urls=http://192.168.81.130:2380 \
--initial-advertise-peer-urls=http://192.168.81.130:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.128:2380,etcd-02=http://192.168.81.129:2380,etcd-03=http://192.168.81.130:2380 \
--initial-cluster-state new 

```

可以通过etcd --help查看启动参数说明

**如何检验etcd集群是否安装成功**

```sh
./etcdctl --endpoints=http://192.168.81.128:2379 endpoint  status --cluster -w table
```

![](./img/etcd-check.png)

看上图可知，ip为192.168.81.128节点选为 leader。

---

**region-2**

```sh
./etcd --name=etcd-01 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.131:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.131:2379 \
--listen-peer-urls=http://192.168.81.131:2380 \
--initial-advertise-peer-urls=http://192.168.81.131:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.131:2380,etcd-02=http://192.168.81.132:2380,etcd-03=http://192.168.81.133:2380 \
--initial-cluster-state new 

./etcd --name=etcd-02 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.132:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.132:2379 \
--listen-peer-urls=http://192.168.81.132:2380 \
--initial-advertise-peer-urls=http://192.168.81.132:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.131:2380,etcd-02=http://192.168.81.132:2380,etcd-03=http://192.168.81.133:2380 \
--initial-cluster-state new 

./etcd --name=etcd-03 \
--data-dir=/var/lib/etcd/default.etcd \
--log-outputs=/var/lib/etcd/log \
--listen-client-urls=http://192.168.81.133:2379,http://127.0.0.1:2379 \
--advertise-client-urls=http://192.168.81.133:2379 \
--listen-peer-urls=http://192.168.81.133:2380 \
--initial-advertise-peer-urls=http://192.168.81.133:2380 \
--initial-cluster-token=etcd-cluster \
--initial-cluster etcd-01=http://192.168.81.131:2380,etcd-02=http://192.168.81.132:2380,etcd-03=http://192.168.81.133:2380 \
--initial-cluster-state new 
```

查询：

```sh
./etcdctl --endpoints=https://192.168.81.131:2379 endpoint  status --cluster -w table
```

![](./img/etcd-linux-2.png)

#### 3.3.2 安装 ServiceCenter

> 确保这边获取的是SC，2.1.0版本的包。

![](./img/sc-conf.png)

如上图，包解压后包含的一些内容，首先需要配置修改 conf 目录中的文件，启动 `start-service-center.sh` 和 `start-frontend.sh`

---

修改`conf`中的文件

`app.conf`

![](./img/app-conf.png)

修改 frontend_host_ip 和 httpaddr，为本级的 ip 地址。

`app.yaml`

![](./img/server-host.png)

![](./img/app-yaml.png)

修改 

1.server.host

2.REGISTRY_KIND

3.REGISTRY_ETCD_CLUSTER_NAME

4.REGISTRY_ETCD_CLUSTER_MANAGER_ENDPOINTS

5.REGISTRY_ETCD_CLUSTER_ENDPOINTS

`chassis.yaml`

![](./img/chassis.png)

修改 listenAddress 为本机的 ip 地址。

`syncer.yaml`

![](./img/syncer.png)

开启 enableOnStart 开关，以及修改endpoints，region-2中的sc机子ip。

**重复上述操作，去修改其他机子上的 sc 的配置**。

然后在各个机子上运行

`start-service-center.sh` 和 `start-frontend.sh`

打开随意一个节点的前端界面：

http://192.168.81.128:30103/

![](./img/front-1.png)

![](./img/front-2.png)

### 3.4 安装后验证

curl -k http://192.168.81.128:30100/health

![](./img/health.png)

![](./img/register-service.png)

![](./img/front-3.png)

![](./img/front-4.png)

如上图所示，请求创建一个服务，创建成功后以及同步到另外一个 region 了。

### 3.5 使用Sermant

这边准备好backend，和zookeeper。

![](./img/sermant-prepare.png)

Sermant可以自己编包，或者去 [release](https://github.com/huaweicloud/Sermant/releases) 边下载。

zk[下载地址](http://archive.apache.org/dist/zookeeper/)。

![](./img/sermant-tar.png)



#### 3.5.1 文档资料

https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/document.md

 https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/dubbo-register-migiration.md

#### 3.5.2 对接Nacos

**前提**: 

1.已经部署好了nacos

2.编译好了 [demo 应用](https://github.com/huaweicloud/Sermant/tree/develop/sermant-plugins/sermant-register-center/demo-register/demo-register-dubbo)

> 这边可查看这个文档
>
> https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/dubbo-register-migiration.md

---

步骤一: 部署 provider

```java
java -Ddubbo.registry.address=nacos://127.0.0.1:8848 -jar dubbo-provider.jar
```

![](./img/nacos-provider.png)

![](./img/nacos-provider-front.png)

步骤二：部署consumer

```java
java -Ddubbo.registry.address=nacos://127.0.0.1:8848 -jar dubbo-consumer.jar
```

![](./img/nacos-consumer.png)

![](./img/nacos-consumer-front.png)

步骤三：验证

```http
http://192.168.81.128:28020/test
```

![](./img/nacos-http.png)

#### 3.5.3 对接 sc

修改配置地址

![](./img/config.png)

**步骤一**：部署provider

```java
java -javaagent:/root/sermant/sermant-agent/agent/sermant-agent.jar=appName=dubbo-provider -jar dubbo-provider.jar
```

**步骤二**：部署consumer

```java
java -javaagent:/root/sermant/sermant-agent/agent/sermant-agent.jar=appName=dubbo-provider -jar dubbo-consumer.jar 
```

**步骤三**：测试

```http
http://192.168.81.128:28020/test
```

![](./img/test-sc.png)

![](./img/sc-128.png)

![](./img/sc-133.png)

已经同步到另外一个region了。

#### 3.5.2 双注册

详细见文档：https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/dubbo-register-migiration.md

修改`${agent_package_path}/agent/pluginPackage/register-center/config/config.yaml`

![](./img/sermant-config.png)

**步骤一**：注册provider

```java
java -Ddubbo.registry.address=nacos://192.168.81.128:8848 -javaagent:/root/sermant/sermant-agent/agent/sermant-agent.jar=appName=dubbo-provider -jar dubbo-provider.jar 
```

**步骤二**：注册consumer

```java
java -Ddubbo.registry.address=nacos://192.168.81.128:8848 -javaagent:/root/sermant/sermant-agent/agent/sermant-agent.jar=appName=dubbo-consumer -jar dubbo-consumer.jar 
```

**步骤三**：验证

```http
http://192.168.81.128:28020/test
```

![](./img/nacos-front.png)

![](./img/nacos-c-p-front.png)

![](./img/sc-128-front.png)

![](./img/sc-133-front.png)

如上图所示已经同步成功。
