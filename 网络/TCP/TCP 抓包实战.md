# tcpdump 和 wireshark

## tcpdump

![](./img/tcpdump.png)

常⽤的过滤表⽤法：

![](./img/tcpdump2.png)

## Wireshark 

Wireshark 除了可以抓包外，还提供了可视化分析⽹络包的图形⻚⾯，同时，还内置了⼀系列的汇总分析⼯具。  

```
tcpdump -i eth1 icmp and host 183.232.231.174 -w ping.pcap
```

⽤ Wireshark  分析：

![](./img/ping_wireshark.png)

在其下⾯的⽹络包详情中， 可以更清楚的看到，这个⽹络包在协议栈各层的详细信息。⽐如，以编号 1 的⽹络包为例⼦：  

![](./img/ping_wireshark2.png)

# 解密 TCP 三次握⼿和四次挥⼿  

```
tcpdump -i any tcp and host 192.168.3.200 and port 80 -w http.cap

curl http://192.168.3.200
```

使⽤ Wireshark 打开 http.pcap：

![](./img/tcp_wireshark.png)

- 最开始的 3 个包就是 TCP 三次握⼿建⽴连接的包
- 中间是 HTTP 请求和响应的包
- ⽽最后的 3 个包则是 TCP 断开连接的挥⼿包  

Wireshark 可以⽤时序图的⽅式显示数据包交互的过程，从菜单栏中，点击 "统计 (Statistics)" -> "流量图 (Flow Graph)"，然后，在弹出的界⾯中的 "流量类型" 选择 "TCP Flows"，你可以更清晰的看到，整个过程中 TCP 流的执⾏过程：  

![](./img/tcp_wireshark2.png)

Wireshark ⼯具帮我们做了优化，它默认显示的是序列号 seq 是相对值，⽽不是真实值。  如果你想看到实际的序列号的值，可以右键菜单， 然后找到 "协议⾸选项"，接着找到Relative Seq后，把它给取消，操作如下：  

![](./img/tcp_wireshark3.png)

取消后， Seq 显示的就是真实值了：  

![](./img/tcp_wireshark4.png)

为什么抓到的 TCP 挥⼿是三次，⽽不是书上说的四次？  

因为服务器端收到客户端的 FIN 后，服务器端同时也要关闭连接，这样就可以把 ACK 和 FIN 合并到⼀起发送，节省了⼀个包，变成了“三次挥⼿”。⽽通常情况下，服务器端收到客户端的 FIN 后，很可能还没发送完数据，所以就会先回复客户端⼀个 ACK包，稍等⼀会⼉，完成所有数据包的发送后，才会发送 FIN 包，这也就是四次挥⼿了。  

# TCP 三次握⼿异常情况分析  

## TCP 第⼀次握⼿ SYN 丢包  

```
date;curl http://192.1682.12.36;date
//@ 阻塞。。。。。。

tcpdump  -i eth0 tcp and host 192.1682.12.36 and port 80 -w tcp_sys_timeout.pcap
```

把 tcp_sys_timeout.pcap ⽂件⽤ Wireshark 打开分析，显示如下图：  

![](./img/tcp_syn_miss4.png)

客户端发起了 SYN 包后，⼀直没有收到服务端的 ACK ，所以⼀直超时重传了 5 次，并且每次RTO 超时时间是不同的：  

- 第⼀次是在 1 秒超时重传
- 第⼆次是在 3 秒超时重传
- 第三次是在 7 秒超时重传
- 第四次是在 15 秒超时重传
- 第五次是在 31 秒超时重传  

可以发现，每次超时时间 RTO 是指数（翻倍）上涨的，当超过最⼤重传次数后，客户端不再发送 SYN 包  

在 Linux 中，第⼀次握⼿的 SYN 超时重传次数，是如下内核参数指定的：  

```
cat /proc/sys/net/ipv4/tcp_syn_retries
```

