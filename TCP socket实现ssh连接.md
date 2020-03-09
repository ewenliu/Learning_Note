#### 分析

本服务端代码需要在linux上运行。

客户端连接上服务端后，向服务端发送linux命令，服务端能正常接收并且按照linux的终端结果返回给客户端。

难点：粘包问题的解决


#### 服务端逻辑和解决粘包问题思路

服务端逻辑：

1. 与客户端建立连接；
2. 接收客户端发过来的命令，命令一般都比较短（比如ls、ifconig这种），不太会超过默认的1024个字节；
3. 调用subprocess在本地终端执行，获取结果；
4. 将结果返回给客户端。


解决粘包问题简单思路：
1. 考虑到ifconfig等的执行结果长度的不可预测性，极容易产生粘包现象；
2. 服务端分两次发送，先将长度发送给客户端，客户端接收长度后，再将具体数据发送给客户端。

上面这种思路看起来貌似可行，但是长度也是一个不可预测的值，有没有一种更靠的呢？

优化1：
1. 考虑到python有一个struct的模块，它可以将有限长度的数据变成一个长度为4的固定bytes类型数据；
2. 但是还是仍然有长度限制，即使使用 q 格式（对应c语言long long型），也有可能不够用。

优化2：
1. 制作一个header字典，将要传过去的长度存放在字典的键值对中；
2. 使用json打包header；
3. 使用struct打包已经json化的header长度；
4. 客户端先接收json长度，再接收json header；
5. 从header中取到真正的数据长度，开始按照这个长度接收数据。

#### 具体代码

```python
# 服务端
# -*- coding: utf-8 -*-#
import socket
import struct
import json
import subprocess

phone = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
# 避免出现io占用情况
phone.setsockopt(socket.SOL_SOCKET, socket.SO_REUSEADDR, 1)

phone.bind(('127.0.0.1', 8080))

phone.listen(5)

while True:
    conn, addr = phone.accept()
    while True:
        cmd = conn.recv(1024)
        # 回收无效套接字，这里因为是在linux上跑ssh，所以就没必要去捕获windows下的ConnectionResetError了
        if not cmd:
            break

        res = subprocess.Popen(cmd.decode('utf-8'),
                               shell=True,
                               stdout=subprocess.PIPE,
                               stderr=subprocess.PIPE)
        err = res.stderr.read()

        if err:
            back_msg = err
        else:
            back_msg = res.stdout.read()

        headers = {'data_size': len(back_msg)}
        head_json = json.dumps(headers)
        head_json_bytes = bytes(head_json, encoding='utf-8')

        # 先发json header长度
        conn.send(struct.pack('i', len(head_json_bytes)))
        # 再发送json header
        conn.send(head_json_bytes)
        # 再发送真实数据
        conn.sendall(back_msg)

    conn.close()


# 客户端
# -*- coding: utf-8 -*-#
from socket import *
import struct
import json

ip_port = ('127.0.0.1', 8080)
client = socket(AF_INET, SOCK_STREAM)
client.connect(ip_port)

while True:
    cmd = input('>>: ')
    if not cmd:
        continue
    client.send(bytes(cmd, encoding='utf-8'))

    # 收json header长度
    head = client.recv(4)
    head_json_len = struct.unpack('i', head)[0]
    # 收json header
    head_json = json.loads(client.recv(head_json_len).decode('utf-8'))
    # 拿到真实数据长度
    data_len = head_json['data_size']

    recv_size = 0
    recv_data = b''
    # 开始接收数据，直到收完为止
    while recv_size < data_len:
        recv_data += client.recv(1024)
        # 收集长度，考虑到以后打日志可能会用
        recv_size += len(recv_data)

    print(recv_data.decode('utf-8'))

```




