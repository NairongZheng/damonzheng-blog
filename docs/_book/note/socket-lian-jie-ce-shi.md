# socket连接测试

## 测试结果

两台开发机都有公网ip跟内网ip端口开放情况如下：

| 机器     | 公网ip           | 内网ip          | 协议  | 开放端口        |
| ------ | -------------- | ------------- | --- | ----------- |
| dev01  | 106.55.219.209 | 192.168.18.13 | udp | 40001-40015 |
| dev01  | 106.55.219.209 | 192.168.18.13 | tcp | 60001-60015 |
| street | 106.55.216.245 | 192.168.18.5  | udp | 40001-40015 |

tcp连接测试结果如下：

| 服务器    | 客户端     | 协议及采用端口   | 客户端连接addr            | 连接情况                                                      |
| ------ | ------- | --------- | -------------------- | --------------------------------------------------------- |
| dev01  | street  | tcp:60009 | 106.55.219.209:60009 | TimeoutError: \[Errno 110] Connection timed out           |
| dev01  | street  | tcp:60009 | 192.168.18.13:60009  | 成功                                                        |
| dev01  | windows | tcp:60009 | 106.55.219.209:60009 | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |
| dev01  | windows | tcp:60009 | 192.168.18.13:60009  | 成功                                                        |
|        |         |           |                      |                                                           |
| dev01  | street  | tcp:12346 | 106.55.219.209:12346 | TimeoutError: \[Errno 110] Connection timed out           |
| dev01  | street  | tcp:12346 | 192.168.18.13:12346  | 成功                                                        |
| dev01  | windows | tcp:12346 | 106.55.219.209:12346 | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |
| dev01  | windows | tcp:12346 | 192.168.18.13:12346  | 成功                                                        |
|        |         |           |                      |                                                           |
| street | dev01   | tcp:60009 | 106.55.216.245:60009 | socket.error: \[Errno 110] Connection timed out           |
| street | dev01   | tcp:60009 | 192.168.18.5:60009   | 成功                                                        |
| street | windows | tcp:60009 | 106.55.216.245:60009 | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |
| street | windows | tcp:60009 | 192.168.18.5:60009   | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |
|        |         |           |                      |                                                           |
| street | dev01   | tcp:12346 | 106.55.216.245:12346 | socket.error: \[Errno 110] Connection timed out           |
| street | dev01   | tcp:12346 | 192.168.18.5:12346   | 成功                                                        |
| street | windows | tcp:12346 | 106.55.216.245:12346 | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |
| street | windows | tcp:12346 | 192.168.18.5:12346   | ConnectionResetError: \[WinError 10054] 远程主机强迫关闭了一个现有的连接。 |

udp连接测试结果如下：

| 服务器    | 客户端     | 协议及采用端口   | 客户端连接addr            | 连接情况             |
| ------ | ------- | --------- | -------------------- | ---------------- |
| dev01  | street  | udp:40009 | 106.55.219.209:40009 | 成功               |
| dev01  | street  | udp:40009 | 192.168.18.13:40009  | 成功               |
| dev01  | windows | udp:40009 | 106.55.219.209:40009 | 成功               |
| dev01  | windows | udp:40009 | 192.168.18.13:40009  | 成功               |
|        |         |           |                      |                  |
| dev01  | street  | udp:12346 | 106.55.219.209:12346 | 卡住！！！            |
| dev01  | street  | udp:12346 | 192.168.18.13:12346  | 成功               |
| dev01  | windows | udp:12346 | 106.55.219.209:12346 | 卡住！！！            |
| dev01  | windows | udp:12346 | 192.168.18.13:12346  | 成功               |
|        |         |           |                      |                  |
| street | dev01   | udp:40009 | 106.55.216.245:40009 | 成功               |
| street | dev01   | udp:40009 | 192.168.18.5:40009   | 成功               |
| street | windows | udp:40009 | 106.55.216.245:40009 | 成功               |
| street | windows | udp:40009 | 192.168.18.5:40009   | 客户端卡住，但服务器显示连接成功 |
|        |         |           |                      |                  |
| street | dev01   | udp:12346 | 106.55.216.245:12346 | 卡住！！！            |
| street | dev01   | udp:12346 | 192.168.18.5:12346   | 成功               |
| street | windows | udp:12346 | 106.55.216.245:12346 | 卡住！！！            |
| street | windows | udp:12346 | 192.168.18.5:12346   | 客户端卡住，但服务器显示连接成功 |

## 测试代码

### tcp

server

```bash

import socket

def start_server(server_addr):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        s.bind(server_addr)
        s.listen(1)
        print('Server listening on ', server_addr)
        
        while True:
            conn, client_addr = s.accept()
            try:
                print('Connected by ', client_addr)
                data = conn.recv(1024)
                if data:
                    print('Received from client: ', data.decode())
                    conn.sendall(b'Successful connection')
            finally:
                conn.close()
    finally:
        s.close()

if __name__ == '__main__':
    # server_addr = ("0.0.0.0", 60009)
    server_addr = ("0.0.0.0", 12346)
    start_server(server_addr)
```

client

```bash

import socket

def start_client(server_addr):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    try:
        s.connect(server_addr)
        s.sendall(b'Hello, server')
        data = s.recv(1024)
        print('Received from server: ', data.decode())
    finally:
        s.close()

if __name__ == '__main__':
    # server_addr = ("106.55.219.209", 12346) # dev01
    # server_addr = ("192.168.18.13", 12346) # dev01
    # server_addr = ("106.55.216.245", 12346) # street
    server_addr = ("192.168.18.5", 12346) # street
    start_client(server_addr)
```

### udp

server

```bash
#coding=UTF-8
# udp_server.py
import socket

def start_udp_server(ip, port):
    server_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    server_socket.bind((ip, port))
    
    print("UDP server listening on", ip, ":", port)
    
    while True:
        message, client_address = server_socket.recvfrom(1024)
        print("Received message from,", client_address, ":", message.decode())
        #response_message = f"Echo: {message.decode()}"
        response_message = message.decode()
        server_socket.sendto(response_message.encode(), client_address)

if __name__ == "__main__":
    start_udp_server("0.0.0.0", 40009)
    #start_udp_server("0.0.0.0", 12346)
```

client

```bash
# udp_client.py
import socket

def send_message_to_server(server_ip, server_port, message):
    client_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    
    try:
        client_socket.sendto(message.encode(), (server_ip, server_port))
        response, server_address = client_socket.recvfrom(1024)
        print("Received response from server",server_address,":", response.decode())
    finally:
        client_socket.close()

if __name__ == "__main__":
    #server_ip = "192.168.18.13"
    #server_ip = "106.55.219.209"
    server_ip = "106.55.216.245"
    #server_ip = "192.168.18.5"
    server_port = 40009
    #server_port = 12346
    message = "Hello, UDP server!"
    
    send_message_to_server(server_ip, server_port, message)
```
