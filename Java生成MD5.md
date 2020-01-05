---
title: Java生成MD5
date: 2020-01-05 14:22:42
categories: Java
tags: 工具
---

{% asset_img image01.png  %}

<!-- more -->

# 简介

**MD5**消息摘要算法（英语：**MD5** Message-Digest Algorithm），一种被广泛使用的密码散列函数，可以产生出一个128位（16字节）的散列值（hash value），用于确保信息传输完整一致。


MD5加密算法为现在应用最广泛的哈希算法之一，该算法广泛应用于互联网网站的用户文件加密，能够将用户密码加密为128位的长整数。数据库并不明文存储用户密码，而是在用户登录时将输入密码字符串进行MD5加密，与数据库中所存储的MD5值匹配，从而降低密码数据库被盗取后用户损失的风险。

# 基于Spring实践

spring中提供了加密工具类`org.springframework.util.DigestUtils`，上手简单，可以再工具类的基础上根据自己的业务需求进行加密优化。如在加密时增加加密盐等。

```java
public class MD5Util {
  // 加密盐
  private static final String SALT = "!@#$%^&^%$sdf";
  public static String getMD5(String str) {
    Assert.notNull(str, "加密对象为空");
    String base = str + "/" + SALT;
    String md5 = DigestUtils.md5DigestAsHex(base.getBytes());
    return md5;
  }
  public static void main(String args[]) {
    MD5Util.getMD5("院长");
  }
}
```

上面demo的**加密盐**是固定的，实际项目中可以增加自己的逻辑，最好做到动态加盐。

