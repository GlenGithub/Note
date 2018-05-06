#Windows上将linux目录映射网络驱动器
**转自：https://www.cnblogs.com/abc36725612/p/8183490.html**
> 我有两台PC，一台操作用的Windows，一台linux。为了方便对linux目录的文件操作。需要在Windows上将linux中的/fdsk目录映射为网络驱动器。

首先要将linux安装成为samba服务器
---------------------

 - 安装samba

```
root@pc:~# apt-get install samba
```

 - 修改配置文件

```
root@pc:~# vi /etc/samba/smb.conf
```

 - 在最后添加下面几行
![这里写图片描述](https://img-blog.csdn.net/20180506211040455?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 - 修改文件夹的权限

```
root@pc:/# chmode 777 fdsk
```

 - 重启smbd生效

```
root@pc:~# cd /etc/init.d/
root@pc:/etc/init.d# smbd restart
```

在windows上映射网络驱动器
----------------

```
Linux的静态ip为17.0.11.185
```
![这里写图片描述](https://img-blog.csdn.net/20180506211245589?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 - 映射完成
![这里写图片描述](https://img-blog.csdn.net/20180506211328124?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)