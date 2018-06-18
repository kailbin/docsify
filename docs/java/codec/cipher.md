
# 对称加密

## 常见加密算法

- `DES` 数据加密标准 Data Encryption Standard
- `DESede`(3DES) 3重DES
- `AES` 高级加密标准 Advanced Encryption Standard
- `IDEA` 国际数据加密算法 International Data Encryption Algorithm
- `PEB` 基于口令加密 Password Based Encryption

## 秘钥长度

- DES
    - 秘钥长度
        - 56（默认）
        - 64（Bouncy Castle 实现）
    - 工作模式
        - **ECB**
        - CBC
        - ...
    - 填充方式
        - **NoPadding**
        - **PKCS5Padding**
        - PKCS7Padding（Bouncy Castle 实现）
        - ...
- 3DES
    - 秘钥长度
        - 112
        - 168（默认）
        - 128（Bouncy Castle 实现）
        - 129（Bouncy Castle 实现）
    - 工作模式
        - **ECB**
        - CBC
        - ...
    - 填充方式
        - NoPadding
        - **PKCS5Padding**
        - PKCS7Padding（Bouncy Castle 实现）
        - ...
- AES
    - 秘钥长度
        - 128（默认）
        - 192
        - **256**（需要无政策限制权限文件）
    - 工作模式
        - **ECB**
        - CBC
        - ...
    - 填充方式
        - NoPadding
        - **PKCS5Padding**
        - PKCS7Padding（Bouncy Castle 实现）
        - ...

## 通用工具类


``` java
import org.apache.commons.codec.DecoderException;
import org.apache.commons.codec.binary.Base64;
import org.apache.commons.codec.binary.Hex;
import org.apache.commons.codec.binary.StringUtils;

import javax.crypto.*;
import javax.crypto.spec.SecretKeySpec;
import java.security.InvalidKeyException;

/**
 * KeyGenerator
 * <p>
 * <dependency>
 * <groupId>commons-codec</groupId>
 * <artifactId>commons-codec</artifactId>
 * <version>1.11</version>
 * </dependency>
 */
public class CipherUtil {

    /**
     * xUHdKxzVCbsgVIwTnc1jtpWn
     */
    public static final CipherConfig DESEDE_DESEDE_ECB_PKCS5 = new CipherConfig("DESede", "DESede/ECB/PKCS5Padding");
    public static final CipherConfig DESEDE_DESEDE = new CipherConfig("DESede", "DESede");
    /**
     * e6d1ee87cff971e0c00e9ca8626add5e
     */
    public static final CipherConfig AES_AES_ECB_PKCS5 = new CipherConfig("AES", "AES/ECB/PKCS5Padding");
    public static final CipherConfig AES_AES = new CipherConfig("AES", "AES");

    public static class CipherConfig {
        String keyAlgorithm;
        String cipherAlgorithm;
        String key;
        SecretKey secretKey;


        public CipherConfig(String keyAlgorithm, String cipherAlgorithm) {
            this.keyAlgorithm = keyAlgorithm;
            this.cipherAlgorithm = cipherAlgorithm;
        }

        public CipherConfig initHexKey(String key) {
            this.key = key;
            try {
                secretKey = new SecretKeySpec(Hex.decodeHex(key), keyAlgorithm);
            } catch (DecoderException e) {
                throw new CipherException(e);
            }
            return this;
        }

        public CipherConfig initByteKey(String key) {
            this.key = key;
            secretKey = new SecretKeySpec(StringUtils.getBytesUtf8(key), keyAlgorithm);
            return this;
        }
    }


    /**
     * 加密
     */
    public static String encrypt(String text, CipherConfig config) {
        byte[] encrypt = encrypt(StringUtils.getBytesUtf8(text), config);
        return Base64.encodeBase64String(encrypt);
    }

    /**
     * 加密
     */
    public static byte[] encrypt(byte[] text, CipherConfig config) {
        try {
            Cipher cipher = Cipher.getInstance(config.cipherAlgorithm);
            cipher.init(Cipher.ENCRYPT_MODE, config.secretKey);
            return cipher.doFinal(text);
        } catch (java.security.NoSuchAlgorithmException | BadPaddingException | IllegalBlockSizeException | NoSuchPaddingException | InvalidKeyException e1) {
            throw new CipherException(e1);
        }
    }

    /**
     * 解密
     */
    public static String decrypt(String cipherText, CipherConfig config) {
        byte[] decrypt = decrypt(Base64.decodeBase64(cipherText), config);
        return StringUtils.newStringUtf8(decrypt);
    }

    /**
     * 解密
     */
    public static byte[] decrypt(byte[] cipherText, CipherConfig config) {
        try {
            Cipher cipher = Cipher.getInstance(config.cipherAlgorithm);
            cipher.init(Cipher.DECRYPT_MODE, config.secretKey);
            return cipher.doFinal(cipherText);
        } catch (java.security.NoSuchAlgorithmException | NoSuchPaddingException | BadPaddingException | InvalidKeyException | IllegalBlockSizeException e1) {
            throw new CipherException(e1);
        }
    }

    public static class CipherException extends RuntimeException {

        public CipherException(Exception ex) {
            super(ex);
        }

        public CipherException(String message, Throwable cause) {
            super(message, cause);
        }
    }

}
```

## Read More