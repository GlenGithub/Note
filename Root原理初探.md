***下载recovery刷机包***

1. adb连接手机 `adb devices` 查看设备是否连上
2. `adb reboot bootloader` 进入bootloader模式 （同时按电源键和音量-键）
3. recover执行`fastboot`命令
  

>  **注意：**    
**危险命令1**  `fastboot flash bootloader bootloader.img`（镜像不匹配会导致手机主板瘫痪，无法维修，只能换主板）   
**危险命令2**  自毁命令 `fastboot erase bootloader` ,手机变砖    除了bootloader分区 其他都可做测试

 1. 正确命令 `fastboot flash recovery my-recovery.img`
 2. 启动设备 `fastboot reboot`

**Edify 语言编写脚本语言**


> 执行`adb reboot recovery` 进入recovery模式 
> 或者在bootloader模式下可通过音量上下键进入正常的系统还是recovery模式 可将升级包放在SD卡中 选择SD卡升级模式进行升级

**在APP中提取root权限**
```Runtime.getRuntime().exec("su");
OutputStream os = process.getOutputStream();
os.write("ls /system/app".getBytes());
os.flush();
os.close();
```

**修改Android设备的启动动画**

> 只需要替换/system/media目录下的bootanimation.zip文件即可

------