# 一次诡异的MySQL连接超时排查，让我重新认识了TCP握手

上周五下午，正准备摸鱼等下班，运维突然在群里@我：线上某个服务的错误日志暴涨，全是 `MySQL Connection timed out`。

我第一反应是：数据库挂了？

赶紧连上跳板机看了看数据库监控，CPU、连接数、慢查询都正常。ping了下内网也通。这就奇怪了。

## 重启大法失效

试了重启业务容器，没用。重启数据库？不敢，线上核心库。

看了下业务日志，错误很规律：
[ERROR] 2024-05-20 14:23:15 - com.mysql.jdbc.exceptions.jdbc4.CommunicationsException:
Communications link failure. The last packet sent successfully to the server was 0 milliseconds ago.
The driver has not received any packets from the server.

text

关键信息是：`0 milliseconds ago` —— 这说明连接根本没建立起来。

## 第一直觉：连接池爆了

看应用配置的HikariCP，最大连接数20，数据库那边`show processlist`一看，才接了15个连接。

不对。

试着用`mysql -h xxx -P 3306 -u root -p`手动连接，卡住了。过了大概10秒才返回`ERROR 2003 (HY000): Can't connect to MySQL server on 'xxx' (110)`。

110错误码？查了下是`ETIMEDOUT`，连接超时。

这里有个细节：不是拒绝连接，是超时。说明TCP包发出去后，数据库根本没回。

## 抓包看真相

在业务机器上开tcpdump：

```bash
tcpdump -i eth0 host 数据库IP and port 3306 -nn -S
抓到的东西让我有点懵：

text
14:30:01.123456 IP 10.0.1.100.54321 > 10.0.0.200.3306: Flags [S], seq 123456789, win 29200
14:30:10.234567 IP 10.0.1.100.54321 > 10.0.0.200.3306: Flags [S], seq 123456789, win 29200
14:30:29.345678 IP 10.0.1.100.54321 > 10.0.0.200.3306: Flags [S], seq 123456789, win 29200
TCP三次握手的第一步SYN包发出去后，没有收到SYN+ACK。重试了两次（默认的重试机制），最后放弃。

所以问题是：数据库为什么不回SYN-ACK？

排查数据库端
登录数据库服务器，看/proc/sys/net/ipv4/tcp_syn_retries，默认是5。没问题。

用ss -lnt看了下监听状态：

text
State   Recv-Q  Send-Q   Local Address:Port   Peer Address:Port
LISTEN  1024    0              0.0.0.0:3306          0.0.0.0:*
Recv-Q是1024？正常应该是0。

这里暴露问题了：Recv-Q 不为0，说明全连接队列满了。

解释一下：TCP建立连接分半连接队列（SYN队列）和全连接队列（Accept队列）。ss输出的Recv-Q是全连接队列里等待accept的连接数。Send-Q是队列最大长度。

1024/1024，满了，所以内核直接丢弃新来的SYN包。

为什么全连接队列会满
查了下应用的连接池配置，有个参数被上游哥们在两周前改过：

yaml
maximum-pool-size: 100  # 从20改到了100
但是数据库的max_connections还是120。理论上100 < 120，够用啊。

问题在于：全连接队列的大小是 min(backlog, net.core.somaxconn)。服务端调用listen()时传入的backlog参数，MySQL默认是128。而系统的net.core.somaxconn是128。所以队列最大128。

应用突然从20个连接涨到100，高峰期加上其他服务，瞬间全连接队列就满了。

解决方案
三个方向：

调大net.core.somaxconn到1024

bash
sysctl -w net.core.somaxconn=1024
调整MySQL的backlog（需要重启）
在my.cnf里加：

ini
back_log = 1024
紧急情况先调小应用连接池，让队列消化掉

选了方案1+2，改完后Recv-Q明显降下来了，连接恢复正常。

一点感悟
TCP三次握手的细节，平时觉得“懂了就行”，真踩坑才理解为什么要懂。

还有就是，线上改动一定要走Review。那个改连接池配置的同学没通知任何人，测试环境也没验证过，直接上生产。虽然锅是他背，但事后我们也补了监控：全连接队列使用率超过80%就告警。

这次排查用了半天，其实最开始看ss -lnt的Recv-Q就能定位问题，绕了一大圈。写出来给遇到类似问题的同学省点时间。

最后留个问题：Recv-Q和Send-Q在TCP的不同状态下分别代表什么含义？搞懂这个，很多网络问题都不用瞎猜了。