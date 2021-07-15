## LVS

`Linux Virtual Server` 的缩写，是一个虚拟的服务器集群系统。 1998 年章文嵩博士成立，是中国国内最早出现的自由软件项目之一。 4 层协议

&nbsp;

在这里普及个知识： 

ARP : 地址解析协议， （`Address Resolution Protocol`）, 是根据 IP 地址获取物理地址的一个 TCP/IP 协议。 

```bash
$ cd /proc/sys/net/ipv4/conf/*IF*/
# /proc/sys/net/ipv4/conf/*IF*/ 从内存映射的文件
# ipv4 协议
```

&nbsp;

上面文件中有以下 2 个文件

arp 的两个参数

- arp_ignore : 
  - 定义接收到 ARP 请求时的响应级别；
    - `0`: 只要本地配置的有相应地址，就给与相应。
    - `1`:  仅在请求的目标（MAC）地址配置请求到达接口上的时候，才给与相应；

- arp_announce : 定义将自己地址向外通告时的通告级别
  - `0` -> 将本地所有接口上的地址向外通告
    - 本机网卡上的所有 ip 地址都向外通告
      - 公共 ip
      - 内部 ip
      - 所有网卡
  - `1` -> 试图仅向目标网络通告预期网络匹配的地址
    - 公共 ip 向外通告
  - `2` -> 仅向与本地接口上地址匹配的网络进行通告
    - 匹配上的

&nbsp;

### 负载均衡调度算法：

首先一个前置知识点： 负载均衡器需要具备 “偷窥” 能力， 才能知道 RIP 的信息。 “只看不动手”

记录 Client 与 Server 信息



&nbsp;

### IPVS 内核模块

Linux 根据 LVS 技术封装在系统的内置内核模块， 它不需要安装

&nbsp;

但是需要安装另外一个和外界交互的 ipvsadm ， 它仅仅是传递者

```bash
$ yum install ipvsadm -y
```

&nbsp;

#### **管理集群服务**

添加： 

- -A : 添加 
  - `-t` :  TCP 协议的集群
  - `-u`: UDP 协议的集群
  - `server-address` :  `ip:port`
  - `-s scheduler` 
    - 负载均衡调度算法
    - 详情见前面内容

`-E` ：修改

`-D`:    删除

&nbsp;

`-t|u|f service-address [-s scheduler]`

e.g. 

```bash
$ ipvsadm -A -t 192.168.1.100:80 -s rr
```

&nbsp;

#### **添加集群服务中的 RS** 

- -a ： 添加
  - `-a -t|u|f service-address -r server-address [-g|i|m] [-w weight]`
  - `-t|u|f service-address` : 事先定义好的某集群服务
  - `-r server-address` : 某 RS 的地址，在 NAT 模型中，可使用 `ip:port` 事先端口映射。
  - `[-g|i|m]`
    - `-g` ： DR 
    - `-i` : TUN
    - `-m` : NAT
    - [-w weight] : 定义服务器权重

```bash
$ ipvsadm -a -t ip1:port1 -r ip2:port2 -g
```

&nbsp;

- `-e` : 修改
- `-d` : 删除
- `-l|L` : 查看
- `-n` : 数字格式显示主机地址和端口
- `--stats` : 统计数据
- `--rate` : 速率
- `--timeout` : 显示 tcp, tcpfin 和 udp 的会话超时时长
- `-:c` : 显示当前的 ipvs 连接状况
  - lvs 不和 RS  连接, 只“偷窥”

删除所有集群服务

- `-C` : 清空 ipvs 规则
- `-S` : 保存

```bash
$ ipvsadm -S > /path/to/somefile
```

载入此前的规则： 

- `-R `

```bash
$ ipvsadm -R < /path/to/somefile
```

&nbsp;

- 









&nbsp;



