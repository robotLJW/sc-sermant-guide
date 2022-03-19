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

### 1.2 注册中心插件介绍文档

https://github.com/huaweicloud/Sermant/blob/develop/docs/user-guide/register/document.md

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

进入3 台 etcd 所在目录，执行（不过这一块可以由systemd去维护）

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





#### 3.3.2 安装 ServiceCenter



### 3.4 安装后验证


