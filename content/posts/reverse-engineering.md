+++
title = 'Task 1: Reverse Engineering'
date = 2025-01-30T13:09:06+05:30
author = 'Akhil T'
draft = true
+++

## Android Binaries
### L01.apk
First, I went with the android binary L01.apk. I used an online decompiler called [Decopiler.com](https://www.decompiler.com/) to decompile the apk file. The decompiled files are available [here](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk).
In the [MainActivity.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/uncrackable1/MainActivity.java) file, I found that an alert box is made and its value is set to "This is the correct secret." or "That's not it. Try again." by checking a.a(obj).
![MainActivity.java of L01.apk](/reverse-engineering/mainactivity-L01.png)
We can see that a.a is a function in [a.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/a/a.java) file. The function uses aes decryption algorithm and gets the flag from the encrypted string.
The [a.java](https://www.decompiler.com/jar/bef9713f757940b590d0bfbd6111c3f0/L01.apk/sources/sg/vantagepoint/uncrackable1/a.java) inside the uncrackable1 directory contains the encrypted message and the key used for decryption.
![a.java of L01.apk](/reverse-engineering/a-L01.png)
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
![Decrypted Message](/reverse-engineering/tuxcode-L01.png)

Thus I got the flag for L01.apk as *`I want to believe`*.
