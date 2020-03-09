#### 阻塞IO

默认情况下的socket通信过程：
![https://images2017.cnblogs.com/blog/1036857/201708/1036857-20170831215423765-2063960072.png](https://images2017.cnblogs.com/blog/1036857/201708/1036857-20170831215423765-2063960072.png)

具体步骤：  
1. 程序执行 recvfrom/recv/send等网络操作，触发系统调用；
2. os等待数据的到来；
3. os如果等到了数据，拷贝进内存/buffer，如果没等到则继续等（阻塞期间无法执行任何操作只能傻傻的等），也可能超时出错；
4. 程序从buffer中取出数据，至此阻塞完毕。

改进方案：
1. 使用多线程；
2. 使用线程池。

PS：以上两种改进方案在遇到高并发时可能会遇到瓶颈。

**阻塞IO SOCKET例子**

```python
# 服务端代码
# -*- coding: utf-8 -*-#

import socket

# 1、新手机
phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

# 2、绑定手机卡
phone.bind(('127.0.0.1', 8080))  # 0-65535:0-1024给操作系统使用

# 3、监听
phone.listen(5)

# 4、等待建链
print('starting...')
conn, client_addr = phone.accept()

# 5、收，发消息
data = conn.recv(1024)  # 1、单位：bytes 2、1024代表最大接收1024个bytes
print('客户端的数据', data)

conn.send(data.upper())

# 6、挂电话
conn.close()

# 7、关机
phone.close()


# 客户端代码
# -*- coding: utf-8 -*-#

import socket

client = socket.socket()

# 建链
client.connect(('127.0.0.1', 8080))

while 1:
    res = input(">>:").encode('utf-8')
    client.send(res)
    data = client.recv(1024)

    print(data.decode('utf-8'))




```


#### 非阻塞IO

具体步骤：  
1. 程序执行 recvfrom/recv/send等网络操作，触发系统调用；
2. os判断数据是否到来，如果数据没有到来，抛出异常；
3. 程序捕获异常，继续干其他的事情；
4. 重复1-3步骤。

**非阻塞IO SOCKET例子**

```python
# 服务端
# -*- coding: utf-8 -*-#
import socket

server = socket.socket()
# 避免端口占用异常
server.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
server.bind(('127.0.0.1', 8080))
server.listen(5)

# 设置为非阻塞模式
server.setblocking(False)

# 已建链客户端
r_list = []
# 要发送数据的客户端
w_list = {}

while 1:
    try:
        conn, addr = server.accept()
        r_list.append(conn)
    # 捕获因为非阻塞而抛出的异常
    except BlockingIOError:
        # 遍历读列表，依次取出套接字读取内容
        del_rlist = []
        for conn in r_list:
            # 捕获因为非阻塞而抛出的异常
            try:
                data = conn.recv(1024)
                if not data:
                    conn.close()
                    del_rlist.append(conn)
                    continue
                w_list[conn] = data.upper()
            except BlockingIOError:  # 没有收到数据，继续遍历下一个客户端
                continue
            except ConnectionResetError:  # 客户端链接断开，加入删除列表，等待被清除
                conn.close()
                del_rlist.append(conn)

        # 遍历要发送数据的客户端，依次取出套接字发送内容
        del_wlist = []
        for conn, data in w_list.items():
            # 捕获因为非阻塞而抛出的异常
            try:
                conn.send(data)
                del_wlist.append(conn)
            except BlockingIOError:
                continue

        # 清理无用的套接字,无需再监听它们的IO操作
        for conn in del_rlist:
            r_list.remove(conn)

        for conn in del_wlist:
            w_list.pop(conn)



# 客户端

# -*- coding: utf-8 -*-#

import socket

client = socket.socket()
client.connect(('127.0.0.1', 8080))

while 1:
    res = input('>>: ').strip().encode('utf-8')
    client.send(res)
    data = client.recv(1024)

    print(data.decode('utf-8'))


```

非阻塞IO缺点：
1. CPU占用率高（我的电脑光运行服务端就25的cpu占用）；
1. 无法及时切换到其他活。

#### 多路复用IO

> 使用select模块不断轮询所有建立好的socket，当某个socket数据就绪了，就告知用户进程。

步骤：
1. 程序select触发系统调用，os监视所有select中的套接字，一旦某一个socket数据就绪，select即返回，这是一次IO（系统调用）；
2. 程序recv上一步选中的socket数据，触发系统调用（一次IO）；
3. 和阻塞IO比起来，多路复用IO其实多了一步IO，如果连接的socket不是很多的话，可能还不如多线程+阻塞IO效率高。多路复用IO的优势在于能处理更多的socket，而不是对单个链接处理的更快。

**多路复用IO SOCKET例子**


```python
# 服务端
# -*- coding: utf-8 -*-#

from socket import *
import select

server = socket(AF_INET, SOCK_STREAM)
server.bind(('127.0.0.1', 8080))
server.listen(5)
server.setblocking(False)
print('starting...')

rlist = [server, ]
wlist = []
wdata = {}

while True:
    # input/output/异常列表/timeout
    rl, wl, xl = select.select(rlist, wlist, [], 5)
    print(wl)
    for sock in rl:
        if sock == server:
            conn, addr = sock.accept()
            rlist.append(conn)
        else:
            try:
                data = sock.recv(1024)
                if not data:
                    sock.close()
                    rlist.remove(sock)
                    continue
                wlist.append(sock)
                wdata[sock] = data.upper()
            except Exception:
                sock.close()
                rlist.remove(sock)

    for sock in wl:
        sock.send(wdata[sock])
        wlist.remove(sock)
        wdata.pop(sock)


# 客户端
# -*- coding: utf-8 -*-#

import socket

client = socket.socket()
client.connect(('127.0.0.1', 8080))

while 1:
    res = input('>>: ').strip().encode('utf-8')
    client.send(res)
    data = client.recv(1024)

    print(data.decode('utf-8'))

```


