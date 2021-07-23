## RocketMQ 配置启动

&nbsp;

> Java 环境搭建，这里不赘述，请参考其它专题

&nbsp;

### 切换用户和安装路径

```bash
$ su user
$ cd /path
```

&nbsp;

### Name Server 配置

```bash
$ vim conf/namesrv.properties
```

```bash
listenPort=9876
serverWorkerThreads=8
serverCallbackExecutorThreads=0
serverSelectorThread=3
serverOnewaySemaphoreValue=256
serverAsyncSemaphoreValue=256
serverChannelMaxIdleTimeSeconds=120
serverSocketSndBufsize=2048
serverSocketRcvBufSize=1024
serverPooledByteBufAllocatorEnable=false
```

> 有需要可对其参数进行优化配置（前提：了解透彻）

&nbsp;

### 启动 Name Server 

当所有机器配置完成后（scp, 这不是重点，不再阐述）， 启动 nameservice 进程

```bash
$ nohup bin/mqnamesrv -c conf/namesrv.properties &
```

&nbsp;

### 查看 Name Server 日志

```bash
cd logs/rocketmqlogs
```

&nbsp;

## Broker 配置

### 切换用户和目录

```bash
$ su user
$ cd /path
```

&nbsp;

### Broker 配置

> 两 master 搭建方式

```bash
$ vim conf/broker.properties
```

```bash
brokerClusterName=cluster_name
brokerName=broker-1

namesrvAddr=ip1:9876,ip2:9876
defaultTopicQueueNums=8
autoCreateTopicEnable=true
autoCreateSubscriptionGroup=true
listenPort=9876
deleteWhen=04
fileReservedTime=24
mapedFileSizeCommitLog=1073741824
mapedFileSizeConsumeQueue=8000000
destroyMapedFileIntervalForcibly=120000
redeleteHangedFileInterval=120000
diskMaxUsedSpaceRatio=88
storePathRootDir=/path/data/sotre
storePathCommitLog=/path/data/store/commit-log
maxMessageSize=65536
flushCommitLogLeastPages=4
flushConsumeQueueLeastPages=2
flushCommitLogThoroughInterval=10000
flushConsumeQueueThoroughInterval=60000
sendMessageThreadPoolNums=128
pullMessageThreadPoolNums=128
brokerRole=ASYNC_MASTER
flushDiskType=ASYNC_FLUSH
```

> 有需要可对其参数进行优化配置（前提：了解透彻）

&nbsp;

### 启动 Broker

当所有机器配置完成后（scp, 这不是重点，不再阐述）， 启动 nameservice 进程

```bash
## 配置中如果有 Name Server 信息， 可以不指定 -n ip1:9876,直接使用配置文件启动即可
$ nohup bin/mqbroker -c conf/broker.properties &
```

&nbsp;

### 查看 Broker 日志

```bash
$ cd logs/rocketmqlogs
```

