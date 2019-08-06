## 描述

人肉破解wifi，不来一发吗？

nc pwn.jarvisoj.com 9893

## 题解

先试试这个命令，得到

```
$ nc pwn.jarvisoj.com 9893
Welcome to HuWang Bei WPA2 Simulation System.. Initilizing Parameters..

SSID = HuWang

PSK = ycJgSYFHOXhSeLE7

AP_MAC = 5A:25:99:0A:37:79

AP_Nonce = 9335164f237d012644fdbaf9d47c12375a12a926a79a94fcddd4e014faa13e90

STA_MAC = 78:96:82:29:A8:B0

STA_Nonce = a732a078d96679f025ffce60548002e2f4ff8bfed7cda281b5be49b01bcc29cf

CCMP Encrypted Packet = 88423a0178968229a8b05a25990a37795a25990a3779609200002a12002054000000e3a8b00a91fbd94b2cf507c61ca3ffc69fab716a5616e3b5c46aea172035ff19141ae8746569eced33

Input decrypted challenge value in Packet:
```

完全不了解 WPA2。

搜了几篇文章：

- 802.11无线网络基础介绍：https://zhuanlan.zhihu.com/p/51695002
- WIFI：https://ctf-wiki.github.io/ctf-wiki/misc/traffic/protocols/WIFI-zh/

上面那些英文简写都能在第一篇文章找到。

题解：https://xz.aliyun.com/t/2892#toc-10

使用 https://github.com/REMath/80211_Cryptography/blob/master/80211_Crypto.ipynb 的代码

```python
from binascii import a2b_hex, b2a_hex, a2b_qp
from pbkdf2 import PBKDF2
import hmac
from hashlib import sha1
import struct
from Crypto.Cipher import AES

def PRF512(key,A,B):
    blen = 64
    R    = ''
    for i in range(0,4):
        hmacsha1 = hmac.new(key,A+B+chr(i),sha1)
        R = R+hmacsha1.digest()
    return R[:blen]


def frame_type(packet):
    header_two_bytes = struct.unpack("h", (packet[0:2]))[0]
    fc_type = bin(header_two_bytes)[-8:][4:6]
    if fc_type == "10":
        return "data"
    else:
        return None

def compute_pairwise_master_key(preshared_key, ssid):
    return PBKDF2(preshared_key, ssid, 4096).read(32)

def compute_message_integrity_check(pairwise_transient_key,data):
    return hmac.new(pairwise_transient_key[0:16],data,sha1).digest()[0:16]

def compute_pairwise_transient_key(pairwise_master_key, A, B):
    return PRF512(pairwise_master_key, A, B)

ssid = raw_input("SSID = ")
preshared_key = raw_input("PSK = ")
mac_access_point = a2b_hex(''.join(raw_input("AP_MAC = ").split(':')))
a_nonce = a2b_hex(raw_input("AP_Nonce = "))
mac_client = a2b_hex(''.join(raw_input("STA_MAC = ").split(':')))
s_nonce = a2b_hex(raw_input("STA_Nonce = "))
ccmp = raw_input("CCMP Encrypted Packet = " )

A = "Pairwise key expansion" + '\x00'
B = min(mac_access_point,mac_client)+max(mac_access_point,mac_client)+min(a_nonce,s_nonce)+max(a_nonce,s_nonce)

pairwise_master_key = compute_pairwise_master_key(preshared_key, ssid)
pairwise_transient_key = compute_pairwise_transient_key(pairwise_master_key, A, B)

key_confirmation_key = pairwise_transient_key[0:16]
key_encryption_key = pairwise_transient_key[16:16*2]
temporal_key = pairwise_transient_key[16 * 2:(16 * 2) + 16]
mic_authenticator_tx = pairwise_transient_key[16 * 3:(16 * 3) + 8]
mic_authenticator_rx =  pairwise_transient_key[(16 * 3) + 8:(16 * 3) + 8 + 8]

packet_103_encrypted_total_packet = ccmp
packet_103_encrypted_total_packet = a2b_hex(packet_103_encrypted_total_packet)
packet_103_encrypted_data = packet_103_encrypted_total_packet[34:34+84]

ccmp_header = packet_103_encrypted_total_packet[26:26 + 8]
ieee80211_header = packet_103_encrypted_total_packet[0:26]
source_address = packet_103_encrypted_total_packet[10:16]

PN5 = ccmp_header[7]
PN4 = ccmp_header[6]
PN3 = ccmp_header[5]
PN2 = ccmp_header[4]
PN1 = ccmp_header[1]
PN0 = ccmp_header[0]

last_part_of_nonce = PN5 + PN4 + PN3 + PN2 + PN1 + PN0

flag = a2b_hex('01')
qos_priorty = a2b_hex('00')

nonce_ = qos_priorty + source_address + last_part_of_nonce
IV = flag + nonce_

class WPA2Counter(object):
    def __init__(self, secret):
        self.secret = secret
        self.current = 1
    def counter(self):
        count = a2b_hex(struct.pack('>h', self.current).encode('hex'))
        i = self.secret + count
        self.current += 1
        return i

counter = WPA2Counter(IV)
crypto = AES.new(temporal_key, AES.MODE_CTR, counter=counter.counter)
test = packet_103_encrypted_data[0:-8]
print(crypto.decrypt(test))
```

然后就得到了类似这样的输出

```
Challenge Vlaue: fq8vO2mX6kLLLojn
```

将后面那串字符再输入回去即可得到 flag。

## 答案

flag{cf2e57e956660eaecf1cd920c6f2e057}
