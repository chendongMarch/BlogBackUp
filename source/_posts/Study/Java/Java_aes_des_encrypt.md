---
layout: post
title: '加密解密（DES,AES）'
categories:
  - Study
  - Java
tags: Java
keywords:
  - Java
  - DES
  - AED
  - 加密
abbrlink: 3556316359
date: 2015-08-15 00:00:00
---

加密解密的原理是个很麻烦的问题，我之前上过一门课叫密码学，最后也没怎么学懂，所以这里我们只是使用java代码实现加密解密的功能，而不是讨论他的原理。

<!--more-->

## DES简单介绍

- DES(Data Encryption Standard)即数据加密标准，使用56bit密钥，将64bit的明文数据块加密为64bit密文。


- DES使用56bit密钥加密，秘钥要求8个字节64bit，每个字节有一位是奇偶校验位。


- DES加密强度小，容易被破解 

## DES加密代码实现
```java
/**
     * DES加密解密
     * @param data 需要加密的数据
     * @param key 秘钥，必须是8个字节
     * @param mode 加密或者解密模式
     *             Cipher.ENCRYPT_MODE
     *             Cipher.DECRYPT_MODE
     * @return
     */
    public static byte[] DESCrypt(byte[] data, byte[] key, int mode) {
        byte[] result = null;
        if (data == null || key == null || data.length <= 0 || key.length <= 0)
            return null;
        //DES密码必须是8个字节，64bit长度
        try {
            //创建加密引擎
            Cipher cipher = Cipher.getInstance("DES");
            //指定8个字节密码
            DESKeySpec desKeySpec = new DESKeySpec(key);
            //生成密码工厂
            SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DES");
            //生成密码
            SecretKey secretKey = secretKeyFactory.generateSecret(desKeySpec);
            //设置模式，加密解密
            cipher.init(mode, secretKey);
            //加密。设置字节数组作为待加密内容,返回值是最终加密结果
            result = cipher.doFinal(data);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (InvalidKeySpecException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        }
        return result;
    }

```


## AES简单介绍
- AES(Advanced Encryption Standard)，即为高级加密标准。在密码学中又称Rijndael加密法，是美国联邦政府采用的一种区块加密标准。这个标准用来替代原先的DES，已经被多方分析且广为全世界所使用。AES的区块长度固定为128 比特，密钥长度则可以是128，192或256比特；

## 低强度加密代码实现
```java
/**
     * AES低强度加密解密
     * @param data
     * @param key
     * @param mode
				   Cipher.ENCRYPT_MODE
     *             Cipher.DECRYPT_MODE
     * @return
     */
    public static byte[] AESCrypt(byte[] data,byte[] key,int mode){
        byte[] result = null;
        if (data == null || key == null || data.length <= 0 || key.length <= 0)
            return null;
        try {
            Cipher cipher = Cipher.getInstance("AES");
            SecretKeySpec secretKeySpec = new SecretKeySpec(key,"AES");
            cipher.init(mode,secretKeySpec);
            result = cipher.doFinal(data);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        }
        return result;
    }
```
### 高强度加密，使用iv参数

```java
/**
     * AES高强度加密
     *
     * @param data
     * @param key
     * @param iv   用于AES/CBC、PKCS5Padding这个带有加密模式的算法
     * @param mode
     * @return
     */
    public static byte[] AESCrypt(byte[] data, byte[] key, byte[] iv, int mode) {
        byte[] result = null;
        if (data == null || key == null || data.length <= 0 || key.length <= 0)
            return null;
        try {
            Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
            SecretKeySpec secretKeySpec = new SecretKeySpec(key, "AES");
            //准备iv参数，用于支持CBC或者ECB模式
            IvParameterSpec ivParameterSpec = new IvParameterSpec(iv);
            cipher.init(mode, secretKeySpec, ivParameterSpec);
            result = cipher.doFinal(data);
        } catch (NoSuchAlgorithmException e) {
            e.printStackTrace();
        } catch (NoSuchPaddingException e) {
            e.printStackTrace();
        } catch (InvalidKeyException e) {
            e.printStackTrace();
        } catch (BadPaddingException e) {
            e.printStackTrace();
        } catch (IllegalBlockSizeException e) {
            e.printStackTrace();
        } catch (InvalidAlgorithmParameterException e) {
            e.printStackTrace();
        }
        return result;
    }

```



