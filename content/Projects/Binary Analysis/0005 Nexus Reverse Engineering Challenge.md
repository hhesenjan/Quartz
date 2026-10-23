---
title: "Nexus - Reverse Engineering Challenge"
created: 2025-12-26
modified: 2025-12-26
tags: ["CRACKMES", "RE", "IDA"]
draft: true
---

## Introduction

The [Nexus! Reverse Engineering Challenge](https://crackmes.one/crackme/694af49f0c16072f40f5a379) is an really interesting challenge which consists of reading through the assmebly code to find secret key but finding the secret key is relatively easy. In this article I'll go through finding the secret key and deep dive into the way application encrypts the secret key.

## Application Information

* **Application Name:** `nexus-lite.exe`
* **Application SHA-256:** `05BFA131FEF54FFC3DEC074FD61009118E7B7880F50CC0262F7BA25020393557`

## Reverse Engineering

Loading `nexus-lite.exe` into IDA Pro reveals the secret key in `main()` as variable `aNexusMasterKey`.

![[0005 Nexus-Reverse-Engineering-Challenge-01.png]]

Running `nexus-lite.exe` and providing the value of `aMasterKey` (`NEXUS-MASTER-KEY-2025`) results in the encrypted form of the `aNexusMasterKey` being printed out.

![[0005 Nexus-Reverse-Engineering-Challenge-02.png]]

The encrypted output of `aNexusMasterKey` increased couriousity which lead to deeper analysis and identifying `sub_140001800()` which is responsible for encryting the `aNexusMasterKey`.

![[0005 Nexus-Reverse-Engineering-Challenge-03.png]]

Further analysis shows that  `sub_140001800()` calls `sub_140001630()`, `sub_140001470()`, and `sub_1400012A0()` - all the called functions performs modifications to `aNexusMasterKey`.

![[0005 Nexus-Reverse-Engineering-Challenge-04.png]]

The `sub_140001630()` performs XOR encryption on `aNexusMasterKey`, using the character `@` as the static key.

![[0005 Nexus-Reverse-Engineering-Challenge-05.png]]

The `sub_140001470()` performs operation `((array[i] - 84) % 26 + 97)` on all lowercase characters of the XOR-encrypted `aNexusMasterKey`.

![[0005 Nexus-Reverse-Engineering-Challenge-06.png]]

The `sub_1400012A0()` applies Base64 encoding to the `aNexusMasterKey` as a final touch to add additional layer of obfuscation.

![[0005 Nexus-Reverse-Engineering-Challenge-07.png]]

Congratulations! At this point the secret key has been recovered and the encryption methods used by the application has been reverse engineerd.

## Solution

As always I highly recommend reverse engineering the application to understand the encryption methods it applies and then create a Python script which can encrypt and decrypt these strings.

```python title="main.py"
import base64

def encrypt(string):
    # Variables
    x = []
    b = ""

    # Performing XOR encryption/decryption
    for i in range(0, len(string)):
        x.append(ord(string[i]) ^ ord('@'))
    
    # Converting hex to ascii
    for i in range(0, len(x)):
        x[i] = chr(x[i])

    # Performing calculation on lowecase characters
    for i in range(0, len(x)):
        if ord(x[i]) > 60:
            x[i] = chr((ord(x[i]) - 84) % 26 + 97)

    # Base64 Decoding
    b = "".join(x)
    return base64.b64encode(b.encode("UTF-8")).decode()

def decrypt(string):
    # Base64 Decoding
    b64 = base64.b64decode(string)
    b64 = b64.decode("UTF-8")

    # Variables
    a = []
    x = []
    r = ""

    # Converting each character to hex character
    for i in range(0, len(b64)):
        a.append(hex(ord(b64[i])))
    
    # Performing calculation on lowecase characters
    for i in range(0, len(a)):
        if int(a[i], 16) > 60:
            a[i] = hex(((int(a[i], 16) - 84) % 26 + 97 ))
    
    # Performing XOR encryption/decryption
    for i in range(0, len(a)):
        x.append(int(a[i], 16) ^ ord('@'))
    
    # Displaying text in plaintext
    for i in range(0, len(x)):
        r += chr(x[i])
    
    return r

print(encrypt("NEXUS-MASTER-KEY-2025"))
print(decrypt("DgUYFRN6DQETFAUSegsFGXplY2Vo"))
```

```shell title="Output"
DgUYFRN6DQETFAUSegsFGXplY2Vo
NEXUS-MASTER-KEY-2025
```

The `encrypt()` performs actions which encrypts the string while `decrypt()` performs actions to decrypt the string. The reason these functions are separated is because multiple of actions are performed to encrypt and decrypt the strings.

## Conclusion

The `nexus-lite.exe` allowed us to recver the plaintext version of `aNexusMasterKey` from `main()`. Further analysis revealed that the application obfuscates the key using combination of XOR encryption, ROT-13 encryption, and Base64 encoding. I highly recommend reverse engineering the encryption methods the application uses instead of blindly submitting the secret key as that will help with increasing your understanding of low level code.