---
title: Script Kiddie
date: 2021-01-12 00:00 +0200
categories: [CryptoHack, Diffie Hellman]
tags: [notation, misc, easy]
---

## Given source file

```python
from Crypto.Cipher import AES
import hashlib
import secrets


def header():
    print("""  _____  _  __  __ _
 |  __ \(_)/ _|/ _(_)
 | |  | |_| |_| |_ _  ___
 | |  | | |  _|  _| |/ _ \
 | |__| | | | | | | |  __/
 |_____/|_|_| |_| |_|\___|
 | |  | |    | | |
 | |__| | ___| | |_ __ ___   __ _ _ __
 |  __  |/ _ \ | | '_ ` _ \ / _` | '_ \
 | |  | |  __/ | | | | | | | (_| | | | |
 |_|  |_|\___|_|_|_| |_| |_|\__,_|_| |_|

                                        """)


def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def pkcs7_unpad(message, block_size=16):
    if len(message) == 0:
        raise Exception("The input data must contain at least one byte")
    if not is_pkcs7_padded(message):
        return message
    padding_len = message[-1]
    return message[:-padding_len]


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)
    return pkcs7_unpad(plaintext).decode('ascii')


def generate_public_int(g, a, p):
    return g ^ a % p


def generate_shared_secret(A, b, p):
    return A ^ b % p


def goodbye():
    print('Goodbye!')


def main():
    header()
    print('[-] Collecting data from Alice')
    p = int(input('> p: '))
    q = (p - 1) // 2
    g = int(input('> g: '))
    A = int(input('> A: '))
    print('[+] All data collected from Alice')

    print('[+] Generating public integer for alice')
    b = secrets.randbelow(q)
    print(f'secret b = {b}')
    B = generate_public_int(g, b, p)
    print(f'[+] Please send the public integer to Alice: {B}')
    print('')
    input('[+] Press any key to continue')
    print('')

    print('[+] Generating shared secret')
    shared_secret = generate_shared_secret(A, b, p)

    query = input('Would you like to decrypt a message? (y/n)\n')
    if query == 'y':
        iv = input('[-] Please enter iv (hex string)\n')
        ciphertext = input('[-] Please enter ciphertext (hex string)\n')
        flag = decrypt_flag(shared_secret, iv, ciphertext)
        print(f'[+] Flag recovered: {flag}')
        goodbye()
    else:
        goodbye()


if __name__ == '__main__':
    main()

```

The file looks like it is using normal Diffie Hellman key exchange. However look closely at the notion that is used to generate the keys.

```python
def generate_public_int(g, a, p):
    return g ^ a % p


def generate_shared_secret(A, b, p):
    return A ^ b % p
```

In SAGE this might've worked but we are in python. `^` means XOR not exponent. Thus, we can pretty easily get the private key back by just using XOR and the generator.
Here is the solution script for this problem.

## Solution script

```python
from Crypto.Cipher import AES
import hashlib

def is_pkcs7_padded(message):
    padding = message[-message[-1]:]
    return all(padding[i] == len(padding) for i in range(0, len(padding)))


def pkcs7_unpad(message, block_size=16):
    if len(message) == 0:
        raise Exception("The input data must contain at least one byte")
    if not is_pkcs7_padded(message):
        return message
    padding_len = message[-1]
    return message[:-padding_len]


def decrypt_flag(shared_secret: int, iv: str, ciphertext: str):
    # Derive AES key from shared secret
    sha1 = hashlib.sha1()
    sha1.update(str(shared_secret).encode('ascii'))
    key = sha1.digest()[:16]
    # Decrypt flag
    ciphertext = bytes.fromhex(ciphertext)
    iv = bytes.fromhex(iv)
    cipher = AES.new(key, AES.MODE_CBC, iv)
    plaintext = cipher.decrypt(ciphertext)
    return pkcs7_unpad(plaintext).decode('ascii')


p= 2410312426921032588552076022197566074856950548502459942654116941958108831682612228890093858261341614673227141477904012196503648957050582631942730706805009223062734745341073406696246014589361659774041027169249453200378729434170325843778659198143763193776859869524088940195577346119843545301547043747207749969763750084308926339295559968882457872412993810129130294592999947926365264059284647209730384947211681434464714438488520940127459844288859336526896320919633919
g= 2
A= 539556019868756019035615487062583764545019803793635712947528463889304486869497162061335997527971977050049337464152478479265992127749780103259420400564906895897077512359628760656227084039215210033374611483959802841868892445902197049235745933150328311259162433075155095844532813412268773066318780724878693701177217733659861396010057464019948199892231790191103752209797118863201066964703008895947360077614198735382678809731252084194135812256359294228383696551949882
B= 652888676809466256406904653886313023288609075262748718135045355786028783611182379919130347165201199876762400523413029908630805888567578414109983228590188758171259420566830374793540891937904402387134765200478072915215871011267065310188328883039327167068295517693269989835771255162641401501080811953709743259493453369152994501213224841052509818015422338794357540968552645357127943400146625902468838113443484208599332251406190345653880206706388377388164982846343351
iv= 'c044059ae57b61821a9090fbdefc63c5'
encrypted_flag= 'f60522a95bde87a9ff00dc2c3d99177019f625f3364188c1058183004506bf96541cf241dad1c0e92535564e537322d7'

# notation indicates XOR function used not pow
b = B ^ g % p

shared = b ^ A % p
flag = decrypt_flag(shared, iv, encrypted_flag)
print(f'[+] Flag recovered: {flag}')
```