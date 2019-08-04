# Traceroute
unixコマンド、"traceroute"の仕組みを理解し、Pythonで実装する。

## TL;TR
- tracerouteはパケットがどんな道筋を辿ったかを調べるツール。
- IPヘッダのTTL(Time To Live)値を増加させ、通ったルーターからの返信を見る。

## IPヘッダ
[パケット形式図参考](https://ja.wikipedia.org/wiki/IPv4#%E3%83%91%E3%82%B1%E3%83%83%E3%83%88)

IPヘッダの「生存時間」はルーターを通過するたびに減少する。
そして0になったとき現在のルーターからソースへと、パケットが送り返される。

送り返されるパケットには、送り返したルーターのipアドレスが記載されている。
そのため、パケットがどのような道順をたどり目的地に到達するかがわかる。
TTL値を目的地に到達するまで増加させていくことで、道順をトレースすることができる。

## 実装
```python
import socket
import sys

socket.setdefaulttimeout(10)

def send_packet(address, port, ttl):
    # create a packet for receive.
    recv_socket = socket.socket(socket.AF_INET, socket.SOCK_RAW, socket.getprotobyname('icmp'))
    recv_socket.bind(("", port))

    # create a packet for send.
    send_socket = socket.socket(socket.AF_INET, socket.SOCK_DGRAM, socket.getprotobyname('udp'))
    send_socket.setsockopt(socket.SOL_IP, socket.IP_TTL, ttl)
    send_socket.sendto(bytes(512), (address, port))

    try:
        name, addr = recv_socket.recvfrom(512)
        name, addr = valid_addr(addr[0])

        print("%d  %s  %s" % (ttl, name, addr))

        if address == addr:
            return 0
        else:
            return send_packet(address, port, ttl + 1)

    except socket.error:
        pass

    finally:
        recv_socket.close()
        send_socket.close()

def get_host_name(addr):
    try:
        return socket.gethostbyaddr(addr)[0]
    except socket.error:
        return addr

def valid_addr(addr):
    if addr is not None:
        return get_host_name(addr), addr
    else:
        return '*', '*'

def traceroute(dist, port):
    addr = socket.gethostbyname(dist)
    code = send_packet(addr, port, 1)
    if code is not 0:
        print('Time out.')

if __name__ == "__main__":
    traceroute(str(sys.argv[1]), int(sys.argv[2]))

```

- 送信プロトコルはUDP、受信プロトコルはICMPとした。ICMPにはエラー通知などの情報が含まれる。
- 再帰関数send_packetを呼び出すたびにttlを増加させる。
- ルーターによってはipアドレスを返さないため、ipアドレスの検証を行った(valid_addr)。
