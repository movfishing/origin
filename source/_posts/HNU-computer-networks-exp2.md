---
title: HNU computer networks exp2
date: 2022-11-08 22:57:54
tags: experiments
categories: ComputerNetworks
---

# 网络基础编程（Python）

实验源码已上传[github](https://github.com/movfishing/Chat-Room-BY-socket)，欢迎查阅

### 一、实验目的：

　　通过本实验，学习采用Socket（套接字）设计简单的网络数据收发程序，理解应用数据包是如何通过传输层进行传送的。  

### 二、实验内容：

Socket（套接字）是一种抽象层，应用程序通过它来发送和接收数据，就像应用程序打开一个文件句柄，将数据读写到稳定的存储器上一样。一个socket允许应用程序添加到网络中，并与处于同一个网络中的其他应用程序进行通信。一台计算机上的应用程序向socket写入的信息能够被另一台计算机上的另一个应用程序读取，反之亦然。

不同类型的socket与不同类型的底层协议族以及同一协议族中的不同协议栈相关联。现在TCP/IP协议族中的主要socket类型为流套接字（sockets sockets）和数据报套接字（datagram sockets）。流套接字将TCP作为其端对端协议（底层使用IP协议），提供了一个可信赖的字节流服务。一个TCP/IP流套接字代表了TCP连接的一端。数据报套接字使用UDP协议（底层同样使用IP协议），提供了一个"尽力而为"（best-effort）的数据报服务，应用程序可以通过它发送最长65500字节的个人信息。一个TCP/IP套接字由一个互联网地址，一个端对端协议（TCP或UDP协议）以及一个端口号唯一确定。

#### 2.1、采用TCP进行数据发送的简单程序

#### 2.2、采用UDP进行数据发送的简单程序

#### 2.3多线程\线程池对比

当一个客户端向一个已经被其他客户端占用的服务器发送连接请求时，虽然其在连接建立后即可向服务器端发送数据，服务器端在处理完已有客户端的请求前，却不会对新的客户端作出响应。

并行服务器：可以单独处理没一个连接，且不会产生干扰。并行服务器分为两种：一客户一线程和线程池。

每个新线程都会消耗系统资源：创建一个线程将占用CPU周期，而且每个线程都自己的数据结构（如，栈）也要消耗系统内存。另外，当一个线程阻塞（block）时，JVM将保存其状态，选择另外一个线程运行，并在上下文转换（context switch）时恢复阻塞线程的状态。随着线程数的增加，线程将消耗越来越多的系统资源。这将最终导致系统花费更多的时间来处理上下文转换和线程管理，更少的时间来对连接进行服务。那种情况下，加入一个额外的线程实际上可能增加客户端总服务时间。

我们可以通过限制总线程数并重复使用线程来避免这个问题。与为每个连接创建一个新的线程不同，服务器在启动时创建一个由固定数量线程组成的线程池（thread pool）。当一个新的客户端连接请求传入服务器，它将交给线程池中的一个线程处理。当该线程处理完这个客户端后，又返回线程池，并为下一次请求处理做好准备。如果连接请求到达服务器时，线程池中的所有线程都已经被占用，它们则在一个队列中等待，直到有空闲的线程可用。

#### 2.4写一个简单的chat程序，并能互传文件，编程语言不限。

### 三、实验步骤

#### 2.1、采用TCP进行数据发送的简单程序

众所周知，TCP是一种面向连接的协议，在进行数据传输之前需要先建立连接，随后通信时就不需要指明源目的地址。

服务端

```python
from socket import *

server_port = 12000

server_socket = socket(AF_INET, SOCK_STREAM)

server_socket.bind(('', server_port))

server_socket.listen(1)

print("I'm ready, go on.")

while 1:
    connection_socket, addr = server_socket.accept()
    message = connection_socket.recv(1024)
    modified_message = message.decode().upper()
    connection_socket.send(modified_message.encode())
    connection_socket.close()

print("server accidently closed.")
```

客户端

```python
from socket import *

server_name = '127.0.0.1'

server_port = 12000

client_socket = socket(AF_INET, SOCK_STREAM)

client_socket.connect((server_name, server_port))

message = input("input lowercase sentence:")

client_socket.send(message.encode())

modified_message = client_socket.recv(1024)

print(modified_message.decode())

client_socket.close()
```

#### 2.2、采用UDP进行数据发送的简单程序

众所周知，UDP是一种无连接的协议，不用在数据传输前建立连接，但是每次通信需要指明目的端口与地址。

服务端

```python
from socket import *

server_port = 12000

server_socket = socket(AF_INET, SOCK_DGRAM)

server_socket.bind(('', server_port))

print("I'm ready, go on.")

while 1:
    message, client_address = server_socket.recvfrom(2048)
    modified_message = message.decode().upper()
    server_socket.sendto(modified_message.encode(), client_address)

print("server accidently closed.")
```

客户端

```python
from socket import *

server_name = '127.0.0.1'

server_port = 12000

client_socket = socket(AF_INET, SOCK_DGRAM)

message = input("input lowercase sentence")

client_socket.sendto(message.encode(), (server_name, server_port))

modified_message, server_address = client_socket.recvfrom(2048)

print(modified_message.decode())

client_socket.close()

```

#### 2.3多线程\线程池对比

若使用多线程则会导致每次建立新连接需要先创建线程并分配资源，在断开连接也需要回收资源，在进行线程切换的时候，系统也需要耗费资源进行管理；而使用线程池则可以避免许多时间消耗与内存开销。

关于线程池，我们可以使用python中的`ThreadPoolExecutor`模块，使用方法十分简单，可以自行搜索使用方法。

多线程服务端程序示例代码：

```python
from socket import *
import threading

def handle(conn,addr) :
    # your code

host = "host_addr"
port = "port_num"

sk = socket(AF_INET, SOCK_STREAM)

sk.bind((host,port))

sk.listen(10)

while Ture :
    conn, addr = sk.accept()
    t = threading.Thread(target=handle, args=(conn, addr))
    t.start()
    conn.close()
```

线程池服务端程序示例代码：

```python
from concurrent.futures import ThreadPoolExecutor
from socket import *

def handle(conn,addr) :
    # your code

executor.ThreadPoolExecutor(10) #创建一个最多10个线程的线程池
host = "host_addr"
port = "port_num"

sk = socket(AF_INET, SOCK_STREAM)

sk.bind((host,port))

sk.listen(10)

while True:
    conn, addr = sk.accept()
    executor.submit(handle,conn,addr)
    conn.close()
```

#### 2.4写一个简单的chat程序，并能互传文件，编程语言不限。

socket不能处理并发的用户请求，但socketserver可以。服务端使用socketserver模块，接受用户端输入的信息与上传的文件，并在服务端上显示出来，同时也可以处理用户端下载文件的请求。

* 服务端部分代码(涉及文件传输的部分过长，有需要请移步github查阅)
```python
import socketserver
import sys
import os
class MyServer(socketserver.BaseRequestHandler):

    def handle(self):
        conn = self.request
        clientlist.append(self.client_address)
        print('{} connect successfully.'.format(
            user_name[clientlist.index(self.client_address)]))
        outFlag = True
        while outFlag:
            mysign = clientlist.index(self.client_address)
            data = conn.recv(1024).decode()
            if data == 'exit':  # 用户断开连接
                outFlag = False
                print("{} has disconnected.".format(
                    user_name[clientlist.index(self.client_address)]))
                clientlist.pop(clientlist.index(self.client_address))
            elif data == 'message':  # 用户发送信息
                conn.sendall("ok".encode())
                while True:
                    recvmsg = conn.recv(1024).decode()
                    if recvmsg == "#exit#":
                        break
                    print("{}:{}".format(
                        user_name[mysign], recvmsg))
            elif data == 'file':  # 用户上传文件
                #sendfile()
            elif data == 'recvfile':  # 用户请求下载文件
                #recvfile()


if __name__ == '__main__':
    server = socketserver.ThreadingTCPServer(('175.10.204.45', 9999), MyServer)
    server.serve_forever()
```

* 用户端部分代码：

```python
def sends():
    while True:
        print("press 1 , 2 or 3 to choose:")
        sel = input('[1]send message [2]send file [3]recive file [4]exit\n')
        if sel == '1':
            sk.sendall("message".encode())
            sk.recv(1024)
            sendmsg() #发信息
        elif sel == '2':
            sk.sendall("file".encode())
            sk.recv(1024)
            sendfile() #上传文件
        elif sel == '3':
            sk.sendall("recvfile".encode())
            print("files:")
            while True:
                names = sk.recv(1024).decode()# 先列出所有已上传的文件
                if names == '1':# 再下载需要的文件
                    needfile = input("which one:")
                    sk.sendall(needfile.encode())
                    recvfile()
                    break
                sk.sendall("ok".encode())
                print(names)
        elif sel == '4':
            sk.sendall("exit".encode())
            time.sleep(1)
            break
        else:
            print("please enter 1 , 2 or 3!")

sends()

sk.close()
```

* 聊天室运行截图：

![用户1](/img/computernetworksexp2/c1.png)

![用户2](/img/computernetworksexp2/c2.png)

![服务端](/img/computernetworksexp2/s1.png)

---

实验到这里就结束啦！

---

### 关于本人在此次试验踩过的雷：

* 在实验指导文档提供的文件中使用了socketserver，是可以直接处理并发连接请求的，不需要再去创建线程池了！

* socket.recv一定要记得填写接收数据大小，而且要注意避免数据粘包。

* socket.recv是阻塞进行的，所以要注意避免下种情况：语句B的recv由于语句A的recv而阻塞，语句A后紧接着的语句C的recv想要接受的数据会被语句B的recv抢走。