0x00 Arduino IDE环境安装
========================
在 https://www.arduino.cc/en/Main/Software 上找到Windows的最新版本，下载Installer安装或ZIP解压都可以。

0x01 ESP8266 SDK安装与配置
==========================
在IDE里点选“文件”->“首选项”

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-0.PNG)

点选“附加开发板管理器网址”右侧的按钮

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-1.PNG)

输入 http://arduino.esp8266.com/stable/package_esp8266com_index.json 并保存

点选“工具”->“开发板xxxxxx”->“开发板管理器”，等待平台索引更新

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-2.PNG)

翻到页面最下方，找到“esp8266 by ESP8266 Community”，选择版本2.0.0安装

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/1-3.PNG)

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
CH340芯片： http://www.wch.cn/download/CH341SER_EXE.html

CP2102芯片： https://www.silabs.com/products/development-tools/software/usb-to-uart-bridge-vcp-drivers

插入NodeMCU，等待驱动程序安装完成后打开设备管理器，记住使用的端口（图例为CP2102的芯片，COM5）

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/2-1.PNG)

在Arduino IDE里“工具”->“端口”选择对应端口

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/2-2.PNG)

0x03 基础语法
=========
Arduino IDE使用两个入口函数：setup()和loop()，大致语法与C++类似，具体内置函数列表在“帮助”->“参考”。setup()和loop()编译后结构大致如下：
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

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/4-1.PNG)

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

![image](https://github.com/smdll/HackMCU_tutorial/raw/master/resources/4-2.PNG)

手机或电脑连接上建立的热点，打开 http://192.168.1.1/ 即可访问页面

0x05 一点进阶操作：DNS劫持
==========================
头文件添加DNSServer库
```cpp
#include <DNSServer.h>
```

创建全局实例
```cpp
DNSServer dns;
```

setup函数内添加初始化语句
```cpp
dns.start(53, "*", myIP);
```

loop函数内添加循环处理请求
```cpp
dns.processNextRequest();
```

手机或电脑连接上热点后，访问所有非HTTPS协议的页面时均跳转到 http://192.168.1.1/