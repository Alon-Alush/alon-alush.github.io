---
title: "Explaining encrypted strings in .exe files"
classes: wide
header:
  teaser: /assets/images/unpacking/MinGW/demo.png
ribbon: DodgerBlue
description: "Simple explanation on encryped strings in .exe files, and how to get around them when solving crackmes"
categories:
  - Unpacking
toc: true
---

# Encrypted strings


Have you ever tried solving a crackme, only to be baffled as to why the strings like `"Wrong Password!` don't appear in x64dbg's string references search?

The answer is simple: **String encryption**.

When you search for string references in x64dbg, it looks for recognizable ASCII sequences **in the static file** (the `.exe` as it sits on disk).

This works for programs that store their strings as raw text, like this:

![ASCII representation on disk](/assets/images/unpacking/encryptedstring/demo0.png)

The corresponding hexadecimal for `"Hello, World!"` would look like:
```
48 65 6C 6C 6F 2C 20 57 6F 72 6C 64 21
```

However, real-world programs (and many crackmes) often don't store strings like that. Instead, they **encrypt** the strings — so the actual file on disk doesn't contain `"Hello, World!"`, but some meaningless-looking bytes, like this:

![Encrypted representation on disk](/assets/images/unpacking/encryptedstrings/demo1.png)

One of the most common methods used in crackmes is **XOR encryption**.  
Here’s a basic C example to show how it works:

```c
#include <stdio.h>

void decrypt(char *output, const unsigned char *encrypted, int length, unsigned char key) {
    for (int i = 0; i < length; i++) {
        output[i] = encrypted[i] ^ key;
    }
    output[length] = '\0'; // Null-terminate
}

int main() {
    // XOR encrypted version of "Hello, World!" with key 0xAA
    unsigned char encrypted[] = {
        0xE2, 0xCF, 0xC6, 0xC6, 0xC5, 0x86, 0xEA, 0xC5, 0xD8, 0xC6, 0xC5, 0xCE, 0x8B
    };
    int length = sizeof(encrypted) / sizeof(encrypted[0]);
    unsigned char key = 0xAA;

    char decrypted[100];

    decrypt(decrypted, encrypted, length, key);

    printf("Decrypted string: %s\n", decrypted);

    return 0;
}
```

Instead of writing:
```
char string[] = "Hello, World!";
```
the program stores an encrypted version of the string:
```c
unsigned char encrypted[] = {
        0xE2, 0xCF, 0xC6, 0xC6, 0xC5, 0x86, 0xEA, 0xC5, 0xD8, 0xC6, 0xC5, 0xCE, 0x8B
    };
```
At runtime, the program decrypts it back before using it.
Here’s how the encrypted bytes look on disk:


![Encrypted representation on disk](/assets/images/unpacking/encryptedstrings/demo1.png)

Hex representation:
```
E2 CF C6 C6 C5 86 EA C5 D8 C6 C5 CE 8B
```

And hopefully, you now understand why x64dbg misses those strings in search: They are simply encrypted.

In the next chapter, we'll explain how to find the decryption routines without having to rely on a simple string search.

