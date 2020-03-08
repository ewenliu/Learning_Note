> 只有TCP才会出现粘包，UDP永远不会。

#### 什么是粘包

粘包是由TCP协议本身造成的，TCP为提高传输效率，发送方往往要收集到足够多的数据后才发送一个TCP段。若连续几次需要send的数据都很少，通常TCP会根据优化算法（Nagle算法）把这些数据合成一个TCP段后一次发送出去，这样接收方就收到了粘包数据。

通俗例子：

```python

# client

client.send('xxx')
client.send('xxx')

# 由于客户端两次send的数据很少，操作系统底层会将这两次发送的数据整合成一次发送给客户端，
# 而服务端实际上并不知道客户端发送的一次数据，因为只收到了一次数据，这就是粘包。
```

1. TCP（transport control protocol，传输控制协议）是面向连接的，面向流的，提供高可靠性服务。收发两端（客户端和服务器端）都要有一一成对的socket，因此，发送端为了将多个发往接收端的包，更有效的发到对方，使用了优化方法（Nagle算法），将多次间隔较小且数据量小的数据，合并成一个大的数据块，然后进行封包。这样，接收端，就难于分辨出来了，必须提供科学的拆包机制。 即面向流的通信是无消息保护边界的；
2. UDP（user datagram protocol，用户数据报协议）是无连接的，面向消息的，提供高效率服务。不会使用块的合并优化算法，, 由于UDP支持的是一对多的模式，所以接收端的buff(套接字缓冲区）采用了链式结构来记录每一个到达的UDP包，在每个UDP包中就有了消息头（消息来源地址，端口等信息），这样，对于接收端来说，就容易进行区分处理了。 即面向消息的通信是有消息保护边界的；
3. tcp是基于数据流的，于是收发的消息不能为空，这就需要在客户端和服务端都添加空消息的处理机制，防止程序卡住，而udp是基于数据报的，即便是你输入的是空内容（直接回车），那也不是空消息，udp协议会帮你封装上消息头。


#### 粘包现象实例

1、发送端需要等缓冲区满才发送出去，造成粘包（发送数据时间间隔很短，数据了很小，会合到一起，产生粘包）。

程序逻辑：
- 客户端连续发送两次tcp数据，第一次是'hello'，第二次是'world'。  
- 服务端连续接受两次tcp数据。

最终的结果是：客户端发生粘包，tcp优化算法将两次发送的短数据组合成一次数据发送给服务端，即服务端只能接收到一次tcp数据

```python
# 服务端代码
# -*- coding: utf-8 -*-#

import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('127.0.0.1', 8080))  # 0-65535:0-1024给操作系统使用
server.listen(5)

conn, addr = server.accept()

res1 = conn.recv(1024)  # world
res2 = conn.recv(1024)  # world

print(res1)
print(res2)



# 客户端代码
# -*- coding: utf-8 -*-#

import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

client.connect(('127.0.0.1', 8080))

client.send('hello'.encode('utf-8'))
client.send('world'.encode('utf-8'))


# 运行结果

# 服务端打印
b'helloworld'
b''

# 从结果中不难发现，客户端将hello 和 world组成成一次tcp数据发出来了


```

2、接收方不及时接收缓冲区的包，造成多个包接收（客户端发送了一段数据，服务端只收了一小部分，服务端下次再收的时候还是从缓冲区拿上次遗留的数据，产生粘包）。

程序逻辑：
- 客户端发送一个长度为10的数据
- 服务端代码第一次接收长度为2，第二次接收长度为1024。

最终的结果是：服务端第一次没完全收完数据，第二次接着收缓冲区的数据，如果这时客户端再发送一次数据，那么产生粘包（没收完的数据和第二次数据粘在一起）。


```python
# 服务端代码
# -*- coding: utf-8 -*-#

import socket

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind(('127.0.0.1', 8080))  # 0-65535:0-1024给操作系统使用
server.listen(5)

conn, addr = server.accept()

res1 = conn.recv(2)  # world
res2 = conn.recv(1024)  # world

print(res1)
print(res2)


# 客户端代码
# -*- coding: utf-8 -*-#

import socket

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

client.connect(('127.0.0.1', 8080))

client.send('hello'.encode('utf-8'))
client.send('world'.encode('utf-8'))

# 运行结果

# 服务端打印
b'he'
b'lloworld'

```

#### 处理粘包的简单办法

思路：
1. 由于接收端不知道发送端的发送长度，所以粘包
2. 发送端第一次先发送数据长度，接收端接收长度
3. 发送端第二次发送真实数据，接收端开始接收，直到所接收的数据长度等于第一次接收到的数据为止。



```python
# 服务端代码
# -*- coding: utf-8 -*-#

import socket

phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
phone.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
phone.bind(('127.0.0.1', 8080))  # 0-65535:0-1024给操作系统使用
phone.listen(5)

print('starting...')
while True:  # 链接循环
    conn, client_addr = phone.accept()
    print(client_addr)

    while True:  # 通信循环
        try:
            # 1、收命令
            cmd = conn.recv(1024)
            if not cmd:
                break  # 适用于linux操作系统

            # 2、转化为大写
            print(cmd)
            data_to_send = cmd.upper()

            # 3、把结果返回给客户端
            # 第一步：制作固定长度的报头
            total_size = str(len(data_to_send))

            # 第二步：把长度发给客户端
            conn.send(total_size.encode('utf-8'))

            # 第三步：再发送真实的数据
            conn.send(data_to_send)

        except ConnectionResetError:  # 适用于windows操作系统
            break
    conn.close()

phone.close()


# 客户端代码
# -*- coding: utf-8 -*-#

import socket

phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

phone.connect(('127.0.0.1', 8080))

while True:
    # 1、发命令
    cmd = input('>>: ').strip()
    if not cmd:
        continue
    phone.send(cmd.encode('utf-8'))

    # 2、拿命令的结果，并打印

    # 第一步：先收报头
    total_size = phone.recv(4)

    # 第二步：接收真实的数据
    recv_size = 0
    recv_data = b''
    while recv_size < int(total_size):
        res = phone.recv(1024)  
        recv_data += res
        recv_size += len(res)

    print(recv_data.decode('utf-8'))

phone.close()

```

#### 处理粘包的完整办法


思路：
1. 由于数据的长度未知，所以可能接收端的recv buff不够用，当长度一定时，recv(1024)仍然接接收不完长度从而无法解决粘包；
2. 考虑使用struct对数据的长度值进行打包，固定长度为4字节，但是当长度值超过一定限度时，struct也会报错；
3. 将数据长度（数据头header可能不仅包含数据长度）先转成字典再转成json，再将json的长度值用struct打包，几乎可以满足足够大的传输数据长度。
4. 将转成json的header长度发给接收端，再发送header；
5. 客户端先接收header长度，再接收header，从header中剥离出数据长度，再接收数据。


```python
#服务端代码
# -*- coding: utf-8 -*-#
import socket
import struct
import json

phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
phone.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)
phone.bind(('127.0.0.1', 9909))  # 0-65535:0-1024给操作系统使用
phone.listen(5)

print('starting...')
while True:  # 链接循环
    conn, client_addr = phone.accept()
    print(client_addr)

    while True:  # 通信循环
        try:
            # 1、收命令
            cmd = conn.recv(8096)
            if not cmd:
                break  # 适用于linux操作系统

            # 2、转化为大写
            data_to_send = cmd.upper()

            # 3、把结果返回给客户端
            # 第一步：制作固定长度的报头
            header_dic = {
                'filename': 'xxx',
                'md5': 'xxx',
                'total_size': len(data_to_send)
            }

            header_json = json.dumps(header_dic)

            header_bytes = header_json.encode('utf-8')

            # 第二步：先发送报头的长度
            conn.send(struct.pack('i', len(header_bytes)))

            # 第三步：再发报头
            conn.send(header_bytes)

            # 第四步：再发送真实的数据
            conn.send(data_to_send)

        except ConnectionResetError:  # 适用于windows操作系统
            break
    conn.close()

phone.close()


#客户端代码
# -*- coding: utf-8 -*-#
import socket
import struct
import json

phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

phone.connect(('127.0.0.1', 9909))

while True:
    # 1、发命令
    cmd = input('>>: ').strip()
    if not cmd:
        continue
    phone.send(cmd.encode('utf-8'))

    # 2、拿命令的结果，并打印

    # 第一步：先收报头的长度
    obj = phone.recv(4)
    header_size = struct.unpack('i', obj)[0]

    # 第二步：再收报头
    header_bytes = phone.recv(header_size)

    # 第三步：从报头中解析出对真实数据的描述信息
    header_json = header_bytes.decode('utf-8')
    header_dic = json.loads(header_json)
    print(header_dic)
    total_size = header_dic['total_size']

    # 第四步：接收真实的数据
    recv_size = 0
    recv_data = b''
    while recv_size < total_size:
        res = phone.recv(1024)
        recv_data += res
        recv_size += len(res)

    print(recv_data.decode('utf-8'))

phone.close()
```
