---
layout: post
title: Decrypting salto proaccess service operators' password
date: 2023-11-05 16:50 +0200
categories: [Hardware]
tags: [hacking, software, salto]
img_path: /assets/img/salto/
---
## Introduction
In this post I will show you how operator passwords can be recovered in cleartext from a salto proaccess services
installation. This is posible due to the fact that the hashing algorithm is not properly implemented.
This issue has already been reported to salto systems and according to them its on the road map for a fix.
Personally, I can only confirm this issue on the 4.21 version. I would classify this issue as a low security
one, as an attacker would need access to the services' database for exploitation. Even though, passwords are hashed for a 
reason.

## Background

### Salto ProAccess Service

<a href="https://saltosystems.com/es-es/soluciones/proaccess-space/" target="_blank">Salto Proaccess Space</a> is the software shipped by Salto Systems to control their electronic locking solution. 
From here users, locks, doors and readers are managed. It has a web interface were operators can connect with their user and password, is this password which can be extracted easily from the database. What is interesting about this system is 
how data is transferred  between the offline locks and the Proaccess services using the same cards that are used as keys, this is known as "data on card" and a concept of the idea can be shown in <a href="https://www.youtube.com/watch?v=FpFdnUe3rYQ" target="_blank">this</a> video. 

### Salto SysCode
The syscode is a 32 bit random value generated at the time of the service intallation, it is used for identifying each salto installation, so keys on one installation do not work on other installations. Probably, the data
on the card is ciphered using this value, but the unknown format of this data is something which still requires a lot of work. This code is used in conjunction with some keys hardcoded on the source code to "cipher" sensitive
fields on the database, for example the operators' password.

## How does the vulnerable hashing algorithm work
The algorithm using to calculate password hashes is as follows, an implementation can be found [here](https://github.com/vik0t0r/decode_operator_password_PoC.git):
1. Get syscode from db (Where it is ciphered with a static key found on the code)
2. Calculate a salt Xoring the syscode with the id operator (I guess this is done to get a different salt for each operator)
3. Convert the password string to a byte array, padded to 16 bytes
4. Cipher the password using AES with CredentialKey and CredentialIV, both found in the source code
5. Calculate the sha512 hash of the salt 
6. Xor the ciphered password with the hash of the salt
7. Store xor result on the database


## Improvement

This algorithm could easily been upgraded, simply by hashing the operators password with a salt, as usually done in today's systems.

## Conclusion

Although the severity of this vulnerability is really low, there is no reason for implementing a common hashing algorithm instead of creating a flagged one. Anyway, I contacted salto systems and they confirmed that they 
knew this issue and that a fix is on the roadmap.

## References
 1. [Proof Of Concept](https://github.com/vik0t0r/decode_operator_password_PoC.git)
