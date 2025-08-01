https://www.webfx.com/tools/emoji-cheat-sheet/#emoji-support
# T87A0

- 关于2.3.x版本(在T9000基础下测试)：

登陆地址在US，在QA环境下测试。账号 guiyuan.liu@oceanwing.com + 密码 guiyuan2020
guiyuan home

- 关于2.2.x版本：

登陆地址在EU，无法进入QA环境。账号 guiyuan.liu@anker.com + 密码 guiyuan2020
guiyuan home

# 关于adb

全称Android Debug Bridge，用于连接、调试和管理Android设备或模拟器。在智慧屏项目背景下，将T87A0与电脑用数据线连接后，通过常用的adb指令能够：

1️⃣查看设备状态，是否进入QA环境或是否与电脑连接成功；
```
adb device
```

2️⃣打印输出日志，输出位置不用命令指定，在文件夹下cmd。
```
adb pull/sdcard/log
```

## 关于apk
针对Android系统定制开发，需要用平台指定的加密密钥对各个应用（APK）签名，然后分别把它们放到指定的系统目录下，系统开机后这些应用就是“内置、具有高权限的系统应用”，具备普通应用不具备的能力，并且不会被用户轻易卸载。

在智慧屏项目背景下，apk包含：Launcher；Login；Provision；Setting；SystemUI。其中Setting比较特殊，似乎不随着OTA升级烧录而变化(?)。其余的部分在OTA后全被覆盖。为避免安装新版apk包时常出现的权限问题，直接使用强制安装命令：
```
adb install -t -d 包名
```
ps：唯一一次安装Launch包存放在D:\T87A0\UAT下。

## 关于OTA
（测试机非法，每次OTA只采用整机烧录的形式）

OTA - 设备整体的版本升级。

在智慧屏项目背景下，采用RKDevTool作为软件烧录的工具。前置需要安装好驱动。

# 缺陷提单
## 提单核心
- 提单面向开发人员。最重要的标题中要突出两个内容：1）bug造成的结果；2）必现/随机。

- 测试人员要执行的单状态：（只有两种）

1. 已解决->关闭
2. 已解决->拒绝

## 回归验证bug标准
[图片]
随即重现的低概率bug：需要多验证几个版本

## 提单状态流转规范
1. 如果某个已解决(Solved)的BUG经验证回归通过,则Close该BUG;
2. 如果某个已拒绝(Rejected)的BUG经确认不是问题,则Close该BUG,该BUG将会被标记为无效BUG(Close-Rejected);
3. Close Bug的解决方案分类包括:
  a. Fixed (已解决)
  b. Cannot Reproduce (无法重现)
  c. Won't Fix (无需解决)
  d. Not Problem (不是问题)
  e. Duplicate (重复问题)
  f. Incomplete (问题描述不全)
  g. Done (已完成)

# T9000
一、拉取日志的常用字段
示例日志：
```
[11:12:25][115729.904484] usb 1-1.3 ra0: Del sta 24 (f4:9d:8a:f0:1d:ca)
[11:12:25][115729.915923] usb 1-1.3 ra0: Add sta 25 (f4:9d:8a:f0:1d:ca) flags=[SHORT_PREAMBLE][WME][AUTHENTICATED][ASSOCIATED]
[14:04:12][124385.775458] usb 1-1.3 ra0: Ignore PS mode change on invalid sta
[11:12:25][115729.918069] [I]send_msg_user() at 161.mac: p_name: anker_netlink_service, msg type:1.
[11:13:01][115729.918155] [I]send_msg_user() at 192.out -.
```
- [115729.904484]
是系统运行后的时间戳，一般是秒.微秒。从开机或运行后的某一基准时间算起；用户在输入下发指令时不会显示这个时间戳
- usb 1-1.3 ra0:
usb 1-1.3 指明了连接适配器或接口是USB总线上的某个设备；ra0 是无线网卡设备的名称（常见于某些芯片厂的Wi-Fi/基站设备）；
- Del sta 24 (f4:9d:8a:f0:1d:ca)
表示删除了编号为24，物理地址（MAC）为f4:9d:8a:f0:1d:ca的客户端（sta, 即Station，普通连接设备）；
- Add sta 25 (f4:9d:8a:f0:1d:ca) flags=[...]
表示增加了编号为25，同样MAC地址的设备，并后面带有一串flags，说明新的连接状态属性。flags中的常见字段：
  - SHORT_PREAMBLE：表示连接设备使用的是短前导码（提升效率，但需设备支持）。
  - WME：无线多媒体扩展（支持QoS，比如视频优先等）。
  - AUTHENTICATED：已认证，通过了连接身份认证过程。
  - ASSOCIATED：已经关联，连接过程完成，可以正常通信。
  - AUTHORIZED：拥有数据通信权限。
- send_msg_user()
函数名，表示正在调用发送消息到用户空间（或其它进程）；
- p_name: anker_netlink_service
相关的进程服务名（比如监控或数据转发）；
- msg type:1
消息类型编号，不同编号代表不同动作或事件；
- Ignore PS mode change on invalid sta
PS mode 为Power Save，即省电模式。此消息指出尝试为无效设备切换省电模式，但该设备当前已无效（例如刚断开/删除）。
- [I]send_msg_user() at 192.out -.
192.out -.是在定位位置，表示程序内特定代码行标识/分支/文件名与代码行数。很多嵌入式、通信程序喜欢用类似“文件名.行号（File.Line）”或者“序号.分支名”等，便于开发人员定位代码。例如1）161.mac：表示mac.c文件第161行；2）192.out：极可能是“out.c”（或某个out输出相关的C文件）第192行位置的日志点。

# Baby Monitor T6311 + cam
## 串口接线拉取日志（UART通信方式）：
monitor的板子上接线，对应TX（发送数据）连接电脑端的RX（接受数据），RX连接电脑端的TX，还剩下一个地线GND需要连接。3V3（红线）可以不接。
## BBM和cam初期的配网流程？
- 涉及两个阶段，对应两种网络。
  
    阶段一，本地2.4GHZ wifi：cam用本地直连(AP)的方式发出热点，屏端连接。此时工作在本地模式下，可以实现的功能有：出流，对cam的部分（？）功能控制。

    阶段二，Wifi：本地连接成功后，屏幕通过某种方式（？）把WiFi的信息同步推送给cam，cam自己用这些信息尝试连入WiFi。摄像头接入WiFi成功后，会把自身连接成功的信号和实际用的WiFi信息同步给屏幕，屏幕收到信号以后，再去连接家用WiFi，实现和摄像头在同一局域网下的联网通信。所以cam是第一接入WIFI的角色。
  
- cam比屏先接入，这样做的好处？

    cam连接过程一旦出错，可以即时在本地连接条件下反馈到屏（有交互，比简单的cam更能动态做调整与适应），避免屏先入网再等待cam，可能连接延迟或失败。屏根据cam的实际状态动态调整自己的联网流程，提升配对和联调的稳定性。

## cam和BBM连接的流程：
![image](https://github.com/Lyuan15/-.github.io/blob/main/cam%E5%92%8C%E5%B1%8F%E8%BF%9E%E6%8E%A5.png)

## 关于OTA：
可以用手动在secureCRT中下发指令的方式（串口方式要稳定一些，telnet容易断，且抓取日志内容不完整），让cam陷入OTA的循环。这里涉及到两个概念：

    - 如果给cam部署的是真实的灰度号，则cam更新到最新版本后检测不到还有新版本了，所以没办法让cam走正常的OTA流程；

    - 如果给cam部署了虚高的灰度号，则cam更新到最新版本，但在手机app中检测“固件最新版本”时永远能探测到一个更新的cam版本，则cam可以实现检测版本-下载版本-固件擦写更新的步骤。

此时，可以在给cam部署了虚高灰度号的前提下，用编写脚本的方式自动让cam陷入OTA的循环。编写脚本的内容如下：

```
cd /user/              #进入 /user/ 目录
ls                     #查看当前目录下的文件，确认文件是否存在
touch /user/ota_loop   #当前目录下创建一个名为 ota_loop 的空文件。执行过这一步后，cam开始循环执行OTA。
rm ota_loop            #删除 ota_loop 文件，不然cam会陷入死循环
reboot                 #设备重启
```
在需要结束时手动断开连接。此时查看抓取到的日志，根据关键字完成数据统计。统计指标包括：
![image](https://github.com/Lyuan15/-.github.io/blob/main/OTA%E5%8E%8B%E6%B5%8B%E7%BB%9F%E8%AE%A1%E8%A1%A8.jpg)
其中，每一个统计指标要根据如下日志关键字查找：（测试时长是通过自动化脚本实现的，不清楚具体流程）
```
OTA_START | OTA_running      #测试次数
OTA_IMG_DOWNLOAD_SUCCEED     #下载OTA包成功次数
UPGRADE EUFY UGRADE SUCCESS  #OTA成功次数，即擦写flash成功。可以使用关键字OTA_SUCCEED，但偶尔这个关键字会不显示，所以以擦写为准。
OTA_FAILD                    #OTA失败次数。在这个命令前不远处可以看到失败原因。如果是关键字“the wireless is abnormal”代表是因为网络异常失败。
```
## 当触发了大声检测时，屏端有push且保存了录像，但是录像没有标题（哭声和温度都有标题）
执行如下操作：

  - reboot 重启设备
  - 获取root权限
  - 输入命令：zxdebug ui_debug 15
  - 在屏上点击录像回放的片段
  - 在secureCRT上输入cd /tmp/
  - 输入命令ls，查看是否有record.json这个文件
  - 打印出来这个文件的内容

为什么要这样做？为什么用zxdebug的方式获取日志而不是直接看串口打印的内容？

  - 首先关于命令内容。在输入的命令zxdebug ui_debug 15中，zxdebug通常是厂商自带或者工程开发版固件内置的调试/诊断工具，用于打开、关闭系统各个业务模块的debug（调试）开关。ui_debug是它的一个子命令或调试项，专门控制“UI（用户界面）相关业务”的调试功能。15是参数，代表“调试等级”或者“调试bit位”的开关（有些厂商用bit位组合，不同数字含义不同）。
  - 其次输入这个命令后，在屏上点击录像回放的片段才会记录下一个record.json文件。许多嵌入式设备和监控类摄像头，一般将生成业务json、抓包、特殊日志或行为“保护”在调试模式下，只有打开特定debug/测试开关时才触发这些额外输出，普通用户界面下并不会产生这些中间产物以防泄露或者造成性能负担。zxdebug ui_debug 15后，开发/诊断代码会执行“写record.json”这类辅助函数，便于开发、测试、验证业务流程。
  - 最后，关于zxdebug和串口内容。串口日志主要输出一部分实时业务运行信息，如错误、警告、状态提示、关键流程的简明log，受限于输出带宽、隐私、安全和性能，许多详细结构化的调试数据不会输出到串口。record文件属于结构化调试文件，内容量大且层次结构复杂。+串口打印简明log也避免大量的资源占用。

# SecureCRT
## 一、SecureCRT是什么？
是一种串口调试工具。针对Windows、Mac和Linux的终端仿真软件，通过提供会话管理为个人提供安全的远程访问、文件传输和数据隧道。能实现多会话启动。
## 二、环境配置
### 2.1 安装SecureCRT

1. 需要破解，安装流程：https://www.xue51.com/soft/10266.html#xzdz

### 2.2 配置串口
1. 因基站连接直接使用数据线进行串口通信，需要在设备管理器 - 端口中为Silicon Labs CP210x USB to UART Bridge (COM3)安装驱动。Silicon Labs CP210x USB to UART Bridge (COM3)表明CP210x系列（常见型号有 CP2102、CP2104、CP2109 等等）的USB转串口芯片，是指通过USB接口，把电脑的数据转换成为UART（串口通信）信号，让没有传统串口接口（COM口）的电脑可以通过USB连接外部串口设备，比如单片机开发板、3D打印机、嵌入式模块、传感器、路由器刷机等。

2. 安装驱动C210x。安装流程：https://zhuanlan.zhihu.com/p/370245114。安装包（官网www.silabs.com下载）

3. 安装驱动成功后，设备管理器 - 端口中Silicon Labs CP210x USB to UART Bridge (COM3)的黄色警告图标消失，电脑识别CP210x芯片，串口调试工具中正常工作。

## 三、部署T9000基站
每个基站、需要监听的设备都有自己的Baud rate。T9000固定为1500000，直接用数据线连接设定串行通信Serial。（选项卡中其余设定未知？）

## 四、用途
### 4.1 拉取记录日志
自动记录
### 4.2 发送指令
在串口通信设置下取消勾选RTS/CTS+勾选Options-Session options - Advanced - Local echo。

关于RTS/CTS：是一种用于硬件的流控方案，确保双方在合适时机收发数据，避免丢包/冲突。

由于开发板的驱动暂未实现RTS(Request to send)/CTS(Clear To Send)的流控制，所以在勾选状态下如果SecureCRT需要下发(RTS)指令，首先要检测开发板CTS的pin脚，确认自己的pin脚有效后才下发指令，故SecureCRT卡死，用户无法通过SecureCRT下发指令给基站。通信流程如下：
- A想发送数据：A拉高RTS线（发B一个RST信号），告诉B“我想发数据”。
- B准备好接收：B拉高CTS线（响应A的RTS），允许A发送数据。
- A收到了CTS高电平后：开始通过TX/RX线正式发送数据。
- 
### 4.3 制作使用脚本-重复性操作记录为一个脚本文件
执行一系列操作（如发送指令）后，将操作流程全部记录为一个脚本保存至本地。Script - Start recording script，在下方Send commands窗口中输入基础指令后发送，Script-Stop recording script后将脚本（TestScript.vbs）保存至本地。

Script - Run选择TestScript.vbs，脚本自动执行，在本地回显中看到一个NULL指令已下发。

# 杂七杂八Tips
- 项目全程要经历不同的TR阶段，立项、计划、开发、验证、发布...
- WBS作为开发阶段的一部分，代表着市场调研的用户需求转化成开发的解决方案（用户语言到开发语言），把项目目标逐层拆解为更小、更易管理和可执行的任务单元。对于测试，WBS可以帮助项目交付+人力/排期/资源保障。
- “生态”？

  （产品生态？）某个品牌(?)将他的产品、服务全部连接在一起，能用自己的系统把全部产品服务联动起来，彼此依赖相互配合。在用户视角的“生态”概念：设备之间协作以达成某项需求。eg:用iphone拍下的照片，可以在Mac，ipad上查看；airdrop实现各类设备文件互传...
- 关于Testflight

  手机app面向两种系统——Android，上线app相对easy；IOS，管控较严格，选择用TestFlight作为内测工具，让研发以用户的身份对app内测后再正式发布。eufy提测的各种版本都在TestFlight中有所保留，内测时可以随意切换。
- 智慧屏项目背景下，“自动化测试”的应用场景？

  UI自动化测试：用脚本模拟在智慧屏上点点点；
  
  OTA专项自动化测试：是属于UI测试，但因为比较重要所以独立出来；
  
  核心在于压力测试；
  
  针对嵌入式相关的硬件，有很多场景需要机械臂辅助执行自动化测试（eg. 手机扫码绑定摄像头，智慧屏外设按键等）；
- 无法通过Telnet方式拉取BBM屏日志？

  将屏连接至电脑后虚拟网卡加载失败，原因不清楚。改为使用UART原理的串口通信拉取。接线时：TX-RX；RX-TX；GND-GND。两种连接方式：
  
    A. 屏虚拟无线网卡，通过telnet的方式在secureCRT中拉取日志。
   - 只看到登录后的shell终端能访问的内容，是通过标准输入、输出（stdin/stdout）在用户空间里查到的日志，有些低级别early boot信息不会送达telnet终端。
  
    B. 屏通过UART方式连接电脑，通过serial串口方式在secureCRT中拉取日志。
   - 直接获得测试人员专用的debug通道，看到的是系统自启动、驱动加载、核心异常、panic、bootloader刷屏等所有自动打印的log，系统没起来前也会输出东西。通常能捕捉更全的、早期的和出错时的详细信息。
 
后面还在做的事（比较耗时间的）：

1.在coding平台的测试部分执行不同版本下的测试用例；

2.在某个节点前执行兼容性测试，即测试某一个智慧屏系统版本下，好多个cam连接智慧屏时。智慧屏与他们之间的各种交互（出流、出流页面功能、首页控制、布撤防、历史视频、报警与布防模式切换、本地缓存、主被动报警、设备快照、push通知）。

