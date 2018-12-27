作者[smdll]注：本教程仅供学习，不可用于非法用途！

0x00 Arduino IDE环境安装
========================

在<a href="https://www.arduino.cc/en/Main/Software">这里</a>找到Windows的最新版本，下载Installer安装或ZIP解压都可以。

0x01 ESP8266 SDK安装与配置
==========================
#### [环境配置部分]

在IDE里点选“文件”->“首选项”

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-0.PNG)

点选“附加开发板管理器网址”右侧的按钮

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-1.PNG)

输入 http://arduino.esp8266.com/stable/package_esp8266com_index.json 并保存

点选“工具”->“开发板xxxxxx”->“开发板管理器”，等待平台索引更新

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-2.PNG)

翻到页面最下方，找到“esp8266 by ESP8266 Community”，选择版本2.0.0安装（必须是2.0.0！）

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-3.PNG)

------

#### [代码补充部分]

安装完毕后打开文件管理器，输入

%USERPROFILE%\AppData\Local\Arduino15\packages\esp8266\hardware\esp8266\2.0.0\tools\sdk\include （如找不到则把%USERPROFILE%改成当前用户的主文件夹，比如你的账户叫Administrator，就改成C:\Users\Administrator）

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-4.PNG)

修改“user_interface.h”，在文件末尾#endif前添加如下内容
```cpp
typedef void (*freedom_outside_cb_t)(uint8 status);
int wifi_register_send_pkt_freedom_cb(freedom_outside_cb_t cb);
void wifi_unregister_send_pkt_freedom_cb(void);
int wifi_send_pkt_freedom(uint8 *buf, int len, bool sys_seq);
```

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-5.PNG)

保存后重启Arduino IDE，点选“工具”->“开发板xxxxxx”->“NodeMCU 1.0(ESP-12E module)”

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-6.PNG)

0x02 NodeMCU驱动程序
====================
CH340芯片驱动程序在<a href="http://www.wch.cn/download/CH341SER_EXE.html">这里</a>

CP2102芯片驱动程序在<a href="https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers">这里</a>

插入NodeMCU，等待驱动程序安装完成后打开设备管理器，记住使用的端口（图例为CP2102的芯片，COM5）

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/2-0.PNG)

在Arduino IDE里“工具”->“端口”选择对应端口

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/2-1.PNG)

0x03 基础语法
=========
Arduino IDE使用两个入口函数：setup()和loop()，大致语法与C++类似，具体内置函数列表在“帮助”->“参考”。

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/3-0.PNG)

setup()和loop()编译后结构大致如下：
```cpp
int main() {
  setup();
  while(true)
    loop();
}
```

注：微处理器不存在程序运行结束，当程序崩溃后只能手动重置或通过看门狗(WDT)定时重置。

0x04 编程示例：创建软AP
=======================
点选“文件”->“示例”->“ESP8266WiFi”->“WiFiAccessPoint”

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/4-0.PNG)

根据个人喜好做如下修改：
```cpp
...
const char *ssid = "ESPap";//这里修改WiFi名称
//const char *password = "thereisnospoon";//如果开放网络就注释这一行
...
WiFi.softAP(ssid);//如果开放网络就去掉第二个参数
//IPAddress myIP = WiFi.softAPIP();
IPAddress myIP(192, 168, 1, 1);//这里修改热点IP地址
...
```

点击左上角箭头样图标，程序将自动编译并下载到NodeMCU里

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/4-1.PNG)

手机或电脑连接上建立的热点，打开 http://192.168.1.1/ 即可访问页面

0x05 一点进阶操作：DNS劫持
==========================
头文件添加DNSServer库
```cpp
#include <DNSServer.h>

...
DNSServer dns;
...

void setup() {
...
  dns.start(53, "*", myIP);
...
}

void loop() {
...
  dns.processNextRequest();
...
}
```

手机或电脑连接上热点后，访问所有非HTTPS协议的页面时均跳转到 http://192.168.1.1/

0x06 串口通信
=============
NodeMCU拥有一个串口，供下载程序和模块与电脑之间的通信用。串口通信代码示例如下：

```cpp
void setup() {
  Serial.begin(115200);//这里的参数是波特率，波特率越高数据传输越快，但波特率过高可能造成数据丢失
}

void loop() {
  int incomingByte = 0;//存储接收的字符

  if (Serial.available() > 0) {
    incomingByte = Serial.read();//读取电脑发来的字符
    Serial.print("I received: ");//向电脑发送字符串
    Serial.println(incomingByte, DEC);//向电脑发送转成十进制的字符，并在结尾换行
  }
}

```

需要与NodeMCU通信时，打开“工具”->“串口监视器”，设置好端口和波特率后即可发送数据给NodeMCU。

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/6-0.PNG)

注：ESP8266串口接收缓冲区只有128个字节，接收超过128字节后超出部分将被丢弃

0x07 Deauthenticaion攻击
========================
>WiFi Deauthenticaion攻击(以下简称deauth攻击)是一种拒绝服务(DoS)攻击，攻击者以AP的名义向客户端发送802.11管理帧中的结束鉴权帧，使得客户端主动结束鉴权断开连接。Deauth攻击可用于以下三种情况：
>
>恶意伪AP：攻击者对客户端进行deauth攻击，客户端断开当前网络链接后自动连接攻击者创建的热点，从而实现数据包嗅探。
>
>密码攻击：为了暴力破解WPA/WPA2密码，攻击者需要抓到WPA四次握手认证包。攻击者对客户端进行deauth攻击后，即可抓取客户端重新连接网络时发送的握手包。
>
>钓鱼攻击：攻击者进行deauth攻击后，客户端连接攻击者创建的热点，使用中间人攻击手机用户密码。

<div style="text-align: right">from 维基百科 <a href="https://en.wikipedia.org/wiki/Wi-Fi_deauthentication_attack">Wi-Fi deauthentication attack</a></div>

在GitHub上有许多关于deauth攻击的项目，比如<a href="https://github.com/spacehuhn/esp8266_deauther">esp8266_deauther</a> 。我自己也写了一个deauth攻击的演示<a href="https://github.com/smdll/deauther_demo">deauther_demo</a>。编译运行后打开串口监视器即可看到当前无线网络的扫描结果，发送需要攻击的网络编号即开始攻击。这里我尝试攻击自己的热点MX6，攻击开始后所有已连接的设备均被断开链接，且无法再次连接此热点。

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/7-0.PNG)

注：deauth攻击以及0x08章的Beacon Flooding攻击需要正确配置0x01章中的代码补充部分，若未正确配置则代码中的wifi_send_pkt_freedom()函数将无法使用。

0x08 Beacon Flooding攻击
====================================
Beacon帧是802.11管理帧之一，热点在运行时会定期广播Beacon帧，用于公布SSID、频道等参数。Beacon Flooding攻击会不断发送大量随机Beacon帧，用于干扰客户端的扫描列表，甚至能够使某些客户端崩溃。GitHub上有<a href="https://github.com/spacehuhn/esp8266_beaconSpam/">esp8266_beaconSpam</a>等项目，我自己也写了一个攻击演示<a href="https://github.com/smdll/beaconFlooding_demo">beaconFlooding_demo</a>。

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/8-0.png)

# 0x09 microPython内核

### 0x00 准备工作

在电脑上安装Python环境，2.7或3.6都可以。用pip命令安装esptool：

```bash
pip install esptool
```

在<a href="https://micropython.org/download">这里</a>找到Firmware for ESP8266 boards，下载latest固件。连接NodeMCU，确定串口号以后，CMD里运行以下命令（将COM3替换成自己的串口号，esp8266xxxx.bin改成你下载的那个文件）：

```bash
esptool --port COM3 erase_flash
esptool --port COM3 --baud 460800 write_flash --flash_size=detect 0 esp8266xxxx.bin --verify
```

下载完成后，打开串口监视器就可以当作Python解释器来使用了。

注：如果下载过程中出现错误，则把460800波特率修改成115200或更低。如果下载后程序没有运行，则在write_flash后加上”-fm dio“参数。

# 0x0A 一些链接

ESP8266本身以及连接各种模块可以开发出更多东西，完全取决于你的想象力！

在此列举出部分用ESP8266可以实现的项目：

##### Michael shutdown exploitation (WPA/WPA2 TKIP的拒绝服务攻击): <a href="http://securedsolutions.com.my/pdf/WhitePapers/TKIPExploit.pdf">TKIP Exploit</a>    <a href="https://github.com/wi-fi-analyzer/mdk3-master/search?utf8=%E2%9C%93&q=michael&type=">mdk3-michael</a>

##### Key Reinstallation Attacks (KRACK，可以解密WPA/WPA2报文): <a href="https://www.krackattacks.com/">KRACK Attacks</a>    <a href="https://github.com/vanhoefm/krackattacks-scripts">krackattacks-scripts</a>

##### 软件USB模拟(可以模拟键盘设备，实现无线按键注入): <a href="https://github.com/cnlohr/espusb">espusb（此库需要安装低于1.5.4的环境）</a>    <a href="https://github.com/smdll/bad_usb">bad_usb</a>

##### Lua解释器: <a href="https://github.com/Nicholas3388/LuaNode">LuaNode</a>
