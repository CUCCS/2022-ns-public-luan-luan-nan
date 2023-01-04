# 实验九 入侵检测
### 实验要求
- [x] 使用 Snort、Suricata 和 Guardian 工具体验入侵检测
- [x] 使用 Suricata 代替 Snort ，重复本实验
### 实验环境
![实验拓扑图](./img/0%20%E6%8B%93%E6%89%91%E5%9B%BE.jpg)
#### Snort安装
```
# 禁止在apt安装时弹出交互式配置界面
export DEBIAN_FRONTEND=noninteractive

apt install snort
```
安装时报错，查找资料后添加源再进行安装
```
# 进入列表
sudo vim /etc/apt/sources.list

#添加源
deb http://http.kali.org/kali kali-rolling main non-free contrib
deb http://http.kali.org/kali sana main non-free contrib
deb http://security.kali.org/kali-security sana/updates main contrib non-free
deb http://old.kali.org/kali moto main non-free contrib
```
成功安装snort,进入欢迎界面
![](./img/1%20%E5%AE%89%E8%A3%85snort%E6%88%90%E5%8A%9F.png)
#### Suricata
```
sudo apt-get install suricata
```
### 实验流程
#### 实验一：配置snort为嗅探模式
##### Snort
```
# -q 静默操作，不显示版本欢迎信息和初始化信息
snort -q -v -i eth0
```
victim_kali_1开启snort，攻击者`ping` victim_kali_1 , victim_kali_1显示相关信息
![嗅探模式](./img/2%20%E5%97%85%E6%8E%A2.png)
##### Suricata
```
suricata -v -i eth0
```
victim_kali_1开启snort，攻击者`ping` victim_kali_1 , victim_kali_1显示相关信息
![suricata](./img/3%20suricata.png)
#### 实验二：配置并启用snort内置规则
##### Snort
```
# /etc/snort/snort.conf 中的 HOME_NET 和 EXTERNAL_NET 需要正确定义
# 例如，学习实验目的，可以将上述两个变量值均设置为 any

```
![配置规则](./img/4%20%E9%85%8D%E7%BD%AE%E7%8E%AF%E5%A2%83.png)
```
# 启用内置规则嗅探
snort -q -A console -b -P 65535 -i eth0 -c /etc/snort/snort.conf -l /var/log/snort/
```
![应用规则](./img/9%20%E5%86%85%E7%BD%AE%E8%A7%84%E5%88%99%E5%97%85%E6%8E%A2.png)
##### Suricata
```
sudo suricata -i eth0 -c /etc/suricata/suricata.yaml
```
![suricata](./img/5%20suricata.png)
#### 实验三：自定义snort规则
##### Snort
```
# 新建自定义 snort 规则文件
sudo vim /etc/snort/rules/cnss.rules

# INSERT
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Access Violation has been detected on /etc/passwd ";flags: A+; content:"/etc/passwd"; nocase;sid:1000001; rev:1;)
alert tcp $EXTERNAL_NET any -> $HTTP_SERVERS 80 (msg:"Possible too many connections toward my http server"; threshold:type threshold, track by_src, count 100, seconds 2; classtype:attempted-dos; sid:1000002; rev:1;)

# 添加配置代码到 /etc/snort/snort.conf
include $RULE_PATH/cnss.rules

```
![配置规则](./img/6%20%E9%85%8D%E7%BD%AE%E8%A7%84%E5%88%99.png)
![添加规则](./img/7%20%E6%B7%BB%E5%8A%A0%E8%A7%84%E5%88%99%20.png)
```
# 在攻击者处进行压力测试
ping -s 900 172.16.111.136
```
![压力测试](./img/8%20%E5%8E%8B%E5%8A%9B%E6%B5%8B%E8%AF%95.png)
```
# 应用规则开启嗅探
snort -q -A console -b -P 65535 -i eth0 -c /etc/snort/snort.conf
```
![发出警报](./img/10%20%E5%8F%91%E5%87%BA%E8%AD%A6%E6%8A%A5.png)
##### Suricata
```
cat << EOF > /etc/suricata/rules/test.rules
alert icmp 172.16.111.141 any <> 172.16.111.136 any (msg:"Informing ICMP Packet from 172.16.111.141";sid:1000001;rev:1;)
EOF
```
![配置规则](./img/12%20suricata%E9%85%8D%E7%BD%AE%E8%A7%84%E5%88%99.png)
```
# 添加配置代码到 /etc/suricata/suricata.yaml
 - test.rules
```
![添加规则](./img/13%20suricata%E6%B7%BB%E5%8A%A0%E8%A7%84%E5%88%99.png)
```
# 应用规则
suricata -i eth0 -c /etc/suricata/suricata.yaml 
```
![应用规则](./img/14%20%E5%BA%94%E7%94%A8%E8%A7%84%E5%88%99.png)
攻击者`ping` victim_kali_1 , victim_kali_1出现警报
![出现警报](./img/15%20%E8%AD%A6%E6%8A%A5.png)
#### 实验四：和防火墙联动
在 `victim_kali_1` 上：
```
# 获取脚本代码
wget https://c4pr1c3.github.io/cuc-ns/chap0x09/attach/guardian.tar.gz
# 解压缩 Guardian-1.7.tar.gz
tar zxf guardian.tar.gz
```
![下载并解压](./img/16%20%E4%B8%8B%E8%BD%BD%E5%B9%B6%E8%A7%A3%E5%8E%8B.png)
进入配置文件
![进入配置文件](./img/17%20%E8%BF%9B%E5%85%A5%E9%85%8D%E7%BD%AE%E7%95%8C%E9%9D%A2.png)
编辑`guardian.conf`并保存
![编辑配置文件](./img/18%20%E7%BC%96%E8%BE%91%E9%85%8D%E7%BD%AE.png)

```
# 启动 guardian.pl
perl guardian.pl -c guardian.conf
```
![启动 guardian](./img/19%20%E5%90%AF%E5%8A%A8guardian.png)
```
# 攻击者主机对靶机进行nmap扫描
nmap 172.16.111.136 -A -T4 -n -vv
```
![暴力扫描](./img/20%20%E5%BC%80%E5%90%AF%E6%9A%B4%E5%8A%9B%E6%89%AB%E6%8F%8F.png)
![扫描](./img/21%20%E6%89%AB%E6%8F%8F.png)

查看此时防火墙的链，guardian.conf 中默认的来源IP被屏蔽时间是 60 秒，在一分钟之后会更新iptable的规则会被删除
```
root@kali:~/home/kali/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination
REJECT     tcp  --  172.16.111.141       0.0.0.0/0            reject-with tcp-reset
DROP       all  --  172.16.111.141       0.0.0.0/0

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
# 1分钟后，guardian.pl 会删除刚才添加的2条 iptables 规则
root@kali:~/home/kali/guardian# iptables -L -n
Chain INPUT (policy ACCEPT)
target     prot opt source               destination

Chain FORWARD (policy ACCEPT)
target     prot opt source               destination

Chain OUTPUT (policy ACCEPT)
target     prot opt source               destination
```
### 实验思考
IDS与防火墙的联动防御方式相比IPS方式防御存在哪些缺陷？是否存在相比较而言的优势？
- IPS属于主动防御，根据设置的过滤器分析相对应的数据包，通过检查的数据包可以继续前进，包含恶意内容的数据包就会被丢弃，被怀疑的数据包需要接受进一步的检查。
- IDS与防火墙联动属于被动防御，检测规则的更新落后于攻击手段的更新。IDS检测而不阻断任何网络行为。
- IPS基于主动响应和过滤功能，可检测到传统的防火墙+IDS 方案检测不到的攻击行为。
  
### 参考资料
- [Snort installation in Kali Linux from the source](https://koayyongcett.medium.com/snort-installation-in-kali-linux-from-the-source-9a005558a2ea)
- [Unable to locate package snort_Kali LInux ](https://unix.stackexchange.com/questions/594935/unable-to-locate-package-snort-kali-linux-in-vmware-workstation-pro-ver-15-5-6)
- [IDS vs. IPS: Definitions, Comparisons](https://www.okta.com/identity-101/ids-vs-ips/)
- [2019-NS-Public-chencwx](https://github.com/CUCCS/2019-NS-Public-chencwx/blob/ns_chap0x09/ns_chapter9/%E5%85%A5%E4%BE%B5%E6%A3%80%E6%B5%8B.md)