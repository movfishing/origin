---
title: Computer Networks exp3
date: 2022-11-29 10:21:40
tags: experiments
categories: ComputerNetworks
---

# 应用层和传输层协议分析（PacketTracer）

### 一、实验目的：

通过本实验，熟悉PacketTracer的使用，学习在PacketTracer中仿真分析应用层和传输层协议，进一步加深对协议工作过程的理解。

### 二、实验内容：

研究应用层和传输层协议

从 PC 使用 URL 捕获 Web 请求，运行模拟并捕获通信，研究捕获的通信。

Wireshark 可以捕获和显示通过网络接口进出其所在 PC 的所有网络通信。Packet Tracer 的模拟模式可以捕获流经整个网络的所有网络通信，但支持的协议数量有限。我们将使用一台 PC 直接连接到 Web 服务器网络，并捕获使用 URL 的网页请求。

* 任务 1：从 PC 使用 URL 捕获 Web 请求。

  * 步骤 1 : 运行模拟并捕获通信。 进入 Simulation（模拟）模式。单击 PC。在 Desktop（桌面）上打开 Web Browser（Web 浏览器）。在浏览器中访问服务器的web服务（服务器的IP地址请自己设置）。单击 Go（转到）将会发出 Web 服务器请求。最小化 Web 客户端配置窗口。Event List（事件列表）中将会显示两个数据包：将 URL 解析为服务器 IP 地址所需的 DNS 请求，以及将服务器 IP 地址解析为其硬件 MAC 地址所需的 ARP 请求。

    单击 Auto Capture/Play（自动捕获/播放）按钮以运行模拟和捕获事件。收到 "No More Events"（没有更多事件）消息时单击 OK（确定）。

  * 步骤 2 : 研究捕获的通信。 在 Event List（事件列表）中找到第一个数据包，然后单击 Info（信息）列中的彩色正方形。单击事件列表中数据包的 Info（信息）正方形时，将会打开 PDU Information（PDU 信息）窗口。此窗口将按 OSI 模型组织。在我们查看的第一个数据包中，注意 DNS 查询（第 7 层）封装在第 4 层的 UDP 数据段中，等等。如果单击这些层，将会显示设备（本例中为 PC）使用的算法。查看每一层发生的事件。

    打开 PDU Information（PDU 信息）窗口时，默认显示 OSI Model（OSI 模型）视图。此时单击 Outbound PDU Details（出站 PDU 详细数据）选项卡。向下滚动到此窗口的底部，您将会看到 DNS 查询在 UDP 数据段中封装成数据，并且封装于 IP 数据包中。

    查看 PDU 信息，了解交换中的其余事件。

* 任务2：从 PC 访问服务器的HTTPS服务，捕获数据包并分析。

* 任务3：从 PC 访问服务器的FTP服务，捕获数据包并分析。

### 三、实验过程

#### 任务1

进入 packet tracer 后，左下角选择`End Devices`即可选择放置PC主机与服务器。再选择`Connection`中的黑线将两个终端连接。

![](/img/networkexp3/connect.png)

双击放置的终端配置其内容如下图：

PC:

![](/img/networkexp3/pc1.png)

![](/img/networkexp3/pc2.png)

Server:

![](/img/networkexp3/server1.png)

![](/img/networkexp3/server2.png)

![](/img/networkexp3/server3.png)

![](/img/networkexp3/server4.png)

配置完成后，点击右下角的`Simulation`进入模拟仿真模式，双击PC，选择`Desktop`中的`Web Browser`，进入刚刚配置的服务器的网址，再点击`Simulation`的播放按钮即可开始捕获数据包。

捕获结果：

![]()

分析结果：

* 首先的几个`DNS`和`ARP`包是进行域名解析时产生的。

    可以查看第一个DNS包：

    ![](/img/networkexp3/dns1.png)

    可以看到写着用户发送了一个DNS查询到服务器。

    ![](/img/networkexp3/dns2.png)

    可以看到查询的域名是`www.example.com`。

    QR=0表示其为请求报文。

    再看最后的DNS包：

    ![](/img/networkexp3/dns3.png)

    可以看到在DNS Answer中返回了查询到的IP地址。

    QR=1表示其为响应报文。

* 随后是三个`TCP`包，三次握手建立连接。

    ![](/img/networkexp3/hello1.png)

    ![](/img/networkexp3/hello2.png)

    ![](/img/networkexp3/hello3.png)

* 中间是`HTTP`请求。

    ![](/img/networkexp3/http1.png)

* 最后是四个`TCP`包，四次挥手断开连接。

    ![](/img/networkexp3/bye1.png)

    ![](/img/networkexp3/bye2.png)

    ![](/img/networkexp3/bye3.png)

    ![](/img/networkexp3/bye4.png)

可以看到tcp flag字段，最后一位为`FIN`，置1则表示有效。

另，中间的第二次挥手似乎也没有捕捉到，推测是与实验一一样，第二、三次挥手合并了。

#### 任务2

打开实验指导文档给的文件，进行模拟，结果如下：

三次握手与第一次HTTP请求(1028端口)：(由于马上又建立了新的TCP连接所以挥手没截，在下部分)

![](/img/networkexp3/http1hello.png)

第一次TCP连接的断开与两个新TCP连接的建立(1029,1030端口)：

![](/img/networkexp3/http2hello.png)

1029端口与1030端口的HTTP请求：

![](/img/networkexp3/httprq1.png)

![](/img/networkexp3/http1rq2.png)

![](/img/networkexp3/http1rq3.png)

可以看到请求的是图片信息。

但是由于请求对象不存在，请求被拒绝了：

![](/img/networkexp3/httperr1.png)

![](/img/networkexp3/httperr2.png)

由于packet tracer实际上并不会加密数据，所以抓https的包结果与http一样。

#### 任务3

由于数据包过于繁多，所以只列出了部分数据包的ftp段，流程大体与http类似，也是先建立tcp连接。

* 成功连接：

![](/img/networkexp3/welcome1.png)

* 写文件：

![](/img/networkexp3/write1.png)

![](/img/networkexp3/write2.png)

![](/img/networkexp3/write3.png)

可以看到分为了command(指令)和argument(参数)两个部分，sampleFile.txt就是指令操作的对象。

* 读文件：

![](/img/networkexp3/read1.png)

![](/img/networkexp3/read2.png)

![](/img/networkexp3/read3.png)

其中sampleFile2.txt就是读入的文件，由于已存在sampleFile.txt，所以被命名为了sampleFile2.txt。

* 列出目录：

![](/img/networkexp3/dir1.png)

![](/img/networkexp3/dir2.png)

* 文件重命名：

![](/img/networkexp3/rename1.png)

![](/img/networkexp3/rename2.png)

![](/img/networkexp3/rename4.png)

![](/img/networkexp3/rename3.png)

可以看到，本操作分为了两个部分，RNFR表示的是要重命名哪个文件，而RNTO表示的是重命名为了什么名字。

* 删除文件：

![](/img/networkexp3/delete1.png)

![](/img/networkexp3/delete2.png)

![](/img/networkexp3/delete3.png)

* 退出：

![](/img/networkexp3/quit1.png)

![](/img/networkexp3/quit2.png)