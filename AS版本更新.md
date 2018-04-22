1. Android studio 无法在线升级版本的解决方法
-------------------------------

> Check Update一直提示Connection failed. Please check your network
> connection and try again

在AS安装目录 E:\Program Files\Android Studio\bin，

> 用编辑器打开 studio.vmoptions/studio64.vmoptions 或者
> studio.exe.vmoptions/studio64.exe.vmoptions，文件添加如下内容：

```
-Djava.net.preferIPv4Stack=true 
-Didea.updates.url=http://dl.google.com/android/studio/patches/updates.xml 
-Didea.patches.url=http://dl.google.com/android/studio/patches/ 
```

> 保存后，重新打开Android Studio点击Check Update就会弹出更新信息提示了

----------


> AS 检查更新后只有 Download 选项，而没有 Update and restart 的时候（我猜是版本更新的跨度比较大，我的是想从
> 2.1 直接更到 2.3，中间跨了了两大版本），需要进行增量更新，而且是夸一个版本的增量更新，比如我不能直接下载 2.1 到 2.3 的增量 jar 包。

![这里写图片描述](https://img-blog.csdn.net/20180422100239604?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

**增量包的下载**


 - 通过AS 的 Help->About 查看到自己的 build 版本号(如 143.2821654)。
![这里写图片描述](https://img-blog.csdn.net/20180422100621557?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - 查看各个 AS 版本对应的 build 版本号

  [https://dl.google.com/Android/studio/patches/updates.xml](https://dl.google.com/Android/studio/patches/updates.xml)


 - 下载增量更新包 
> https://dl.google.com/android/studio/patches/AI-FROM−TO-patch-win.jar，其中FROM为已安装版本号；TO为最新版本号；
> 
> 下载地址： 
> https://dl.google.com/android/studio/patches/AI-141.2272828-173.4670197-patch-win.jar
![这里写图片描述](https://img-blog.csdn.net/20180422102329182?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - 安装更新包 
> 将下载的更新包拷贝至Android Studio 的安装目录
![这里写图片描述](https://img-blog.csdn.net/20180422105135926?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - 打开命令行提示符，切换到Android Studio 的安装目录

> java -classpath [增量包存放路径+增量包文件名] com.intellij.updater.Runner install . 
注意：空格和最后面的那个点

```
java -classpath AI-141.2288178-145.3537739-patch-win.jar com.intellij.updater.Runner install .
```

> 最后的点”.”代表当前安装到当前目录，安装完毕后，你可以重新启动Android Studio，然后Help-》about查看当前版本
> 
> <br>
> 无法下载增量包，404错误：因为版本跨度太大，需要分多段下载，具体可参考https://dl.google.com/android/studio/patches/updates.xml中from标签所指示的版本；
> <br>
下载后无法解压，提示被JAVA锁定：原因，JAR文件放置位置错误，要放置与Android Studio同一目录下；
<br>**重点内容**
> 若找不到对应的增量包，则只能下载对应的版本了：
> [https://developer.android.google.cn/studio/index.html](https://developer.android.google.cn/studio/index.html)
> [http://www.android-studio.org/](http://www.android-studio.org/)

----------

**Android Studio安装后启动后提示**
![这里写图片描述](https://img-blog.csdn.net/20180422111131518?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

> **原因**： 在第一次安装AS，启动后，检测到电脑没有SDK。
> <br>
> **解决方法**：
> 
> 1.点击Cancel，在后续的界面指定SDK本地路径
> <br>
> 2.设置初次打开AS，不下载sdk
> 
> 在这个Android studio的安装目录下，找到下面这个文件
> `\bin\idea.properties`
> 设置初次打开AS，不检测SDK。使用记事本打开，文件末尾添加一行：
> `disable.android.first.run=true`

**Android使用国内镜像在线更新SDK**

> 由于 Google 服务器在中国大陆无法正常访问，Android SDK 无法正常更新，给安卓开发者带来诸多不便。

 - **修改 hosts 文件**

在使用 Android SDK Manager 的时候，主要会连接到两个地址 dl.google.com 和 dl-ssl.google.com，key发现这两个地址都是无法正常访问的，如何解决呢？我们可以通过修改 hosts 文件，将上面的地址定向到能正常访问的 Google 服务器。我们可以使用站长工具的超级 ping 来查找可用IP。

> 打开地址：http://ping.chinaz.com/，分别测试 dl.google.com 和 dl-ssl.google.com 的IP地址，将获取到的IP写入C:\Windows\System32\drivers\etc\hosts文件。
![这里写图片描述](https://img-blog.csdn.net/2018042211410143?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
![这里写图片描述](https://img-blog.csdn.net/20180422114841933?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

```
dl-ssl.google.com 无法ping通，dl.google.com可ping通
```

 - **使用国内镜像源**

> 先在这里推荐几个：
> mirrors.neusoft.edu.cn //东软信息学院
> ubuntu.buct.edu.cn/ubuntu.buct.cn //北京化工大学
> mirrors.opencas.cn (mirrors.opencas.org/mirrors.opencas.ac.cn) //中国科学院开源协会
> sdk.gdgshanghai.com 端口：8000 //上海GDG镜像服务器
> mirrors.dormforce.net //（栋力无限）电子科技大学
其中，强烈推荐电子科技大学的镜像源！
更多地址参考此博客
[https://blog.csdn.net/wuqilianga/article/details/78540473](https://blog.csdn.net/wuqilianga/article/details/78540473)


----------

> 启动 Android SDK Manager ，打开主界面，依次选择「Tools」、「Options…」，弹出『Android SDK Manager – Settings』窗口；
> 在『Android SDK Manager – Settings』窗口中，在「HTTP Proxy Server」和「HTTP Proxy Port」输入框内填入mirrors.neusoft.edu.cn和80，并且选中「Force https://… sources to be fetched using http://…」复选框。
> 设置完成后单击「Close」按钮关闭『Android SDK Manager – Settings』窗口返回到主界面；

![这里写图片描述](https://img-blog.csdn.net/20180422115437821?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

![这里写图片描述](https://img-blog.csdn.net/20180422115545251?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - **国内谷歌服务器IP地址**

> 进入http://ping.chinaz.com/输入g.cn
![这里写图片描述](https://img-blog.csdn.net/20180422120037460?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
> 
> 然后查询出最快的节点，复制下IP地址。如：IP：203.208.41.87  响应时间：9毫秒

在Android Studio中打开SDK Manager，点击箭头指向的红框
![这里写图片描述](https://img-blog.csdn.net/20180422120423220?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
在SDK Manager中，选择Tools->Options在HTTP Proxy Server中粘贴IP地址如刚才的IP：203.208.41.87，然后在HTTP Proxy Port中填写端口号80，并勾选”Forcus https://...“选项。
![这里写图片描述](https://img-blog.csdn.net/20180422120508679?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 ![这里写图片描述](https://img-blog.csdn.net/2018042212053332?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)



然后就可以点击install packges进行下载了。

速度 是不是很快？如果不是请换个IP再试。。。