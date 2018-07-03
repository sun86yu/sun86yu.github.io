---
layout: post
book: true
background-image: http://ot1cc1u9t.bkt.clouddn.com/17-7-16/91630214.jpg
category: 工具
title: JAVA-PHP-Python通用AES加密
tags:
- AES
---

JAVA 版本
---

***AesEncrypt.java***

```java
package demo;

import java.nio.charset.Charset;
import java.util.Arrays;
import java.util.Random;

import javax.crypto.Cipher;
import javax.crypto.spec.IvParameterSpec;
import javax.crypto.spec.SecretKeySpec;

import org.apache.commons.codec.binary.Base64;

public class AesEncrypt {
	static Charset CHARSET = Charset.forName("utf-8");
	Base64 base64 = new Base64();
	byte[] aesKey;

	public AesEncrypt(String encodingAesKey) throws Exception {
		if (encodingAesKey.length() != 43) {
			System.out.println("加密 key 错误!");
			throw new Exception();
		} else {
			aesKey = Base64.decodeBase64(encodingAesKey + "=");
		}
	}

	// 随机生成16位字符串
	private String getRandomStr() {
		String base = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
		Random random = new Random();
		StringBuffer sb = new StringBuffer();
		for (int i = 0; i < 16; i++) {
			int number = random.nextInt(base.length());
			sb.append(base.charAt(number));
		}
		return sb.toString();
	}

	/**
	 * 对明文进行加密.
	 * 
	 * @param text
	 *            需要加密的明文
	 * @return 加密后base64编码的字符串
	 * @throws Exception
	 *             aes加密失败
	 */
	String encrypt(String text) throws Exception {
		ByteGroup byteCollector = new ByteGroup();
		byte[] textBytes = text.getBytes(CHARSET);

		byte[] randomStrBytes = this.getRandomStr().getBytes(CHARSET);

		byteCollector.addBytes(randomStrBytes);
		byteCollector.addBytes(textBytes);

		// ... + pad: 使用自定义的填充方式对明文进行补位填充
		byte[] padBytes = BlankFillEncoder.encode(byteCollector.size());

		byteCollector.addBytes(padBytes);

		// 获得最终的字节流, 未加密
		byte[] unencrypted = byteCollector.toBytes();

		try {
			// 设置加密模式为AES的CBC模式
			Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
			SecretKeySpec keySpec = new SecretKeySpec(aesKey, "AES");
			IvParameterSpec iv = new IvParameterSpec(aesKey, 0, 16);
			cipher.init(Cipher.ENCRYPT_MODE, keySpec, iv);

			// 加密
			byte[] encrypted = cipher.doFinal(unencrypted);

			// 使用BASE64对加密后的字符串进行编码
			String base64Encrypted = base64.encodeToString(encrypted);

			return base64Encrypted;
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}
	}

	/**
	 * 对密文进行解密.
	 * 
	 * @param text
	 *            需要解密的密文
	 * @return 解密得到的明文
	 * @throws Exception
	 *             aes解密失败
	 */
	String decrypt(String text) throws Exception {
		byte[] original;
		try {
			// 设置解密模式为AES的CBC模式
			Cipher cipher = Cipher.getInstance("AES/CBC/NoPadding");
			SecretKeySpec key_spec = new SecretKeySpec(aesKey, "AES");
			IvParameterSpec iv = new IvParameterSpec(Arrays.copyOfRange(aesKey,
					0, 16));
			cipher.init(Cipher.DECRYPT_MODE, key_spec, iv);

			// 使用BASE64对密文进行解码
			byte[] encrypted = Base64.decodeBase64(text);

			// 解密
			original = cipher.doFinal(encrypted);
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}

		String content;
		try {
			// 去除补位字符
			byte[] bytes = BlankFillEncoder.decode(original);

			// 分离16位随机字符串,网络字节序和AppId
			byte[] result = Arrays.copyOfRange(bytes, 16, bytes.length);

			content = new String(result, CHARSET);
		} catch (Exception e) {
			e.printStackTrace();
			throw e;
		}

		return content;
	}

	public static void main(String[] args) throws Exception {
		// System.out.println("加密后");
		AesEncrypt tools = new AesEncrypt(
				"abcdefghijklmnopqrstuvwxyz0123456789ABCDEFG");
		String replyMsg = "testaesfunction";

		String enc = tools.encrypt(replyMsg);
		System.out.println(enc);

		String toDec = "uMWuzLg0A1mm7wKM3ipY9W72+EZg/s5OWJZHSgYMO7geCv8u2k/xYn+Qzkrr4U26";
		String dec = tools.decrypt(toDec);
		System.out.println(dec);
	}
}

```

用到的其它辅助类:

***ByteGroup.java***

```java
package demo;

import java.util.ArrayList;

class ByteGroup {
	ArrayList<Byte> byteContainer = new ArrayList<Byte>();

	public byte[] toBytes() {
		byte[] bytes = new byte[byteContainer.size()];
		for (int i = 0; i < byteContainer.size(); i++) {
			bytes[i] = byteContainer.get(i);
		}
		return bytes;
	}

	public ByteGroup addBytes(byte[] bytes) {
		for (byte b : bytes) {
			byteContainer.add(b);
		}
		return this;
	}

	public int size() {
		return byteContainer.size();
	}
}

```

***BlankFillEncoder.java***

```java
package demo;

import java.nio.charset.Charset;
import java.util.Arrays;

/**
 * 提供基于PKCS7算法的加解密接口.
 */
class BlankFillEncoder {
	static Charset CHARSET = Charset.forName("utf-8");
	static int BLOCK_SIZE = 32;

	/**
	 * 获得对明文进行补位填充的字节.
	 * 
	 * @param count 需要进行填充补位操作的明文字节个数
	 * @return 补齐用的字节数组
	 */
	static byte[] encode(int count) {
		// 计算需要填充的位数
		int amountToPad = BLOCK_SIZE - (count % BLOCK_SIZE);
		if (amountToPad == 0) {
			amountToPad = BLOCK_SIZE;
		}
		// 获得补位所用的字符
		char padChr = chr(amountToPad);
		String tmp = new String();
		for (int index = 0; index < amountToPad; index++) {
			tmp += padChr;
		}
		return tmp.getBytes(CHARSET);
	}

	/**
	 * 删除解密后明文的补位字符
	 * 
	 * @param decrypted 解密后的明文
	 * @return 删除补位字符后的明文
	 */
	static byte[] decode(byte[] decrypted) {
		int pad = (int) decrypted[decrypted.length - 1];
		if (pad < 1 || pad > 32) {
			pad = 0;
		}
		return Arrays.copyOfRange(decrypted, 0, decrypted.length - pad);
	}

	/**
	 * 将数字转化成ASCII码对应的字符，用于对明文进行补码
	 * 
	 * @param a 需要转化的数字
	 * @return 转化得到的字符
	 */
	static char chr(int a) {
		byte target = (byte) (a & 0xFF);
		return (char) target;
	}

}

```

PHP 版本
---

***AesEncrypt.php***

```php
<?php

namespace App\Services\Aes;

class AesEncrypt
{
    private $key;

    public function __construct($key)
    {
        $this->key = base64_decode($key . "=");
    }

    /**
     * 对明文进行加密
     * @param string $text 需要加密的明文
     * @return string 加密后的密文
     */
    public function encrypt($text)
    {
        try {
            // 生成随机盐值
            $random = $this->getRandomStr();
            $text = $random . $text;

            $pad = ord($text[strlen($text) - 1]);
            if ($pad > 0 && $pad <= 16) {
                $text = substr($text, 0, -$pad);
            }

            $iv = substr($this->key, 0, 16);

            $encrypted = openssl_encrypt($text, 'AES-256-CBC', $this->key, OPENSSL_RAW_DATA, $iv);
            //使用BASE64对加密后的字符串进行编码
            return base64_encode($encrypted);
        } catch (Exception $e) {
            print $e->getMessage();
            return null;
        }
    }

    /**
     * 对密文进行解密
     * @param string $encrypted 需要解密的密文
     * @return string 解密得到的明文
     */
    public function decrypt($encrypted)
    {
        try {
            //使用BASE64对需要解密的字符串进行解码
            $iv = substr($this->key, 0, 16);
            $ciphertext_dec = base64_decode($encrypted);

            $decrypted = openssl_decrypt($ciphertext_dec, 'AES-256-CBC', $this->key, OPENSSL_RAW_DATA | OPENSSL_ZERO_PADDING, $iv);
        } catch (Exception $e) {
            return array(ErrorCode::$IllegalBuffer, null);
        }

        try {
            //去除补位字符
            $pkc_encoder = new BlankFillEncoder;
            $result = $pkc_encoder->decode($decrypted);

            //去除16位随机字符串
            if (strlen($result) < 16)
                return "";

            $result = substr($result, 16, strlen($result));
        } catch (Exception $e) {
            return null;
        }

        return $result;
    }

    /**
     * 随机生成16位字符串
     * @return string 生成的字符串
     */
    function getRandomStr()
    {

        $str = "";
        $str_pol = "ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789abcdefghijklmnopqrstuvwxyz";
        $max = strlen($str_pol) - 1;
        for ($i = 0; $i < 16; $i++) {
            $str .= $str_pol[mt_rand(0, $max)];
        }
        return $str;
    }
}
```

***BlankFillEncoder.php***

```php
<?php
/**
 * Func: BlankFillEncoder.php
 * User: sunyu
 * Date: 2018/7/3
 * Time: 上午9:58
 */

namespace App\Services\Aes;

class BlankFillEncoder
{
    public static $block_size = 32;

    /**
     * 对需要加密的明文进行填充补位
     * @param $text 需要进行填充补位操作的明文
     * @return 补齐明文字符串
     */
    function encode($text)
    {
        $text_length = strlen($text);
        //计算需要填充的位数
        $amount_to_pad = BlankFillEncoder::$block_size - ($text_length % BlankFillEncoder::$block_size);
        if ($amount_to_pad == 0) {
            $amount_to_pad = BlankFillEncoder::block_size;
        }
        //获得补位所用的字符
        $pad_chr = chr($amount_to_pad);
        $tmp = "";
        for ($index = 0; $index < $amount_to_pad; $index++) {
            $tmp .= $pad_chr;
        }
        return $text . $tmp;
    }

    /**
     * 对解密后的明文进行补位删除
     * @param decrypted 解密后的明文
     * @return 删除填充补位后的明文
     */
    function decode($text)
    {
        $pad = ord(substr($text, -1));
        if ($pad < 1 || $pad > 32) {
            $pad = 0;
        }
        return substr($text, 0, (strlen($text) - $pad));
    }

}
```

Python 版本
---

***AesEncrypt.py***

```python
#!/usr/bin/env python
#-*- encoding:utf-8 -*-

import base64
import string
import random
from Crypto.Cipher import AES
import sys
reload(sys)
sys.setdefaultencoding('utf-8')

"""
关于Crypto.Cipher模块，ImportError: No module named 'Crypto'解决方案
请到官方网站 https://www.dlitz.net/software/pycrypto/ 下载pycrypto。
下载后，按照README中的“Installation”小节的提示进行pycrypto安装。
"""

class BlankFillEncoder():
    """提供基于PKCS7算法的加解密接口"""

    block_size = 32

    def encode(self, text):
        """ 对需要加密的明文进行填充补位
        @param text: 需要进行填充补位操作的明文
        @return: 补齐明文字符串
        """
        text_length = len(text)
        # 计算需要填充的位数
        amount_to_pad = self.block_size - (text_length % self.block_size)
        if amount_to_pad == 0:
            amount_to_pad = self.block_size
        # 获得补位所用的字符
        pad = chr(amount_to_pad)
        return text + pad * amount_to_pad

    def decode(self, decrypted):
        """删除解密后明文的补位字符
        @param decrypted: 解密后的明文
        @return: 删除补位字符后的明文
        """
        pad = ord(decrypted[-1])
        if pad < 1 or pad > 32:
            pad = 0
        return decrypted[:-pad]


class AesEncrypt(object):

    def __init__(self, key, addSaltFlg):
        self.addSalt = addSaltFlg
        self.key = base64.b64decode(key+"=")
        # 设置加解密模式为AES的CBC模式
        self.mode = AES.MODE_CBC

    def encrypt(self, text):
        """对明文进行加密
        @param text: 需要加密的明文
        @return: 加密得到的字符串
        """

        # 16位随机字符串添加到明文开头
        if self.addSalt:
            text = self.get_random_str() + text

        pad = ord(text[len(text) - 1])

        if 0 < pad <= 16:
            text = text[0:-pad]

        # 使用自定义的填充方式对明文进行补位填充
        pkcs7 = BlankFillEncoder()
        text = pkcs7.encode(text)

        # 加密
        cryptor = AES.new(self.key, self.mode, self.key[:16])
        try:
            ciphertext = cryptor.encrypt(text)
            # 使用BASE64对加密后的字符串进行编码
            return base64.b64encode(ciphertext)
        except Exception, e:
            print e
            return None

    def decrypt(self,text):
        """对解密后的明文进行补位删除
        @param text: 密文
        @return: 删除填充补位后的明文
        """
        try:
            cryptor = AES.new(self.key, self.mode, self.key[:16])
            # 使用BASE64对密文进行解码，然后AES-CBC解密
            plain_text = cryptor.decrypt(base64.b64decode(text))
        except Exception, e:
            print e
            return None
        try:
            pad = ord(plain_text[-1])

            content = plain_text
            if self.addSalt:
                content = plain_text[16:-pad]
        except Exception, e:
            print e
            return None
        return content

    def get_random_str(self):
        """ 随机生成16位字符串
        @return: 16位字符串
        """
        rule = string.letters + string.digits
        str = random.sample(rule, 16)
        return "".join(str)

pc = AesEncrypt('abcdefghijklmnopqrstuvwxyz0123456789ABCDEFG', True)

sReplyMsg = '我是一只小小小小鸟222'

encrypt = pc.encrypt(sReplyMsg)
decstr = pc.decrypt(encrypt)

print encrypt
print decstr

```

>上面三个版本都在明文前加了一个 16 位的字符串。作为盐值。这样，每次加密的结果就不一样。解密后，把前 16 位抛弃就得到原内容了。
>
>可以把代码改造一下，在构造函数加入是否加盐值的选项。然后在加密和解密的时候根据该值决定是否加减 16 位。或者其它位数的内容。

测试:

```
$tool = new AesEncrypt('abcdefghijklmnopqrstuvwxyz0123456789ABCDEFG');
$todo = 'testaesfunction';

$enc = $tool->encrypt($todo);
$dec = $tool->decrypt($enc);

print_r($dec . PHP_EOL);
print_r($enc . PHP_EOL);
```

把 JAVA 加密后的值放到 PHP 里解密，也是可以成功的。