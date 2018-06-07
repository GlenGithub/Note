# Android APN设置

### 概述

> Apn设置,即“接入点名称”设置，Apn的全称是Access PointName，是用户在通过手机上网时必须配置的一个参数，它决定了您的手机通过哪种接入方式来访问移动网络。
> 
> 对于移动终端用户来说，可以访问的外部网络类型有很多，例如：Internet、WAP网站、集团企业内部网络、行业内部专用网络。而不同的接入点所能访问的范围以及入的方式是不同的，网络侧如何知道移动终端激活以后要访问哪个网络从而分配哪个网段的 IP呢，这就要靠 APN来区分了，即 APN决定了用户的移动终端通过哪种接入方式来访问什么样的网络。
> 
> 常见的 APN有：中国移动的 `cmnet` 和 `cmwap`、中国联通的`uninet` 和 `uniwap`、中国电信的 `ctnet` 和 `ctwap`



### 前言

android4.0以后Google把`android.permission.WRITE_APN_SETTINGS`权限归为系统权限，常规应用无法进行添加修改，需要系统签名为系统应用（[应用系统签名](https://mp.csdn.net/mdeditor/80328851)）。

**解决办法：**
- 需要在Android系统源码的环境下用make来编译：
	- 在应用程序的AndroidManifest.xml中的manifest节点中加入`android:sharedUserId="android.uid.system"`这个属性。
	- 修改Android.mk文件，加入`LOCAL_CERTIFICATE := platform`这一行
	- 使用`mm`命令来编译，生成的apk就有修改系统时间的权限了。
- 直接把eclipse或AS编出来的apk用系统的签名文件签名
	- 加入`android:sharedUserId="android.uid.system"`这个属性。
	- 使用eclipse或AS编译出apk文件。
    - 使用目标系统的platform密钥来重新给apk文件签名。首先找到密钥文件，在我android源码目录中的位置是`"build/target/product/security"`，下面的`platform.pk8`和`platform.x509.pem`两个文件。然后用Android提供的Signapk工具来签名，signapk的源代码是在`"build/tools/signapk"`下，编译后在`out/host/linux-x86/framework`下，用法为`java -jar signapk.jar  platform.x509.pem platform.pk8 input.apk output.apk"`。

**相关知识点参考**

[Android系统权限和root权限](https://blog.csdn.net/superkris/article/details/7709504)

[APN基础常识](https://blog.csdn.net/sjz4860402/article/details/78522871)

[插卡后APN信息的加载流程](http://lib.csdn.net/article/android/56028?knId=297)

[三大运营商上网设置](https://wenku.baidu.com/view/36bf9d19c281e53a5802fff6.html)

[Android 的APN设定与上网处理](https://wenku.baidu.com/view/1717c76c192e45361066f57a.html)

[中国电信物联网专网APN设置说明](https://wenku.baidu.com/view/c59862420029bd64793e2c2f.html)

**附加知识点参考**

[Android创建和使用数据库](https://blog.csdn.net/chaoyu168/article/details/50260829)

[创建数据库到SD卡](https://www.2cto.com/kf/201509/444324.html)

[Android与.Net交互模拟用户屏幕操作添加APN和网络4G/3G切换](https://www.cnblogs.com/yesicoo/p/4459828.html)



### APN Uri介绍

`content://telephony/carriers`代表的是APN数据库的位置，所有的APN都在这个数据库中。

- `content://telephony/carriers` //取得全部apn列表 
- `content://telephony/carriers/preferapn` //取得当前设置的
- `content://telephony/carriers/current` //取得current=1的apn列表
- `content://telephony/carriers/restore` //恢复默认设置


### 三大运营商MNC MCC

| 运营商      |    MNC | MCC  |
| :--------: | :--------:| :--: |
| 电信  | 03、05 、11(4g)|  460   |
| 联通  | 01、06、09|  460   |
| 移动  | 00、02、04、07|  460   |
| 铁通  | 20|  460   |



