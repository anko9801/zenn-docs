---
title: "SSH プロトコルスタックを自作してみた！"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---


## 概要
SSH

## SSH とは何か
**技術者目線の説明**
パソコン間を繋げてリモートシェルを叩けるシステム。

**自作er目線の説明**
公開鍵認証してパケット送受信をするプログラム。

## どうやって作るのか
https://www.openssh.com/portable.html

```shell
$ make && make tests
```

```shell
$ make && sshd -f sshd_config -h ~/.ssh/id_rsa -p 2222 -ddd
$ sudo tcpdump -i lo -w sample.pcap
```

とりあえず次の 5 つを読むと昔の SSH の体系が分かります。

- [RFC4250: The Secure Shell (SSH) Protocol Assigned Numbers](https://datatracker.ietf.org/doc/html/rfc4250)
- [RFC4251: The Secure Shell (SSH) Protocol Architecture](https://datatracker.ietf.org/doc/html/rfc4251)
- [RFC4252: The Secure Shell (SSH) Authentication Protocol](https://datatracker.ietf.org/doc/html/rfc4252)
- [RFC4253: The Secure Shell (SSH) Transport Layer Protocol](https://datatracker.ietf.org/doc/html/rfc4253)
- [RFC4254: The Secure Shell (SSH) Connection Protocol](https://datatracker.ietf.org/doc/html/rfc4254)

規格書あるあるなのですが、規格が更新されたときはバージョン管理をあんまりせず、新しい規格を追加するので情報が取っ散らかりがちで読むのが難しいです。大体は拡張性があるように作られますが、互換性のないものも稀によくあり、最新の実装をしたいな～となったら全ての Updates に目を通しておかないといけません。

OpenSSH 規格集
https://www.openssh.com/specs.html

tex2e
https://tex2e.github.io/rfc-translater/html/

そこでここである程度纏めた情報を作って、私も自作できそうだな～と思えるくらい流れを書いておきます。

## プロトコル

https://datatracker.ietf.org/doc/html/rfc4253

### データ型

| データ型 | 説明 |
| -- | -- |
| byte |
| boolean | 1 バイトに格納される |
| uint32 |  4 バイト (network byte order = big endian) 699921578 (0x29b7f4aa) is stored as 29 b7 f4 aa. |
| uint64 | 8 バイト (network byte order = big endian) に|
| string | バイト文字列。バイト長 (uint32) と文字列を格納する。`testing` は `00 00 00 07 t e s t i n g` と格納される。 |
| mpint | 多倍長整数 (network byte order = big endian) string として格納される MSB が 1 なら負の数と見られないように先頭にゼロバイトを挿入する。 |
| name-list | `,` で区切った文字列のリスト。string として格納される。 |

```
Examples:

value (hex)        representation (hex)
-----------        --------------------
0                  00 00 00 00
9a378f9b2e332a7    00 00 00 08 09 a3 78 f9 b2 e3 32 a7
80                 00 00 00 02 00 80
-1234              00 00 00 02 ed cc
-deadbeef          00 00 00 05 ff 21 52 41 11

Examples:

value                      representation (hex)
-----                      --------------------
(), the empty name-list    00 00 00 00
("zlib")                   00 00 00 04 7a 6c 69 62
("zlib,none")              00 00 00 09 7a 6c 69 62 2c 6e 6f 6e 65
```

### パケット形式
compression, mac, encryption: none
AEAD のときは？

```
uint32    packet_length
byte      padding_length
byte[n1]  payload
byte[n2]  random padding
byte[m]   mac (Message Authentication Code - MAC)

packet = payload + padding
```
mac 以外を暗号化する。

### Version Exchange

```
SSH_protoversion_softwareversion CR LF
SSH_protoversion_softwareversion SP comments CR LF
```

SSH-2.0-OpenSSH_8.9p1 Ubuntu-3 Ubuntu-3ubuntu0.1\r\n
SSH-2.0-OpenSSH_8.2\r\n

### Kex Exchange Init

```
byte       SSH_MSG_KEXINIT
byte[16]   cookie (random bytes)
name-list  kex_algorithms
name-list  server_host_key_algorithms
name-list  encryption_algorithms_client_to_server
name-list  encryption_algorithms_server_to_client
name-list  mac_algorithms_client_to_server
name-list  mac_algorithms_server_to_client
name-list  compression_algorithms_client_to_server
name-list  compression_algorithms_server_to_client
name-list  languages_client_to_server
name-list  languages_server_to_client
boolean    first_kex_packet_follows
uint32     0 (reserved for future extension)
```

### Key Exchange

```
byte      SSH_MSG_KEXDH_INIT / SSH2_MSG_KEX_ECDH_INIT
string    Q_C
```

```
byte      SSH_MSG_KEXDH_REPLY / SSH2_MSG_KEX_ECDH_REPLY
string    server public host key and certificates (K_S)
  string  Host key type
  string  public key
string    Q_S
string    KEX host signature
  string  Host signature type
  string  signature of H
```

```
string   V_C, client's identification string (CR and LF excluded)
string   V_S, server's identification string (CR and LF excluded)
string   I_C, payload of the client's SSH_MSG_KEXINIT
string   I_S, payload of the server's SSH_MSG_KEXINIT
string   K_S, server's public host key
string   Q_C, client's ephemeral public key octet string
string   Q_S, server's ephemeral public key octet string
mpint    K,   shared secret

H = hash(V_C || V_S || I_C || I_S || K_S || Q_C || Q_S || K)
exchange hash
```
```
K1 = HASH(K || H || X || session_id)   (X is e.g., "A")
K2 = HASH(K || H || K1)
K3 = HASH(K || H || K1 || K2)
...
key = K1 || K2 || K3 || ...
```

```
Initial IV client to server      HASH(K || H || "A" || session_id)
Initial IV server to client      HASH(K || H || "B" || session_id)
Encryption key client to server  HASH(K || H || "C" || session_id)
Encryption key server to client  HASH(K || H || "D" || session_id)
Integrity key client to server   HASH(K || H || "E" || session_id)
Integrity key server to client   HASH(K || H || "F" || session_id)
```

```
byte       SSH_MSG_NEWKEYS
```

## 暗号
### 鍵交換方式 (KEX: Key Exchange)

鍵交換に使われる公開鍵暗号とハッシュ関数の 2 つを用いて

| Algorithm | 説明 | 備考 |
| -- | -- | -- |
| diffie-hellman-group1-sha1 | 1024 bit DH with SHA1 |
| diffie-hellman-group14-sha1 | 2048 bit DH with SHA1 |
| diffie-hellman-group-exchange-sha1 | Custom DH with SHA1 |
| diffie-hellman-group14-sha256 | 2048 bit DH with SHA2 | OpenSSH 7.3以降 |
| diffie-hellman-group16-sha512 | 4096 bit DH with SHA2 | OpenSSH 7.3以降 |
| diffie-hellman-group18-sha512 | 8192 bit DH with SHA2 | OpenSSH 7.3以降 |
| diffie-hellman-group-exchange-sha256 | Custom DH with SHA2 | OpenSSH 4.4以降 |
| ecdh-sha2-nistp256 | ECDH over NIST P-256 with SHA2 | OpenSSH 5.7以降 |
| ecdh-sha2-nistp384 | ECDH over NIST P-384 with SHA2 | OpenSSH 5.7以降 |
| ecdh-sha2-nistp521 | ECDH over NIST P-521 with SHA2 | OpenSSH 5.7以降 |
| curve25519-sha256@libssh.org | ECDH over Curve25519 with SHA2 | OpenSSH 6.7以降2 |
| curve25519-sha256 | ECDH over Curve25519 with SHA2 | OpenSSH 7.4以降 |
| sntrup4591761x25519-sha512@tinyssh.org | NTRU with SHA2 | OpenSSH 8.0以降 |

https://datatracker.ietf.org/doc/html/rfc9142

### 公開鍵関連
server host key

| Algorithm | 説明 | 備考 |
| -- | -- | -- |
| ssh-rsa |
| ssh-rsa-cert-v01@openssh.com |
| rsa-sha2-256 |
| rsa-sha2-256-cert-v01@openssh.com |
| rsa-sha2-512 (standard) |
| rsa-sha2-512-cert-v01@openssh.com |
| ecdsa-sha2-nistp256 |
| ecdsa-sha2-nistp256-cert-v01@openssh.com |
| ecdsa-sha2-nistp384 |
| ecdsa-sha2-nistp384-cert-v01@openssh.com |
| ecdsa-sha2-nistp521 |
| ecdsa-sha2-nistp521-cert-v01@openssh.com |
| ssh-ed25519 |
| ssh-ed25519-cert-v01@openssh.com |

https://datatracker.ietf.org/doc/html/rfc8332

### 共通鍵

| Algorithm | 説明 | 備考 |
| -- | -- | -- |
| 3des-cbc |
| aes128-cbc |
| aes192-cbc |
| aes256-cbc |
| aes128-ctr |
| aes192-ctr |
| aes256-ctr |
| aes128-gcm@openssh.com |
| aes256-gcm@openssh.com |
| arcfour |
| arcfour128 |
| arcfour256 |
| blowfish-cbc |
| cast128-cbc |
| chacha20-poly1305@openssh.com |

### MAC
mac = MAC(key, sequence_number || unencrypted_packet)

| Algorithm | 説明 | 備考 |
| -- | -- | -- |
| hmac-md5 |
| hmac-md5-96 |
| hmac-sha1 |
| hmac-sha1-96 |
| hmac-sha2-256 |
| hmac-sha2-512 |
| umac-64 |
| umac-128 |
| hmac-md5-etm@openssh.com |
| hmac-md5-96-etm@openssh.com |
| hmac-sha1-etm@openssh.com |
| hmac-sha1-96-etm@openssh.com |
| hmac-sha2-256-etm@openssh.com |
| hmac-sha2-512-etm@openssh.com |
| umac-64-etm@openssh.com (standard) |
| umac-128-etm@openssh.com |

https://datatracker.ietf.org/doc/html/rfc6668

### 圧縮方式

| Algorithm | 説明 | 備考 |
| -- | -- | -- |
| zlib@openssh.com | |

<!-- | zlib | ZLIB (LZ77) compression | -->

## 実際に作ってみて

## これから
