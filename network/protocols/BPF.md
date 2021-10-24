# primitives 原语

由名称或数字，以及描述它的多个限定词组成

- qualifiers 限定词
- Type：设置数字或者名称所指示类型，例如 host www.baidu.com
- Dir：设置网络出入方向，例如 dst port 80
- Proto：指定协议类型，例如 udp
- 其他

# 原语运算符

- 与：&& 或者 and
- 或：|| 或者 or
- 非：! 或者 not  

# 限定词  

## Type

设置数字或者名称所指示类型

- host、port
- net ，设定子网，net 192.168.0.0 mask 255.255.255.0 等价于 net 192.168.0.0/24
- portrange，设置端口范围，例如 portrange 6000-8000  

## Dir

设置网络出入方向

- src、dst、src or dst、src and dst
- ra、ta、addr1、addr2、addr3、addr4（仅对 IEEE 802.11 Wireless LAN 有效）  

## Proto

指定协议类型

- ether、fddi、tr、 wlan、 ip、 ip6、 arp、 rarp、 decnet、 tcp、udp、icmp、igmp、icmp、
  igrp、pim、ah、esp、vrrp  

## 其他

- gateway：指明网关 IP 地址，等价于 ether host ehost and not host host
- broadcast：广播报文，例如 ether broadcast 或者 ip broadcast
- multicast：多播报文，例如 ip multicast 或者 ip6 multicast
- less, greater：小于或者大于  