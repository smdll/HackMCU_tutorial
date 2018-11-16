0x00 Arduino IDE环境安装
========================
在 https://www.arduino.cc/ 上找到Windows的最新版本，下载安装。

0x01 ESP8266 SDK安装与配置
==========================
在IDE里点选“文件”->“首选项”

![alt text](.\resources\1-0.PNG "1-0")

点选“附加开发板管理器网址”右侧的按钮

![alt text](.\resources\1-1.PNG "1-1")

输入 http://arduino.esp8266.com/stable/package_esp8266com_index.json 并保存

点选“工具”->“开发板xxxxxx”->“开发板管理器”，等待平台索引更新

![alt text](.\resources\1-2.PNG "1-2")

翻到页面最下方，找到“esp8266 by ESP8266 Community”，选择版本2.0.0安装

![alt text](.\resources\1-3.PNG "1-3")

安装完毕后打开文件管理器，输入

%USERPROFILE%\AppData\Local\Arduino15\packages\esp8266\hardware\esp8266\2.0.0\tools\sdk\include （如找不到则把%USERPROFILE%改成当前用户的主文件夹，比如你的账户叫Administrator，就改成C:\Users\Administrator）

![alt text](.\resources\1-4.PNG "1-4")

修改“user_interface.h”，在文件末尾#endif前添加如下内容
```cpp
typedef void (*freedom_outside_cb_t)(uint8 status);
int wifi_register_send_pkt_freedom_cb(freedom_outside_cb_t cb);
void wifi_unregister_send_pkt_freedom_cb(void);
int wifi_send_pkt_freedom(uint8 *buf, int len, bool sys_seq);
```

![alt text](.\resources\1-5.PNG "1-5")

保存后重启Arduino IDE，点选“工具”->“开发板xxxxxx”->“NodeMCU 1.0(ESP-12E module)”

![alt text](.\resources\1-6.PNG "1-6")

0x02 NodeMCU驱动程序
====================
CH340芯片： http://www.wch.cn/download/CH341SER_EXE.html

CP2102芯片： https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers

插入NodeMCU，等待驱动程序安装完成后打开设备管理器，记住使用的端口（图例为CP2102的芯片，COM5）

![alt text](.\resources\2-1.PNG "2-1")

在Arduino IDE里“工具”->“端口”选择对应端口

![alt text](.\resources\2-2.PNG "2-2")

0x03 编程示例：创建软AP
=======================
点选“文件”->“示例”->“ESP8266WiFi”->“WiFiAccessPoint”

![alt text](.\resources\3-1.PNG "3-1")

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

![alt text](.\resources\3-2.PNG "3-2")

手机连接上建立的热点，打开http://192.168.1.1/即可访问页面

0x04 一点进阶操作：DNS劫持
==========================
