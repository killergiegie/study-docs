# Linux-USB无线网卡配置热点

## 0. 查看是否成功识别网卡设备

```bash
ifconfig -a
```
输出如：
```bash
wlxe0b94d135033: flags=4098<BROADCAST,MULTICAST>  mtu 1500
        ether e0:b9:4d:13:50:33  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## 1. 安装工具
```bash
sudo apt install hostapd dnsmasq -y
```
- hostapd: 创建 Wi-Fi 热点
- dnsmasq: 分配 IP 地址


## 2. 分配静态 IP 给 USB 网卡
```bash
sudo ip link set wlxe0b94d135033 up
sudo ip addr add 192.168.50.1/24 dev wlxe0b94d135033
```
- 设定网段为：192.168.50.0，掩码为 255.255.255.0
- Airbox 的 ip 地址为：192.168.50.1
- wlxe0b94d135033 为 USB 网卡设备名
  

## 3. 配置 hostapd 热点服务
### 创建配置文件：
```bash
sudo vi /etc/hostapd/hostapd.conf
```
内容如下：
```bash
interface=wlxe0b94d135033
driver=nl80211
ssid=ServerHotspot
hw_mode=g
channel=6
auth_algs=1
wmm_enabled=0
wpa=2
wpa_passphrase=12345678
wpa_key_mgmt=WPA-PSK
rsn_pairwise=CCMP
```
- interface：指定用于创建热点的网络接口
- driver：指定驱动类型
- ssid：设置热点（Wi-Fi）名称（SSID）
- hw_mode：指定 Wi-Fi 模式为 802.11g（2.4GHz，支持速度可达 54 Mbps）
- channel：设置使用的 Wi-Fi 信道号（频段内的子频率）
- auth_algs：设置认证算法，1 表示只使用开放式系统认证
- wmm_enabled：是否启用 WMM（Wi-Fi Multimedia，提供 QoS 质量保证），设置为 0 表示关闭
- wpa：启用 WPA 安全协议，2 表示使用 WPA2，目前最主流的加密协议（比 WPA 更安全）
- wpa_passphrase：Wi-Fi 密码
- wpa_key_mgmt：指定密钥管理模式，WPA-PSK 表示使用预共享密钥（密码）方式，不需要外部服务器。
- rsn_pairwise：设置 WPA2 使用的加密方式，CCMP 是 WPA2 的标准加密方式（使用 AES 加密）

### 设置 hostapd 启用配置文件
```bash
sudo vi /etc/default/hostapd
```
添加或修改这行：
```bash
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

## 4. 配置 dnsmasq DHCP 服务
备份原文件并新建配置：
```bash
sudo mv /etc/dnsmasq.conf /etc/dnsmasq.conf.bak
sudo vi /etc/dnsmasq.conf
```
输入以下内容：
```bash
interface=wlxe0b94d135033
bind-interfaces
dhcp-range=192.168.50.10,192.168.50.100,12h
dhcp-host=38:7a:cc:40:46:d9,192.168.50.88
```
- interface：指定只监听 USB 无线网卡；
- bind-interfaces：确保只绑定该接口，避免干扰其他网络；
- dhcp-range：分配给连接设备的 IP 范围。
- dhcp-host：配置 maixcam 为固定 ip：192.168.50.88

## 5. 启动服务
```bash
sudo systemctl unmask hostapd    # 解除屏蔽 hostapd 服务
sudo systemctl enable hostapd    # 设置 hostapd 在开机时自动启动
sudo systemctl enable dnsmasq    # 设置 dnsmasq 服务在开机时自动运行

sudo systemctl restart hostapd   # 立即重启 hostapd 服务
sudo systemctl restart dnsmasq   # 重启 dnsmasq，让新的 DHCP 配置立即生效。
```

## 6. 手动关闭、重启服务（如需）
手动关闭服务：
```bash
sudo systemctl stop hostapd
sudo systemctl stop dnsmasq
```
关闭开机自动启动：
```bash
sudo systemctl disable hostapd
sudo systemctl disable dnsmasq
```
如需恢复开机自启：
```bash
sudo systemctl enable hostapd
sudo systemctl enable dnsmasq
```