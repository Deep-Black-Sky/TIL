# urllib with tor
torネットワークを経由してwebスクレイピング

## TL;TR
- IPBan対策
- PySocksを使う

## 実装例
```bash
$ tor
```

```python
import urllib.request, socket, socks

socks.setdefaultproxy(socks.PROXY_TYPE_SOCKS5, '127.0.0.1', 9050)
socket.socket = socks.socksocket

res = urllib.request.urlopen('https://api.ipify.org?format=json').read()
```
