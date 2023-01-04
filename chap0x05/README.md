# 实验五 基于 Scapy 编写端口扫描器
### 实验目的
- 掌握网络扫描之端口状态探测的基本原理
### 实验要求
- [x] TCP connect scan / TCP stealth scan
- [x] TCP Xmas scan / TCP fin scan / TCP null scan
- [x] UDP scan
- [x] 上述每种扫描技术的实现测试均需要测试端口状态为：开放、关闭 和 过滤 状态时的程序执行结果
- [x] 提供每一次扫描测试的抓包结果并分析与课本中的扫描方法原理是否相符？如果不同，试分析原因；
- [x] 在实验报告中详细说明实验网络环境拓扑、被测试 IP 的端口状态是如何模拟的
- [x] 复刻 nmap 的上述扫描技术实现的命令行参数开关
### 网络拓扑
![拓扑图](./img/0%20%E6%8B%93%E6%89%91%E5%9B%BE.jpg)
### scapy基础
```
# 导入模块
from scapy.all import *
# 查看包信息
pkt = IP(dst="")
ls(pkt)
pkt.show()
summary(pkt)
# 发送数据包
send(pkt)  # 发送第三层数据包，但不会受到返回的结果。
sr(pkt)  # 发送第三层数据包，返回两个结果，分别是接收到响应的数据包和未收到响应的数据包。
sr1(pkt)  # 发送第三层数据包，仅仅返回接收到响应的数据包。
sendp(pkt)  # 发送第二层数据包。
srp(pkt)  # 发送第二层数据包，并等待响应。
srp1(pkt)  # 发送第二层数据包，并返回响应的数据包
# 监听网卡
sniff(iface="wlan1",count=100,filter="tcp")
# 应用：简单的SYN端口扫描 （测试中）
pkt = IP("...")/TCP(dport=[n for n in range(22, 3389)], flags="S")
ans, uans = sr(pkt)
ans.summary() # flag为SA表示开放，RA表示关闭
```
### 实验过程
#### TCP connect scan
先发送一个`S`，然后等待回应。如果有回应且标识为`RA`，说明目标端口处于关闭状态；如果有回应且标识为`SA`，说明目标端口处于开放状态。这时`TCP connect scan`会回复一个`RA`，在完成三次握手的同时断开连接.
##### code
```
from scapy.all import *


def tcpconnect(dst_ip, dst_port, timeout=10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port,flags="S"),timeout=timeout)
    if pkts is None:
        print("Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x12):  #Flags: 0x012 (SYN, ACK)
            send_rst = sr(IP(dst=dst_ip)/TCP(dport=dst_port,flags="AR"),timeout=timeout)
            print("Open")
        elif (pkts.getlayer(TCP).flags == 0x14):   #Flags: 0x014 (RST, ACK)
            print("Closed")

tcpconnect('172.16.111.136', 80)
```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/1%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/2%20%E6%8A%93%E5%8C%85.png)
  ```
  # nmap 复刻
  nmap -sT -p 80 172.16.111.136
  ```
![复刻](./img/3%20%E5%A4%8D%E5%88%BB.png)
![复刻抓包](./img/4%20%E5%A4%8D%E5%88%BB%E6%8A%93%E5%8C%85.png)

##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 80/tcp
```
![端口开放](./img/5%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/6%20%E6%8A%93%E5%8C%85.png)
```
  # nmap 复刻
  nmap -sT -p 80 172.16.111.136
```
![复刻](./img/7%20%E5%A4%8D%E5%88%BB.png)
![复刻抓包](./img/8%20%E5%A4%8D%E5%88%BB%E6%8A%93%E5%8C%85.png)

##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 80/tcp
```
![端口过滤](./img/9%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/10%20%E6%8A%93%E5%8C%85.png)
```
  # nmap 复刻
  nmap -sT -p 80 172.16.111.136
```
![复刻](./img/11%20%E5%A4%8D%E5%88%BB.png)

#### TCP stealth scan
先发送一个`S`，然后等待回应。如果有回应且标识为`RA`，说明目标端口处于关闭状态；如果有回应且标识为`SA`，说明目标端口处于开放状态。这时`TCP stealth scan`只回复一个`R`，不完成三次握手，直接取消建立连接。
##### code
```
from scapy.all import *


def tcpstealthscan(dst_ip, dst_port, timeout=10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port, flags="S"), timeout=10)
    if (pkts is None):
        print("Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x12):
            send_rst = sr(IP(dst=dst_ip) /
                          TCP(dport=dst_port, flags="R"), timeout=10)
            print("Open")
        elif (pkts.getlayer(TCP).flags == 0x14):
            print("Closed")
        elif(pkts.haslayer(ICMP)):
            if(int(pkts.getlayer(ICMP).type) == 3 and int(stealth_scan_resp.getlayer(ICMP).code) in [1, 2, 3, 9, 10, 13]):
                print("Filtered")


tcpstealthscan('172.16.111.136', 80)

```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/14%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/15%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sS -p 80 172.16.111.136
```
![复刻](./img/12%20%E5%A4%8D%E5%88%BB.png)
![抓包](./img/13%20%E5%A4%8D%E5%88%BB%E6%8A%93%E5%8C%85.png)
##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 80/tcp
```
![端口开放](./img/16%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/17%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sS -p 80 172.16.111.136
```
![复刻](./img/18%20%E5%A4%8D%E5%88%BB.png)
##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 80/tcp
```
![端口过滤](./img/19%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/20%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sS -p 80 172.16.111.136
```
![复刻](./img/21%20%E5%A4%8D%E5%88%BB.png)
#### TCP Xmas scan
一种隐蔽性扫描，当处于端口处于关闭状态时，会回复一个`RST`包；其余所有状态都将不回复。
##### code
```
from scapy.all import *


def Xmasscan(dst_ip, dst_port, timeout=10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port, flags="FPU"), timeout=10)
    if (pkts is None):
        print("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type) == 3 and int(pkts.getlayer(ICMP).code) in [1, 2, 3, 9, 10, 13]):
            print("Filtered")


Xmasscan('172.16.111.136', 80)
```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/22%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/23%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sX -p 80 172.16.111.136
```
![复刻](./img/24%20%E5%A4%8D%E5%88%BB.png)
##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 80/tcp
```
![端口开放](./img/25%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/26%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sX -p 80 172.16.111.136
```
![复刻](./img/27%20%E5%A4%8D%E5%88%BB.png)
##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 80/tcp
```
![端口过滤](./img/28%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/29%20%E6%8A%93%E5%8C%85.png)
```
# nmap 复刻
sudo nmap -sX -p 80 172.16.111.136
```
![复刻](./img/30%20%E5%A4%8D%E5%88%BB.png)

#### TCP fin scan
仅发送`FIN`包，`FIN`数据包能够通过只监测`SYN`包的包过滤器，隐蔽性较`SYN`扫描更⾼，此扫描与`Xmas`扫描也较为相似，只是发送的包未`FIN`包，同理，收到`RST`包说明端口处于关闭状态；反之说明为开启/过滤状态。
##### code
```
from scapy.all import *


def finscan(dst_ip, dst_port, timeout=10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port, flags="F"), timeout=10)
    if (pkts is None):
        print("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type) == 3 and int(pkts.getlayer(ICMP).code) in [1, 2, 3, 9, 10, 13]):
            print("Filtered")


finscan('172.16.111.136', 80)
```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/31%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/32%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sF -p 80 172.16.111.136
```
![复刻](./img/33%20%E5%A4%8D%E5%88%BB.png)
##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 80/tcp
```
![端口开放](./img/34%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/35%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sF -p 80 172.16.111.136
```
![复刻](./img/36%20%E5%A4%8D%E5%88%BB.png)
##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 80/tcp
```
![端口过滤](./img/37%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/38%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sF -p 80 172.16.111.136
```
![复刻](./img/39%20%E5%A4%8D%E5%88%BB.png)

#### TCP null scan
发送的包中关闭所有`TCP`报⽂头标记，实验结果预期还是同理：收到`RST`包说明端口为关闭状态，未收到包即为开启/过滤状态.
##### code
```
#! /usr/bin/python
from scapy.all import *


def nullscan(dst_ip, dst_port, timeout=10):
    pkts = sr1(IP(dst=dst_ip)/TCP(dport=dst_port, flags=""), timeout=10)
    if (pkts is None):
        print("Open|Filtered")
    elif(pkts.haslayer(TCP)):
        if(pkts.getlayer(TCP).flags == 0x14):
            print("Closed")
    elif(pkts.haslayer(ICMP)):
        if(int(pkts.getlayer(ICMP).type) == 3 and int(pkts.getlayer(ICMP).code) in [1, 2, 3, 9, 10, 13]):
            print("Filtered")


nullscan('172.16.111.136', 80)
```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/40%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/41%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sN -p 80 172.16.111.136
```
![复刻](./img/42%20%E5%A4%8D%E5%88%BB.png)
##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 80/tcp
```
![端口开放](./img/43%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/44%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sN -p 80 172.16.111.136
```
![复刻](./img/45%20%E5%A4%8D%E5%88%BB.png)
##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 80/tcp
```
![端口过滤](./img/46%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/47%20%E6%8A%93%E5%8C%85.png)

```
# nmap复刻
sudo nmap -sN -p 80 172.16.111.136
```
![复刻](./img/48%20%E5%A4%8D%E5%88%BB.png)

#### UDP scan
一种开放式扫描，通过发送`UDP`包进行扫描。当收到`UDP`回复时，该端口为开启状态；否则即为关闭/过滤状态.
##### code
```
from scapy.all import *
def udpscan(dst_ip, dst_port, dst_timeout=10):
    resp = sr1(IP(dst=dst_ip)/UDP(dport=dst_port), timeout=dst_timeout)
    if (resp is None):
        print("Open|Filtered")
    elif (resp.haslayer(UDP)):
        print("Open")
    elif(resp.haslayer(ICMP)):
        if(int(resp.getlayer(ICMP).type) == 3 and int(resp.getlayer(ICMP).code) == 3):
            print("Closed")
        elif(int(resp.getlayer(ICMP).type) == 3 and int(resp.getlayer(ICMP).code) in [1, 2, 9, 10, 13]):
            print("Filtered")
        elif(resp.haslayer(IP) and resp.getlayer(IP).proto == IP_PROTOS.udp):
            print("Open")
udpscan('172.16.111.136', 53)
```
##### 端口关闭
```
# 端口关闭
sudo ufw disable
```
![端口关闭](./img/49%20%E7%AB%AF%E5%8F%A3%E5%85%B3%E9%97%AD.png)
![抓包](./img/50%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sU -p 53 172.16.111.136
```
![复刻](./img/51%20%E5%A4%8D%E5%88%BB.png)
##### 端口开放
```
# 端口开放
sudo ufw enable && sudo ufw allow 53/tcp
```
![端口开放](./img/52%20%E7%AB%AF%E5%8F%A3%E5%BC%80%E6%94%BE.png)
![抓包](./img/53%20%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sU -p 53 172.16.111.136
```
![复刻](./img/54%20%E5%A4%8D%E5%88%BB.png)
##### 端口过滤
```
# 端口过滤
sudo ufw enable && sudo ufw deny 53/tcp
```
![端口过滤](./img/55%20%E7%AB%AF%E5%8F%A3%E8%BF%87%E6%BB%A4.png)
![抓包](./img/8%20%E5%A4%8D%E5%88%BB%E6%8A%93%E5%8C%85.png)
```
# nmap复刻
sudo nmap -sU -p 53 172.16.111.136
```
![复刻](./img/57%20%E5%A4%8D%E5%88%BB.png)
### 实验总结
|扫描方式/端口状态|开放|关闭|过滤|
|---|---|---|---|
|TCP connect / TCP stealth|	完整的三次握手，能抓到ACK&RST包|	只收到一个RST包	|收不到任何TCP包|
|TCP Xmas / TCP FIN / TCP NULL|	收不到TCP回复包	|收到一个RST包	|收不到TCP回复包|
|UDP	|收到UDP回复包|	收不到UDP回复包	|收不到UDP回复包|
### 参考资料
- [实验·网络安全](https://c4pr1c3.github.io/cuc-ns/chap0x05/exp.html)
- [2021-ns-public-Lychee00](https://github.com/CUCCS/2021-ns-public-Lychee00/blob/chap0x05/chap0x05/report05.md#tcp-connect-scan)
- [Scapy2.5.0文档](https://scapy.readthedocs.io/en/latest/)