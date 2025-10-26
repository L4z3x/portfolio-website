---
title: "Innocent : linux forensic challenge"
published: 2025-10-25
description: 'a detailed write-up of my first linux forensic challenge from nextrace event.'
image: ''
tags: ["DFIR","Easy","MyChallenges"]
category: 'CTF'
draft: false 
lang: 'en'
---

## Description
zohir wanted to learn hacking, like REAL hacking, so he installed kali and joined a russian hacking telegram group, and started chatting with one of the group members, after 2 days he came to me saying that suspicious files are appearing on his machine 
can you help him find out what is going on?  
files provided :[here](https://github.com/n3xusss/nexus_nextrace_2025/tree/main/Forensics/Innocent/challenge)
> **Author** : L4Z3X


we have three artifacts the log folder, the pcap file and the home dir of the victim.<br/>
First, we take a look at the `auth.log` file located in the log folder, there we can see that there are a lot of failed ssh attampts from `192.168.56.5` which means there was a ssh bruteforce attack and after that we can see that the attacker succesfully logged via ssh:
```
2025-10-13T05:25:23.462609-04:00 kali sshd-session[227351]: Accepted password for zohir from 192.168.56.5 port 39866 ssh2
```
from here we then proceed to the /home/zohir dir and take a look at the `.bash_history` file,
```bash
   23  ls
   30  git clone https://github.com/l4z3x/better-prompt/
   33  rm -rf better-prompt
   36  ls
``` 

## Investigating the better-prompt repo
the file was intentioally tempered with some commands removed, but we can see that the attacker cloned the repo `better-prompt` from github, so we go ahead and take a look at the repo [here](https://github.com/l4z3x/better-prompt/). reading the code we can't find anything suspicious, so let's inspect the previous commits.<br/> the commit with the message "bp2" has a suspicious script that listens (on port 1337) for commands from a remote server and decrypts them with aes 128 cbc with the key from an ENV variable named `KEY` and the iv hardcoded with the value `0123456789abcdef` and then it executs the command and encrypts the output with same parameters and finally sends it back to the server, the commit also contained the executable version of the script under `dist` folder 
<br/>
## Investigating the `.bashrc` and `.bash_aliases` files
now let's look for the KEY variable and where the script is being executed, going back to zohir folder, we see a file named `.prompt`, it's likely related to the better-prompt repo, running `file` cmd on it we can see that it's a ELF 64-bit executable,this confirms that it's the malicious program.<br/>
inspecting the `.bashrc` file we can see at the end the key used for aes encryption which is `oushou_n_tkhsayt`, we know that `.profile` and `.bash_aliases` are often sourced in `.bashrc`, in our case we only have `.bash_aliases` sourced, so we go ahead and inspect the `.bash_aliases` file:
```bash
./.prompt > /dev/null 2>/dev/null &
```
basically what this line does is that it runs the malicious `.prompt` file (from the better-promt repo) in the background and redirects both stdout and stderr to /dev/null, so we can conclude that the `.prompt` file is being executed every time zohir open the terminal, now to the last part is extracting the communication between the attacker and the victim.

## Analyzing the pcapng file
we open the pcapng file in wireshark and filter the traffic with `tcp.port == 1337` since it's the port used in the script, then we can see the communication between the attacker and the victim, from there we can follow the tcp stream and extract the commands and the output, run it on the decryptor script and get the flag.
NOTE: you have the install the required libraries
```bash
python -m venv env
source env/bin/activate
pip install pycryptodome
```
and then
```python
import base64
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad

KEY = b"oushou_n_tkhsayt"
IV = b"0123456789abcdef"

def aes_decrypt_b64(ciphertext_b64: str) -> str:
    try:
        ciphertext = base64.b64decode(ciphertext_b64)
        cipher = AES.new(KEY, AES.MODE_CBC, IV)
        plaintext_padded = cipher.decrypt(ciphertext)
        plaintext = unpad(plaintext_padded, AES.block_size)
        return plaintext.decode("utf-8", errors="replace")
    except Exception as e:
        return f"<decrypt error: {e}>"
```

#### using scapy
scapy is a powerful python library used for packet manipulation, we can use it to extract the commands and the output from the pcapng file, we can use the following script to do that:
```python
import json
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64
try:
    from scapy.all import rdpcap, TCP
except ImportError:
    print("Scapy is not installed. Please install it using 'pip install scapy'")
    print("use .env")
    print("Exiting...")
    exit(1)
packet = rdpcap("../../Innocent/traffic.pcapng")

filtered_packets = [
    pkt
    for pkt in packet
    if TCP in pkt and (pkt[TCP].dport == 1337 or pkt[TCP].sport == 1337)
]
payloads = []
for pkt in filtered_packets:
    if pkt[TCP].payload:
        payloads.append(bytes(pkt[TCP].payload).decode(errors="ignore"))

ouput = []
commands = []
for payload in payloads:
    if "payload" in payload:
        j = json.loads(payload)
        commands.append(j["payload"])
    else:
        ouput.append(payload)


# decrypt the commincations

KEY = b"oushou_n_tkhsayt"

IV = b"0123456789abcdef"





def aes_decrypt(ciphertext_b64):
    try:
        ciphertext = base64.b64decode(ciphertext_b64)
        cipher = AES.new(KEY, AES.MODE_CBC, IV)
        padded_plaintext = cipher.decrypt(ciphertext)
        plaintext = unpad(padded_plaintext, 16)
        return plaintext.decode("utf-8", errors="replace")

    except Exception as e:
        print("AES decryption error: ", e)
        return ""


for i in range(len(commands)):
    print(f"> {aes_decrypt(commands[i])}")
    print(f"{aes_decrypt(ouput[i])}")
    print("-----")
```
```bash
source "you env folder"
pip install scapy
python solve.py
```
output:
```
> id
uid=1001(zohir) gid=1001(zohir) groups=1001(zohir),27(sudo),100(users)

-----
> ls
Desktop
Documents
Downloads
Music
Pictures
Public
zohir
zohir.tar
Templates
Videos

-----
> echo "you found meee"
you found meee

-----
> echo "good boooooy"
good boooooy

-----
> echo "don't ever contact a russion hacker"
don't ever contact a russion hacker

-----
> echo "i hope you learned your lesson"
i hope you learned your lesson

-----
> echo "bmV4dXN7YzJfZXhmMWx0cjR0MTBuX3ZpYV9iNHNoX2FsMTRzM3N9Cg=="
bmV4dXN7YzJfZXhmMWx0cjR0MTBuX3ZpYV9iNHNoX2FsMTRzM3N9Cg==

-----
> echo "see you next time ;)"
see you next time ;)

-----
> exit
```
base64-decode command and you'll get the flag. 
```bash
base64 -d <<< "bmV4dXN7YzJfZXhmMWx0cjR0MTBuX3ZpYV9iNHNoX2FsMTRzM3N9Cg=="                                                    
nexus{c2_exf1ltr4t10n_via_b4sh_al14s3s}
```
## Summary
an SSH brute-force attack from 192.168.56.5 resulted in a successful login to user zohir. The attacker deployed a backdoor by cloning a repository (better-prompt), running a local ELF named .prompt at shell startup, and used AES-encrypted TCP communications on port 1337 to receive commands and return results. We extracted the network traffic from the provided pcapng and decrypted the command/response exchange to recover the flag.



## Flag
> `nexus{c2_exf1ltr4t10n_via_b4sh_al14s3s}`
