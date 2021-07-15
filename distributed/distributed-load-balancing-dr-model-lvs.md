### DR 模型实验手册

![load-balancing-lvs-man](/Users/alton/Documents/profile/notebook/Java/play-java/distributed/images/load-balancing-lvs-man.png)

&nbsp;

## 实操

环境准备

### 协议和网络配置

- 在 lvs 服务器的机器上，设置网络层 （192.168.1.101）

```bash
$ ifconfig
# 设置网络层： eth0:n
# /24 = Mask 255.255.255.0
$ ifconfig eth0:1 192.168.1.110/24

# 上述配置方法和下面的等同
# $ ifconfig eth0:1 192.168.1.110 netmask 255.255.255.0
Link encap:Ethernet HWaddr mac address
inet addr:192.168.1.100 Bcast:192.168.1.255 Mask:255.255.255.0
```

&nbsp;

```bash
# 关闭
$ ifconfig eth0:1 down
```

&nbsp;

- Server 机器上，设置不大一样，需要先调协议，然后才能设置网络层。所有 Server 机器一样的配置方式（其它服务机器）

调整协议：

```bash
# 调整协议
$ cd /proc/sys/net/ipv4/conf/eth0
$ cat arp_ignore
0  # 默认级别
# 这里需要修改成 1， 注意不能使用 vi 或 vim 命令，使用它们会报错 
# 'En: Fsync failed'

# 使用 echo 命令来修改
$ echo 1 > arp_ignore

# 同理修改 arp_announce
$ echo 2 > arp_announce
$ cat arp_announce
2 # 0 默认级别

# 为了其它网卡都能生效
$ cd /proc/sys/net/ipv4/conf/all
$ echo 1 > arp_ignore
$ echo 2 > arp_announce
```

&nbsp;

修改网络：

```bash
# 注意 mask 需要 255.255.255.255， 因为实际网络交互时， ip 是需要和 netmask 进行按位 & 操作（路由表）
# 这里的 lo:2 的 2 可以自定义
$ ifconfig lo:2 192.168.1.110 netmask 255.255.255.255
```

&nbsp;

### 搭建 RS 服务器（非 LVS 机器）

```bash
# 首先所有机器安装 httpd 协议,webserver 并启动
$ yum install httpd -y

# 启动
$ service httpd start
```

&nbsp;

在默认主页编辑

```bash
# 以 192.168.1.102 机器为例，加上下列内容
$ vim /var/www/html/index.html
from 192.168.1.102
```

&nbsp;

### LVS  机器服务配置(192.168.1.101)

首先安装 `ipvsadm`

```bash
$ yum install ipvsadm -y
```

&nbsp;

添加入口包

```bash
$ ipvsadm -A -t 192.168.1.110:80 -s rr
## 查看规则
$ ipvsadm -ln

TCP 192.168.1.110:80 rr
```

&nbsp;

添加负载包

```bash
$ ipvsadm -a -t 192.168.1.110:80 -r 192.168.1.102 -g -w 1
$ ipvsadm -a -t 192.168.1.110:80 -r 192.168.1.103 -g -w 1
```

&nbsp;

查看 lvs 入口出口规则信息

```bash
$ ipvsadm -ln
TCP 192.168.1.110:80 rr
-> 192.168.1.102:80 Route 1 0 0
-> 192.168.1.103:80 Route 1 0 0
```

&nbsp;

### 验证 lvs 服务器

游览器访问

- 游览器访问 
  - 192.168.1.110
- 看到负载
  - from 192.168.1.102 或
  - from 192.168.1.103
- 刷新 -> f5 (windows)

&nbsp;

lvs 服务器是看不到刚才验证的活跃连接信息的

102，103 机器可以看到刚才活跃的连接信息

```bash
$ netstat -natp
Active Internet connections (servers and established)
Proto Recv-Q Send-Q Local Address Foreign Address State PID/Program name
```

&nbsp;

上面有说，看不到连接信息，但是可以通过 lvs 查看连接记录(偷窥记录表)

```bash
$ ipvsadm -lnc
pro expire state source virtual destination 
```

&nbsp;

state:

- FIN_WAIT : 代表连接过
- FIN_RECV : lvs 出现此状态记录, 证明 lvs 没事， 一定是后边网络层出问题了

