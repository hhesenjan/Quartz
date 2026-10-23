---
title: "Puzzle - Reverse Engineering Challenge"
created: 2025-12-13
modified: 2025-12-13
tags: ["CRACKMES", "RE", "IDA"]
draft: false
---

## Introduction

The [Puzzle Challenge](https://crackmes.one/crackme/691de1d12d267f28f69b7f16) from crackmes.one is about understanding the stack and conditional statements which are commonly used by threat actors to confuse reverse engineers. In this article I'll go through the methods I used to obtain the password and solve the challenge.

## Application Information

* **Application Name**: `puzzle.exe`
* **SHA-256** : `3BBD399E847D739F1A99CC694EF5DDF923D4B5D2178144D03FD2A2FC8DCE14E7`

## Reverse Engineering

The `main` function invokes `sub_7FF6A6EA25C0` and `sub_7FF6A6EA29F0` but only `sub_7FF6A6EA25C0` is relevant for our analysis as it's responsible for handling the input and outputs.

![[0001 Puzzle-Reverse-Engineering-Challenge-01.png]]

All the strings and characters referenced by `sub_7FF6A6EA25C0` is XOR-encrypted and decrypted at runtime prior to being displayed to the user. 

![[0001 Puzzle-Reverse-Engineering-Challenge-02.png]]

The `sub_7FF6A6EA25C0` also invokes the `_fgetchar` function to read user input and stores the result inside the stack. This value is later used for comparing the password lengths and contents. 

![[0001 Puzzle-Reverse-Engineering-Challenge-03.png]]

The password length is checked at `check_password_len` label. If the password length is 9 the execution continues along the valid path otherwise the access is denied.

![[0001 Puzzle-Reverse-Engineering-Challenge-04.png]]

At `cmp_password` the XOR-ecnrypted password is decypted byte by byte using the `9Fh` hexadecimal code after decryption the passwords are compared byte by byte.

![[0001 Puzzle-Reverse-Engineering-Challenge-05.png]]

Attach debugger to the application and inspect the `[rds + rdi]` addreess which contains the XOR-encrypted bytes. These bytes can be decrypted using the `9Fh` key. 

![[0001 Puzzle-Reverse-Engineering-Challenge-06.png]]

## Solution

Instead of restarting the application multiple of times to obtain the password, I wrote a Python script which will decrypt the XOR bytes. 

```python title="solve.py"
def main():
    xor_password = (
        210, # 0D2h
        198, # 0C6h
        207, # 0CFh
        222, # 0Deh
        204, # 0CCh
        204, # 0CCh
        174, # 0AEh
        173, # 0ADh
        172  # 0ACh
    )
 
    characters = ""
 
    for number in xor_password:
        characters += chr(number ^ 159)                                                
    
    print("Password: " + characters)
        
if __name__ == "__main__":
    main()
```

```powershell title="Output"
PS C:\Users\Student> python3 .\main.py
Password: MYPASS123
```

Python script managed to successfully decrypt the encrypted XOR bytes. I would highly recommend creating Python script to solve the reverse engineering challenges as this will significantly improve your efficiency and effectiveness over time.

## Conclusion

The [Puzzle Challenge](https://crackmes.one/crackme/691de1d12d267f28f69b7f16) is a challenge that I would highly recommend for beginners that are just getting into reverse engineering as it will help them with understanding XOR-encryption and the way threat actors stores the XOR bytes inside the stack instead of storing them in `.rodata` segment.