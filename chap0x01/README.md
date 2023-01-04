# 基于 VirtualBox 的网络攻防基础环境搭建
### 实验目的

### 实验环境
### 实验要求
- [x] 靶机可以直接访问攻击者主机
- [x] 攻击者主机无法直接访问靶机
- [x] 网关可以直接访问攻击者主机和靶机
- [x] 靶机的所有对外上下行流量必须经过网关
- [x] 所有节点均可以访问互联网
### 实验步骤
#### 1 虚拟机实现多重加载
- 管理->虚拟介质管理->虚拟硬盘
- 选中导入的vdi文件后，类型->多重加载
    ![多重加载](./img/1%20%E8%99%9A%E6%8B%9F%E6%9C%BA%E5%AE%9E%E7%8E%B0%E5%A4%9A%E9%87%8D%E5%8A%A0%E8%BD%BD.png)
#### 2 配置拓扑环境所需的各网络
- 搭建如下网络拓扑
  ![网络拓扑](./img/30%20%E7%BD%91%E7%BB%9C%E6%8B%93%E6%89%91%E5%9B%BE.jpg)
##### 2.1 Gateway_Debian网关配置
- 网卡1：NAT网络
- 网卡2：Host-only网络
- 内部网络：intnet0
- 内部网络：intnet1
    ![网关配置](./img/2%20%E7%BD%91%E5%85%B3%E9%85%8D%E7%BD%AE%E5%9B%9B%E5%9D%97%E7%BD%91%E5%8D%A1.png)
- 四块网卡地址如下图
    ![网卡地址](./img/3%20%E7%BD%91%E5%85%B3%E5%9B%9B%E5%9D%97%E7%BD%91%E5%8D%A1%E5%9C%B0%E5%9D%80.png)
##### 2.2 Attacker_Kali攻击者配置
- 只需配置一块网卡，设置为NAT网络
    ![攻击者配置](./img/4%20%E6%94%BB%E5%87%BB%E8%80%85%E9%85%8D%E7%BD%AE.png)
- 和网关的一个端口在同一网段
    ![攻击者网段](./img/5%20%E6%94%BB%E5%87%BB%E8%80%85%E7%BD%91%E7%BB%9C%E5%9C%B0%E5%9D%80.png)
##### 2.3 内部网络intnet0中靶机Victim_XP_1配置
- 配置内部网络intnet0
    ![内网0中xp靶机配置](./img/6%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C0%E4%B8%ADwin7%E9%9D%B6%E6%9C%BA.png)
- 查看ip地址
    ![靶机ip地址](./img/7%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C0%E4%B8%ADwin7%E9%9D%B6%E6%9C%BA%E5%9C%B0%E5%9D%80.png)
##### 2.4 内部网络intnet0中靶机Victim_Kali_1配置
- 配置内部网络intnet0
    ![内网0中kali靶机配置](./img/8%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C0%E4%B8%ADkali%E9%9D%B6%E6%9C%BA.png)
- 查看ip地址
    ![靶机ip地址](./img/9%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C0%E4%B8%ADkali%E9%9D%B6%E6%9C%BA%E5%9C%B0%E5%9D%80.png)
  
##### 2.5 内部网络intnet1中靶机Victim_XP_2配置
- 配置内部网络intnet1
    ![内网1中xp靶机配置](./img/10%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C1%E4%B8%ADwin7%E9%9D%B6%E6%9C%BA.png)
    ![靶机ip地址](./img/11%20%E5%86%85%E9%83%A8%E7%BD%91%E7%BB%9C1%E4%B8%ADwin7%E9%9D%B6%E6%9C%BA%E5%9C%B0%E5%9D%80.png)
  
#### 3 连通性测试
  |节点|ip地址|
  |----|----|
  |Attacker_Kali|10.0.2.4|
  |Victim_XP_1|172.16.111.139|
  |Victim_Kali_1|172.16.111.136|
  |Victim_XP_2|172.16.222.122|

##### 3.1 靶机可以直接访问攻击者主机
- 内部网络0中靶机Victim_XP_1访问攻击者Attacker_Kali主机
  ![Victim_XP_1访问攻击者](./img/12%20%E5%86%85%E7%BD%910%E9%9D%B6%E6%9C%BAwin7%E8%AE%BF%E9%97%AE%E6%94%BB%E5%87%BB%E8%80%85.png)
- 内部网络0中靶机Victim_Kali_1访问攻击者Attacker_Kali主机
  ![Victim_Kali_1访问攻击者](./img/13%20%E5%86%85%E7%BD%910%E9%9D%B6%E6%9C%BAkali%E8%AE%BF%E9%97%AE%E6%94%BB%E5%87%BB%E8%80%85.png)
- 内部网络1中靶机Victim_XP_2访问攻击者Attacker_Kali主机
  ![](./img/14%20%E5%86%85%E7%BD%911%E9%9D%B6%E6%9C%BAxp%E8%AE%BF%E9%97%AE%E6%94%BB%E5%87%BB%E8%80%85.png)
##### 3.2 攻击者主机无法直接访问靶机
- 攻击者Attacker_Kali无法访问靶机Victim_XP_1
  ![攻击者无法访问XP_1](./img/15%20%E6%94%BB%E5%87%BB%E8%80%85%E6%97%A0%E6%B3%95%E8%AE%BF%E9%97%AExp_1.png)
- 攻击者Attacker_Kali无法访问靶机Victim_Kali_1
  ![攻击者无法访问Kali_1](./img/16%20%E6%94%BB%E5%87%BB%E8%80%85%E6%97%A0%E6%B3%95%E8%AE%BF%E9%97%AEkali_1.png)
- 攻击者Attacker_Kali无法访问靶机Victim_XP_2
  ![攻击者无法访问XP_2](./img/17%20%E6%94%BB%E5%87%BB%E8%80%85%E6%97%A0%E6%B3%95%E8%AE%BF%E9%97%AExp_2.png)
##### 3.3 网关可以直接访问攻击者主机和靶机
- 网关Gateway_Debian可以直接访问攻击者Attacker_Kali
  ![网关可以访问攻击者](./img/18%20%E7%BD%91%E5%85%B3%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AE%E6%94%BB%E5%87%BB%E8%80%85.png)
- 网关Gateway_Debian可以直接访问靶机Victim_XP_1
  ![网关可以访问xp_1](./img/19%20%E7%BD%91%E5%85%B3%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AExp_1.png)
- 网关Gateway_Debian可以直接访问靶机Victim_Kali_1
  ![网关可以访问kali_1](./img/20%20%E7%BD%91%E5%85%B3%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AEkali_1.png)
  ![网关可以访问xp_2](./img/21%20%E7%BD%91%E5%85%B3%E5%8F%AF%E4%BB%A5%E8%AE%BF%E9%97%AExp_2.png)
##### 3.4 靶机的所有对外上下行流量必须经过网关
- `tcpdump -i enp0s9` 监控 `内部网络0` 中流量
  - 靶机Victim_XP_1对外流量经过网关
    ![xp_1流量经过网关](./img/22%20xp_1%E6%B5%81%E9%87%8F%E7%BB%8F%E8%BF%87%E7%BD%91%E5%85%B3.png)
  - 靶机Victim_Kali_1对外流量经过网关
    ![kali_1流量经过网关](./img/24%20kali_1%E6%B5%81%E9%87%8F%E7%BB%8F%E8%BF%87%E7%BD%91%E5%85%B3.png)
- `tcpdump -i enp0s10` 监控 `内部网络1` 中流量
  - 靶机Victim_XP_2对外流量经过网关
    ![xp_2流量经过网关](./img/23%20xp_2%E6%B5%81%E9%87%8F%E7%BB%8F%E8%BF%87%E7%BD%91%E5%85%B3.png)
##### 3.5 所有节点均可以访问互联网
- 网关Gateway_Debian访问互联网
  ![网关访问互联网](./img/25%20%E7%BD%91%E5%85%B3%E8%AE%BF%E9%97%AE%E4%BA%92%E8%81%94%E7%BD%91.png)
- 靶机Victim_xp_1访问互联网
  ![xp_1访问互联网](./img/26%20xp_1%E8%AE%BF%E9%97%AE%E4%BA%92%E8%81%94%E7%BD%91.png)
- 靶机Victim_Kali_1访问互联网
  ![kali_1访问互联网](./img/27%20kali_1%E8%AE%BF%E9%97%AE%E4%BA%92%E8%81%94%E7%BD%91.png)
- 靶机Victim_xp_2访问互联网
  ![xp_2访问互联网](./img/28%20xp_2%E8%AE%BF%E9%97%AE%E4%BA%92%E8%81%94%E7%BD%91.png)
- 攻击者Attacker_Kali访问互联网
  ![攻击者访问互联网](./img/29%20%E6%94%BB%E5%87%BB%E8%80%85%E8%AE%BF%E9%97%AE%E4%BA%92%E8%81%94%E7%BD%91.png)
### 参考资料
- [单向能ping通，反向不通故障解决过程](https://blog.csdn.net/wj31932/article/details/89634302)
- [虚拟硬盘多重加载](https://www.expoli.tech/articles/2021/06/07/1623066136894)
- [配置Debian允许root用户SSH登录](https://www.cnblogs.com/pengpengboshi/p/16042972.html)