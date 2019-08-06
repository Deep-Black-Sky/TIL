# Known plaintext attack
暗号化されたzipファイルの脆弱性

## TL;TR
- 暗号化されたzipファイルは、それに含まれる同一のファイルを用いて復号できる。
- `pkcrack -C <暗号化されたzip> -c <含まれるファイル名> -p <ファイル> -d <復号されたzip>`
