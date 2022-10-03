---
title: HNU computer networks exp1
date: 2022-10-03 20:35:00
categories:
- ComputerNetworks
tags:
- experiments
---

# 应用协议与数据包分析(Wireshark)
### 一、实验目的
通过本实验，熟练掌握Wireshark的操作和使用，学习对HTTP协议进行分析。

---

### 二、实验内容
#### 1.HTTP 协议简介
略。

#### 2.实验环境与说明
（1）实验目的 
在PC 机上访问Web 页面，截获报文，分析HTTP 协议的报文格式和HTTP 
协议的工作过程。

（2）实验设备和连接 
本地实验室环境，无须设备连接； 
注意：请通过访问可以连接的WWW 站点或使用IIS 建立本地WWW 服务器来进行实验。

（3）实验分组 
每四名同学为一组，每人一台计算机独立完成实验。

#### 3.实验步骤
步骤1：在PC 机上运行Wireshark，开始截获报文； 

步骤2：从浏览器上访问Web 界面，如 http://csee.hnu.edu.cn 打开网页，待浏览器的状态栏出现“完毕”信息后关闭网页。 

步骤3：停止截获报文，将截获的报文命名为http-学号保存。

---

***开始实验吧！***

---

#### 一、Wireshark的下载与使用
Wireshark（前称Ethereal）是一个网络封包分析软件。网络封包分析软件的功能是截取网络封包，并尽可能显示出最为详细的网络封包资料。Wireshark使用WinPCAP作为接口，直接与网卡进行数据报文交换。

直接前往[wireshark官网](https://www.wireshark.org/)下载安装即可。

下面介绍一些本次实验相关的基本步骤与操作。

* 安装完成后打开wireshark，界面如下：

![](/img/networkexp1/zjm.png)

  wireshark是根据你选择的网口来进行抓包，所以我们要选择你当前使用的网口，比如我现在使用的是校园宽带，就是图中的以太网。(其实也可以通过看网口旁边的小曲线图有没有起伏判断你当前使用的网口)

* 选择好网口后，双击就可以进入抓包的界面了。可以看到包多且杂，需要用到wireshark所提供的**过滤功能**。

    1.捕获过滤器

    在 捕获-->捕获过滤器 中设置，需要在捕获前设置。

    ![](/img/networkexp1/bhglq.png)

    语法：<协议><方向>< host><值><逻辑操作符><其他表达式>

    例：`ip src host 180.101.49.12 and ip dst host 175.10.204.113`

    那么就只会捕获来源为ip=180.101.49.12和目的地为ip=175.10.204.113的包。

    (~~但是我在捕获过滤器中设置了不会用~~，所以我是在选择网口的界面中设置的。如下图)

    ![](/img/networkexp1/wdbhglq.png)

    2.显示过滤器

    在 分析-->显示过滤器 中设置，在捕获时或捕获后设置。

    ![](/img/networkexp1/xsglq.png)

    语法：<协议><字段><比较运算符><值>

    例：输入 `http` 就只会显示http协议的内容。

    `ip.addr==180.101.49.12`就只会显示包来源或目的地ip=180.101.49.12的内容。

    (~~但是我在显示过滤器中设置了也不会用~~，所以我是在捕获界面中直接设置的。如下图)

    ![](/img/networkexp1/wdxsglq.png)

* 关于设置过滤的条件，本人采用如下方法：

    * 先选择好网口后开始捕获，然后用浏览器打开你想操作的网址。
    * 打开cmd，输入命令 `ping 你想操作的网址`(关于ping的下载使用请自行百度)，可以得到一个ip。
    * 将刚刚得到的ip在显示过滤器中设置：`ip.addr == xxx.xxx.xxx.xxx and http`，即可显示所捕获的http报文。

    ![](/img/networkexp1/bw.png)

    记得及时停止捕获，不然过滤还是有点费时间的。

```
 在捕获时，这里有一个本人踩过的小坑。wireshark是无法解析https格式的报文的，所以你如果是访问的https网站，在捕获界面压根不会显示https的相关内容(当然肯定也没有http)。所以建议还是使用实验指导文档中所提供的网站。
```

* 最后在左上角点击 文件-->导出特定分组 即可导出你所需要保存的报文。

#### 二、http报文的分析(实验问题1-4)

* 问题1：综合分析截获的报文，查看有几种HTTP 报文？

    很明显，是请求报文和响应报文两种。其中有`GET`字样的报文就是请求报文，其他的是响应报文。

* 问题2：在截获的HTTP 报文中，任选一个HTTP 请求报文和对应的 HTTP应答报文，仔细分析它们的格式，填写表1.1 和表1.2。

    ![](/img/networkexp1/to.png)

表1.1 HTTP 请求报文格式

<table border=0 cellpadding=0 cellspacing=0 width=640 style='border-collapse:
 collapse;table-layout:fixed;width:480pt'>
 <col width=64 span=10 style='width:48pt'>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 width=64 style='height:15.0pt;width:48pt'>方 法</td>
  <td class=xl66 width=64 style='width:48pt'><span lang=EN-US>GET</span></td>
  <td class=xl66 width=64 style='width:48pt'>版 本</td>
  <td colspan=7 class=xl68 width=448 style='border-right:1.0pt solid black;
  border-left:none;width:336pt'><span lang=EN-US>http/1.1</span></td>
 </tr>
 <tr height=38 style='mso-height-source:userset;height:28.8pt'>
  <td height=38 class=xl67 width=64 style='height:28.8pt;width:48pt'><span
  lang=EN-US>URL</span></td>
  <td colspan=9 class=xl68 width=576 style='border-right:1.0pt solid black;
  border-left:none;width:432pt'><span lang=EN-US>http://csee.hnu.edu.cn/js/bdtxk.js</span></td>
 </tr>
 <tr height=39 style='height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'>首部字段名</td>
  <td colspan=4 class=xl70 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>字段值</td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>字段所表达的信息</td>
 </tr>
 <tr height=39 style='mso-height-source:userset;height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Host</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>csee.hnu.edu.cn</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>指明对象所在的主机</td>
 </tr>
 <tr height=39 style='mso-height-source:userset;height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Connection</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>keep-alive</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>要求服务器采用持续连接</td>
 </tr>
 <tr height=80 style='mso-height-source:userset;height:60.0pt'>
  <td height=80 class=xl67 width=64 style='height:60.0pt;width:48pt'><span
  lang=EN-US>User-Agent</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>Mozilla/5.0 (Windows NT 10.0;
  Win64; x64; rv:105.0) Gecko/20100101 Firefox/105.0</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>指明用户发送请求的浏览器的类型</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Accept</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>*/*</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>表示接受的响应body可以是任何类型</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Accept-Encoding</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>gzip, deflate</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>表示这个请求的内容希望被压缩,减少网络流量</td>
 </tr>
 <tr height=66 style='mso-height-source:userset;height:49.95pt'>
  <td height=66 class=xl67 width=64 style='height:49.95pt;width:48pt'><span
  lang=EN-US>Accept-Language</span></td>
  <td colspan=4 class=xl72 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'><span lang=EN-US>zh-CN,zh;q=0.9,en;q=0.8,en-GB;q=0.7,en-US;q=0.6</span></td>
  <td colspan=5 class=xl68 width=320 style='border-right:1.0pt solid black;
  border-left:none;width:240pt'>浏览器支持以上语言，使用的优先级以q的值从大到小递减</td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
 </tr>
 <![endif]>
</table>

![](/img/networkexp1/back.png)

表2.2 HTTP 应答报文格式

<table border=0 cellpadding=0 cellspacing=0 width=512 style='border-collapse:
 collapse;table-layout:fixed;width:384pt'>
 <col width=64 span=8 style='width:48pt'>
 <tr height=20 style='height:15.0pt'>
  <td height=20 class=xl65 width=64 style='height:15.0pt;width:48pt'>版 本</td>
  <td class=xl66 width=64 style='width:48pt'><span lang=EN-US>http/1.0</span></td>
  <td class=xl66 width=64 style='width:48pt'>状态码</td>
  <td colspan=5 class=xl71 width=320 style='border-left:none;width:240pt'><span
  lang=EN-US>304</span></td>
 </tr>
 <tr height=20 style='mso-height-source:userset;height:15.0pt'>
  <td height=20 class=xl67 width=64 style='height:15.0pt;width:48pt'>短 语</td>
  <td colspan=7 class=xl72 width=448 style='border-left:none;width:336pt'><span
  lang=EN-US>Not Modified</span></td>
 </tr>
 <tr height=39 style='height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'>首部字段名</td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'>字段值</td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>字段所表达的信息</td>
 </tr>
 <tr height=78 style='mso-height-source:userset;height:58.2pt'>
  <td height=78 class=xl67 width=64 style='height:58.2pt;width:48pt'><span
  lang=EN-US>Date</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>Mon, 03 Oct 2022 16:28:07 GMT</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>服务器产生并发送该响应报文的时间</td>
 </tr>
 <tr height=39 style='mso-height-source:userset;height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Server</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>*********</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>web服务器软件名称</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>X-Frame-Options</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>SAMEORIGIN</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>表示该页面可以在相同域名页面的 frame 中展示</td>
 </tr>
 <tr height=77 style='mso-height-source:userset;height:57.6pt'>
  <td height=77 class=xl67 width=64 style='height:57.6pt;width:48pt'><span
  lang=EN-US>Accept-Ranges</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>bytes</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>表明服务器是否支持指定范围请求及哪种类型的分段请求</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Cache-Control</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>max-age=3600</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>告诉所有的缓存机制是否可以缓存及哪种类型</td>
 </tr>
 <tr height=78 style='mso-height-source:userset;height:58.2pt'>
  <td height=78 class=xl67 width=64 style='height:58.2pt;width:48pt'><span
  lang=EN-US>ETag</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>&quot;4fc5-5672ebfe27840-gzip&quot;</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>请求变量的实体标签的当前值</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Vary</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>Accept-Encoding</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>告诉代理服务器缓存两种版本的资源：压缩和非压缩</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Content-Encoding</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>gzip</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>web服务器支持的返回内容压缩编码类型</td>
 </tr>
 <tr height=39 style='height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Content-Length</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>5163</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>响应body长度</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.8pt'>
  <td height=58 class=xl67 width=64 style='height:43.8pt;width:48pt'><span
  lang=EN-US>Content-Type</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>application/javascript</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>内容的类型，这里为javascript</td>
 </tr>
 <tr height=39 style='mso-height-source:userset;height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Content-Language</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>zh-CN</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>响应内容的语言，这里为简中</td>
 </tr>
 <tr height=39 style='mso-height-source:userset;height:29.4pt'>
  <td height=39 class=xl67 width=64 style='height:29.4pt;width:48pt'><span
  lang=EN-US>Expires</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>01:39:51 GMT</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>响应过期的日期和时间</td>
 </tr>
 <tr height=78 style='mso-height-source:userset;height:58.2pt'>
  <td height=78 class=xl67 width=64 style='height:58.2pt;width:48pt'><span
  lang=EN-US>Last-Modified</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>Mon, 12 Mar 2018 03:29:29 GMT</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>请求资源的最后修改时间</td>
 </tr>
 <tr height=58 style='mso-height-source:userset;height:43.2pt'>
  <td height=58 class=xl67 width=64 style='height:43.2pt;width:48pt'><span
  lang=EN-US>Age</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>1867</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>从原始服务器到代理缓存形成的估算时间</td>
 </tr>
 <tr height=78 style='mso-height-source:userset;height:58.2pt'>
  <td height=78 class=xl67 width=64 style='height:58.2pt;width:48pt'><span
  lang=EN-US>X-Cache</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>HIT from Hnu-Chinanet-Proxy</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>请求资源在哪个代理服务器找到</td>
 </tr>
 <tr height=78 style='mso-height-source:userset;height:58.2pt'>
  <td height=78 class=xl67 width=64 style='height:58.2pt;width:48pt'><span
  lang=EN-US>Via</span></td>
  <td colspan=3 class=xl69 width=192 style='border-right:1.0pt solid black;
  border-left:none;width:144pt'><span lang=EN-US>1.0 Hnu-Chinanet-Proxy (squid)</span></td>
  <td colspan=4 class=xl69 width=256 style='border-right:1.0pt solid black;
  border-left:none;width:192pt'>告知代理客户端响应是通过哪里发送的</td>
 </tr>
 <![if supportMisalignedColumns]>
 <tr height=0 style='display:none'>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
  <td width=64 style='width:48pt'></td>
 </tr>
 <![endif]>
</table>

* 问题3：分析在截获的报文中，客户机与服务器建立了几个连接？服务器和客户机分别使用了哪几个端口号？

    客户机的端口号是固定的一个：80

    服务器使用了个6端口号：64036，64037，64041，64042，64043，64044

    所以我们可以通过分析得出客户机与服务器建立了六个连接。

* 问题4：综合分析截获的报文，理解HTTP 协议的工作过程，将结果填入表1.3 中。

![](/img/networkexp1/tcps.png)

表1.3 HTTP 协议工作过程

| HTTP 客户机端口号 | HTTP 服务器端口号 | 所包括的报文号 | 步骤说明                             |
| ----------------- | ----------------- | -------------- | ------------------------------------ |
| 80                | 64036             | 889            | 浏览器向服务器发出连接请求           |
| 80                | 64036             | 891            | 服务器回应了浏览器的请求，并要求确认 |
| 80                | 64036             | 892            | 浏览器回应了服务器的确认，连接成功   |
| 80                | 64036             | 893            | 浏览器发出一个页面HTTP请求           |
| 80                | 64036             | 898            | 服务器确认                           |
| 80                | 64036             | 899-903        | 服务器发送数据                       |
| 80                | 64036             | 904            | 服务器发送状态响应码                 |
| 80                | 64036             | 906            | 客户端浏览器确认                     |

## 那么实验到这里就结束啦！
---

***个人的一点碎碎念：***

第一次尝试写博客，总之就是痛~~并快乐着~~

本来想试试抓我自己博客网站的包，结果折腾了大半天才知道wireshark原来不支持解析https协议 ~~，很急~~

总之希望自己能坚持下去咯，加油加油







