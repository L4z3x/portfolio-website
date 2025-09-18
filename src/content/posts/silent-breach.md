---
title: "Silent Breach : Cyberdefenders Lab"
published: 2025-07-03
description: 'A detailed write-up of the Silent Breach Lab Cyberdefenders, showcasing the techniques and tools used to solve the challenge.'
image: '/silent-breach-cyberdefenders/Silent_Breach.webp'
tags: ["DFIR","Medium","Cyberdefenders"]
category: 'CTF'
draft: false 
lang: 'en'
---

> **Link**: <code>[silent-breach](https://cyberdefenders.org/blueteam-ctf-challenges/silent-breach/)</code>
 
# Overview
**Scenario**: The IMF is hit by a cyber attack compromising sensitive data. Luther sends Ethan to retrieve crucial information from a compromised server. Despite warnings, Ethan downloads the intel, which later becomes unreadable. To recover it, he creates a forensic image and asks Benji for help in decoding the files.<br/>
<br/>
**Resources:** [Windows Mail Artifacts: Microsoft HxStore.hxd (email) Research](https://boncaldoforensics.wordpress.com/2018/12/09/microsoft-hxstore-hxd-email-research/)<br/>
**Difficulty**: Medium<br/>
**Category**: Endpoint Forensics<br/>
**Points**: 25<br/>

## Question 1
> What is the MD5 hash of the potentially malicious EXE file the user downloaded?
we use [image ftk](https://www.exterro.com/ftk-product-downloads/ftk-imager-4-7-3-81) to mount the disk image
<img src="/silent-breach-cyberdefenders/mount-image.png" alt="Mounting the disk image with FTK Imager" />
then cd into the `Downloads` folder from there we can see the file `IMF-Info.pdf.exe` which is the milicious file we are looking for. 
```bash
md5sum IMF-Info.pdf.exe
336a7cf476ebc7548c93507339196abb
```

## Question 2
> What is the URL from which the file was downloaded?<br/>

for this question i use image FTK gui to open the disk image and navigate to the `Downloads` folder, when we click on  `IMF-Info.pdf.exe` file i can see another file with the name Zone.Identifier which contains the URL from which the file was downloaded.
<img src="/silent-breach-cyberdefenders/url.png" alt="URL from which the file was downloaded" />

answer: `http://192.168.16.128:8000/IMF-Info.pdf.exe`

## Question 3
> What application did the user use to download this file?<br/>

after searching the file tree i found this folder `C:\AppData\Local\Microsoft\Edge` which emidiately indicates that the user used Microsoft Edge to download the file.
answer: `Microsoft Edge`

## Question 4
> By examining Windows Mail artifacts, we found an email address mentioning three IP addresses of servers that are at risk or compromised. What are the IP addresses?<br/>

to answer this question we need to read about the `hxstore.hxd` file in the website [Windows Mail Artifacts: Microsoft HxStore.hxd (email) Research](https://boncaldoforensics.wordpress.com/2018/12/09/microsoft-hxstore-hxd-email-research/), the summary of the article is that the `hxstore.hxd` file contains emails and attachments from the .edb file which is the database file for the Windows Mail application, and it located in the `C:\Users\Administrator\AppData\Local\Packages\microsoft.windowscommunicationsapps_8wekyb3d8bbwe\LocalState\` folder.<br/>
after opening  the `strings` output of the `hxstore.hxd` file in vscode and searching for `body` i found the ips here  
<img src="//silent-breach-cyberdefenders/ips.png" alt="IPs from the email"/>

answer: `145.67.29.88, 212.33.10.112, 192.168.16.128`

## Question 5
> By examining the malicious executable, we found that it uses an obfuscated PowerShell script to decrypt specific files. What predefined password does the script use for encryption?<br/>

in this question we need to analyze the `IMF-Info.pdf.exe` file using strings command.<br/>
```bash
strings IMF-Info.pdf.exe > strings_output.txt
```
after opening the `strings_output.txt` file in vscode and searching for `PowerShell` i found this line:

<img src="/silent-breach-cyberdefenders/strings-output.png" alt="PowerShell script in strings output" />

we can see that the variable `$wy7qIGPnm36HpvjrL2TMUaRbz` holds a long b64 encoded strings_output and $9U5RgiwHSYtbsoLuD3Vf6 holds the reverse value of 1st variable.<br/>
to extract the powershell script i runned this python script:

```python
import base64
a = "long-strings" # replace this with the actual long b64 encoded string
b = a[::1] 
b = base64.b64decode(b)
with open("s.ps1","wb") as f:
    f.write(b)

```

open the script and you'll find the password.<br/>
answer: `Imf!nfo#2025Sec$`

## Question 6
> After identifying how the script works, decrypt the files and submit the secret string.<br/>

we can see that the powershell script is encripting the files using AES encryption, so we'll write a powershell script to decrypt the files, the script is as follows:<br/>
**Note**: change the path output files to your desired path
```powershell
$password = "Imf!nfo#2025Sec$"
$salt = [Byte[]](0x01,0x02,0x03,0x04,0x05,0x06,0x07,0x08)
$iterations = 10000
$keySize = 32
$ivSize = 16

$deriveBytes = New-Object System.Security.Cryptography.Rfc2898DeriveBytes($password, $salt, $iterations)
$key = $deriveBytes.GetBytes($keySize)
$iv = $deriveBytes.GetBytes($ivSize)

$encryptedFiles = @(
    "E:\\Desktop\\IMF-Secret.enc",
    "E:\\Desktop\\IMF-Mission.enc" # make sure the mounted disk image is on E or change this path accordingly
)

foreach ($encFile in $encryptedFiles) {
    $baseName = [System.IO.Path]::GetFileNameWithoutExtension($encFile)
    $outputFile = "C:\ctf\$baseName.decrypted.pdf"  # Change this to your desired output path

    $aes = [System.Security.Cryptography.Aes]::Create()
    $aes.Key = $key
    $aes.IV = $iv
    $aes.Mode = [System.Security.Cryptography.CipherMode]::CBC
    $aes.Padding = [System.Security.Cryptography.PaddingMode]::PKCS7

    $decryptor = $aes.CreateDecryptor()

    [byte[]]$cipherBytes = [System.IO.File]::ReadAllBytes($encFile)

    $inStream = New-Object System.IO.MemoryStream(, $cipherBytes)

    $cryptoStream = New-Object System.Security.Cryptography.CryptoStream($inStream, $decryptor, [System.Security.Cryptography.CryptoStreamMode]::Read)

    $buffer = New-Object byte[] $cipherBytes.Length
    $bytesRead = $cryptoStream.Read($buffer, 0, $buffer.Length)

    [System.IO.File]::WriteAllBytes($outputFile, $buffer[0..($bytesRead - 1)])

    $cryptoStream.Close()
    $inStream.Close()
}
```

after running the script we can open the IMF-Mission.decrypted.pdf and get the flag.<br/>
answer: `CyberDefenders{N3v3r_eX3cuTe_F!l3$_dOwnL0ded_fr0m_M@lic10u5_$erV3r}`

## Conclusion
This challenge was a great exercise in understanding how malware can obfuscate its actions and the importance of forensic analysis in cybersecurity. By analyzing the email artifacts, decrypting the files, and extracting the necessary information, we were able to successfully complete the challenge and retrieve the flag.
