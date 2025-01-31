+++
title = 'Task 1: Reverse Engineering'
date = 2025-01-30T13:09:06+05:30
author = 'Akhil T'
weight = 1
draft = false
+++

## Android Binaries
### L01.apk
First, I went with the android binary L01.apk. I used an online decompiler called [Decopiler.com](https://www.decompiler.com/) to decompile the apk file. The decompiled files are available [here](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk).
In the [MainActivity.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/uncrackable1/MainActivity.java) file, I found that an alert box is made and its value is set to "This is the correct secret." or "That's not it. Try again." by checking a.a(obj).
![MainActivity.java of L01.apk](/cybersec-writeup/reverse-engineering/mainactivity-L01.png)
We can see that a.a is a function in [a.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/a/a.java) file. The function uses aes decryption algorithm and gets the flag from the encrypted string.
The [a.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/uncrackable1/a.java) inside the uncrackable1 directory contains the encrypted message and the key used for decryption.
![a.java of L01.apk](/cybersec-writeup/reverse-engineering/a-L01.png)
I asked ChatGPT to decrypt the message. The prompt is as follows:
```
  package sg.vantagepoint.a;

import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;

public class a {
    public static byte[] a(byte[] bArr, byte[] bArr2) {
        SecretKeySpec secretKeySpec = new SecretKeySpec(bArr, "AES/ECB/PKCS7Padding");
        Cipher instance = Cipher.getInstance("AES");
        instance.init(2, secretKeySpec);
        return instance.doFinal(bArr2);
    }
}

bArr = sg.vantagepoint.a.a.a(b("8d127684cbc37c17616d806cf50473cc"), Base64.decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=", 0));

I need to get the value of bArr
```

It gave a java program to decrypt the message. The code is as follows:
```java
import javax.crypto.Cipher;
import javax.crypto.spec.SecretKeySpec;
import java.util.Base64;

class AESDecrypt {
    public static void main(String[] args) throws Exception {
        byte[] key = hexStringToByteArray("8d127684cbc37c17616d806cf50473cc");
        byte[] encryptedData = Base64.getDecoder().decode("5UJiFctbmgbDoLXmpL12mkno8HT4Lv8dlat8FxR2GOc=");

        byte[] decryptedData = decryptAES(key, encryptedData);
        System.out.println(new String(decryptedData)); // Output the decrypted value
    }

    public static byte[] decryptAES(byte[] key, byte[] data) throws Exception {
        SecretKeySpec secretKeySpec = new SecretKeySpec(key, "AES");
        Cipher cipher = Cipher.getInstance("AES/ECB/PKCS5Padding");
        cipher.init(Cipher.DECRYPT_MODE, secretKeySpec);
        return cipher.doFinal(data);
    }

    public static byte[] hexStringToByteArray(String hex) {
        int len = hex.length();
        byte[] data = new byte[len / 2];
        for (int i = 0; i < len; i += 2) {
            data[i / 2] = (byte) ((Character.digit(hex.charAt(i), 16) << 4)
                                  + Character.digit(hex.charAt(i+1), 16));
        }
        return data;
    }
}
```

I ran this code in [TuxCode](https://tuxcode.fosscell.org) which is an online compiler made by me, Rehan Tadpathri and Harshan during CodeInit 2025.
![Decrypted Message](/cybersec-writeup/reverse-engineering/tuxcode-L01.png)

Thus I got the flag for L01.apk as *`I want to believe`*.

### L02.apk
Next, I went with the android binary L02.apk. I used the same online decompiler called [Decopiler.com](https://www.decompiler.com/) to decompile the apk file. The decompiled files are available [here](https://www.decompiler.com/jar/486c43bb79154a8f947f77d0e54efec3/L02.apk).
In the [MainActivity.java](https://www.decompiler.com/jar/486c43bb79154a8f947f77d0e54efec3/L02.apk/sources/sg/vantagepoint/uncrackable2/MainActivity.java) file, I found that an alert box is made and its value is set to "This is the correct secret." or "That's not it. Try again." by checking m.a(obj).
![MainActivity.java of L02.apk](/cybersec-writeup/reverse-engineering/mainactivity-L02.png)
In the top of the file we can see that a variable m is initialized with CodeCheck type which is defined in the [CodeCheck.java](https://www.decompiler.com/jar/486c43bb79154a8f947f77d0e54efec3/L02.apk/sources/sg/vantagepoint/uncrackable2/CodeCheck.java) file. But the interesting part is the ```System.loadlibrary("foo")``` which is used to load a shared library named foo.
![MainActivity.java of L02.apk showing shared library loading](/cybersec-writeup/reverse-engineering/mainactivity-L02-foo.png)
We can find the shared libraries in the [lib](https://www.decompiler.com/jar/486c43bb79154a8f947f77d0e54efec3/L02.apk/resources/lib) directory. The shared library foo.so for each architecture is available in it. I took the [x86_64 version](https://www.decompiler.com/jar/486c43bb79154a8f947f77d0e54efec3/L02.apk/resources/lib/x86_64/libfoo.so) of it and used an online decompiler called [DogBolt](https://dogbolt.com/) to decompile the shared library. The decompiled files are available [here](https://dogbolt.org/?id=8996e9d9-7a99-4b67-ad49-518d13332721#Hex-Rays=115).

![Decompiled shared library foo.so of L02.apk](/cybersec-writeup/reverse-engineering/foo-L02.png)
In the decompiled file generated by Hex-Rays, in the 115th line, we can see the flag which is *```Thanks for all the fish```*.
