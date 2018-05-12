Android P 行为变更
--------------

> **Android P 功能和 API: https://developer.android.google.cn/preview/features**

> 对运行在 Android P 设备上的所有应用都有影响的关键变化。

 - 对于非 SDK 接口的限制
> 现已禁止访问特定的非 SDK 接口，无论是直接访问，还是通过 JNI 或反射进行间接访问。尝试访问受限制的接口将会生成 NoSuchFieldException 和 NoSuchMethodException 之类的错误；
在 Developer Preview 1 中，您的应用可以继续访问这些受限的接口；该平台通过 toast 和日志条目提醒您注意这些接口。 如果您的应用显示这样的 toast，则必须寻求受限接口之外的其他实现策略。 

 - 移除加密提供程序
> 从 Android P 开始，Crypto JCA 提供程序现已被移除。调用 SecureRandom.getInstance("SHA1PRNG", "Crypto") 将会引发 NoSuchProviderException。

 - 更严格的 UTF-8 解码器
> 在 Android P 中，针对 Java 语言的 UTF-8 解码器比以往更严格，并且遵循 Unicode 标准。
 1. 原先UTF-8 字节序列可由 String(byte[] bytes) 之类的 String 构造函数解码
 2. 在 Android P 中解码修改后的 UTF-8/CESU-8 序列，请使用 DataInputStream.readUTF() 函数或 NewStringUTF() JNI 函数。

 - 禁止空闲应用访问摄像头、麦克风和传感器
> 在应用处于空闲状态时，不能再访问摄像头、麦克风或 SensorManager 传感器。

 - 后台应用中的输入和数据隐私

    Android P 通过限制后台应用访问用户输入和传感器数据的能力增强了隐私性。 如果您的应用在运行 Android P 的设备上在后台运行，系统将对您的应用施加以下限制：
   -  您的应用不能访问麦克风或摄像头。
   -  使用连续报告模式的传感器（例如加速度计和陀螺仪）不会接收事件
   -  使用发送变化时或一次性报告模式的传感器不会接收事件。

    <br>
    如果您的应用需要在运行 Android P 的设备上检测传感器事件，请使用前台服务。
    注：对 SensorManager 的某个实例调用 flush() 的应用不会受此变更影响。

 - 强制执行 FLAG_ACTIVITY_NEW_TASK 要求
> 在 Android P 中，您不能从非 Activity 环境中启动 Activity，除非您传递 Intent 标志 FLAG_ACTIVITY_NEW_TASK。 如果您尝试在不传递此标志的情况下启动 Activity，则该 Activity 不会启动，系统会在日志中输出一则消息。

 - 网络地址查询可能会导致网络违规
StrictMode 类是一个有助于开发者检测代码问题的开发工具。 现在，StrictMode 可以检测需要名称解析的网络地址查询所导致的网络违规。
> 开发者在交付应用时不应启用 StrictMode。 否则，其应用可能会遭遇新的异常，例如，在使用 detectNetwork() 或
> detectAll() 函数获取用于检测网络违规的政策时，会出现 NetworkOnMainThreadException。

    解析数字 IP 地址不被视为阻塞性操作。 数字 IP 地址解析的工作方式与 Android P 以前的平台版本中所采用的方式相同。

 - 套接字标记
> 在 Android P 以前的平台版本上，如果使用 setThreadStatsTag() 函数标记某个套接字，则当使用带ParcelFileDescriptor 容器的 binder IPC 将其发送给其他进程时，套接字会被取消标记。
<br>
从 Android P 开始，利用 binder IPC 将套接字发送至其他进程时，其标记将得到保留。 此变更可能影响网络流量统计，例如，使用queryDetailsForUidTag() 函数时。 您可以通过先调用 untagSocket() 然后再将套接字发送至其他进程，保留以前的行为。

 - 报告的套接字中可用字节数
> 在调用 shutdownInput() 函数后调用 available() 函数会返回 0。

 - 更详尽的 VPN 网络功能报告
> 从 Android P 开始，当 VPN 调用 setUnderlyingNetworks() 函数时，Android 系统将会合并任何底层网络的传输和能力并返回 VPN 网络的有效网络能力作为结果。

 - 应用无法再访问 xt_qtaguid 文件夹中的文件
> 不再允许应用直接读取 /proc/net/xt_qtaguid 文件夹中的文件

 - 屏幕旋转变更

    |屏幕方向|行为|
    |--|--|
    |未指定、user|在自动屏幕旋转和旋转锁定下，Activity 可以纵向或横向（以及颠倒纵向或横向）呈现。 预期同时支持纵向和横向布局。|
    |userLandscape|在自动屏幕旋转和旋转锁定下，Activity 可以横向或颠倒横向呈现。 预期只支持横向布局。|
    |userPortrait|	在自动屏幕旋转和旋转锁定下，Activity 可以纵向或颠倒纵向呈现。 预期只支持纵向布局。|
    |fullUser|在自动屏幕旋转和旋转锁定下，Activity 可以纵向或横向（以及颠倒纵向或横向）呈现。 预期同时支持纵向和横向布局;<br><br>旋转锁定用户将可选择锁定到颠倒纵向，通常为 180º。|
    |sensor、fullSensor、sensorPortrait、sensorLandscape|忽略旋转锁定模式首选项，视为自动屏幕旋转已启用。 请仅在例外情况下并经过仔细的用户体验考量后再使用此项。


 - 前台服务
> 针对 Android P或更高版本并使用前台服务的应用必须请求FOREGROUND_SERVICE 权限。这是普通权限，因此，系统会自动为请求权限的应用授予此权限。
<br>
> 如果针对 Android P 的应用尝试创建一个前台服务且未请求 FOREGROUND_SERVICE，则系统会引发 SecurityException。

 - 设备串行访问限制
> Build.SERIAL 字段在 Android 8.0（API 级别 26）中已被弃用。 在 Android P 中，Build.SERIAL 始终设置为 "UNKNOWN"。 此变更旨在保护用户的隐私。<br><br>
如果您的应用需要访问设备的硬件序列号，您应请求 READ_PHONE_STATE 权限，然后调用 getSerial()。

 - 视图焦点
> - 0 面积的视图（即宽度或高度为 0）再也不能被聚焦。
> - Activity 不再隐式分配触摸模式下的初始焦点， 而是由您显式请求初始焦点

 - 不允许共享 WebView 数据目录
 > 现在，不允许应用在不同进程之间共享一个 WebView 数据目录。如果您的应用有多个进程使用 WebView、CookieManager 或 android.webkit 软件包中的任何其他 API，则在第二个进程调用 WebView 函数时，您的应用将会崩溃。

 - SELinux 禁止访问应用的数据目录
 > 系统强制每个应用的 SELinux 沙盒对每个应用的私有数据目录强制执行逐个应用的 SELinux 限制。现在，不允许直接通过路径访问其他应用的数据目录。应用可以继续使用进程间通信 (IPC) 机制（包括通过传递 FD）共享数据。

 
 
 
 
 
 
 