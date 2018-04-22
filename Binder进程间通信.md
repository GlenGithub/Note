# Binder进程间通信系统

------

> Android应用程序由Activity、Service、BroadcastReceiver、ContentProvider四种类型组件构成，它们可能运行在同一进程中，也可能运行在不同进程中，此外各种系统组件也运行在独立的进程中，例如，Activity管理服务AMS和PMS都运行在系统进程System进程中，那么，这些运行在不同进程中的应用程序和系统组件是如何通信的呢？

----------


> Android基于Linux内核开发，Linux提供的进程间通信：

> * 管道(Pipe)
> * 信号(Signal)
> * 消息队列(MessageQunue)
> * 共享内存(ShareMemory)
> * 套接字(Socket)
> 
> Android 进程间通信 -- Binder

    Binder进程通信机制采用CS通信方式;
    
    提供服务的进程 -> Server进程(可同时运行多个Service组件向Client进程提供服务)
    访问服务的进程 -> Client进程(可同时向多个Service组件请求服务，每个请求对应一个Client组件或者称为Service代理对象)
    
    每一个Server进程和Client进程都维护一个Binder线程池来处理进程间通信请求
    
    Server进程和Client进程依靠内核空间的binder驱动程序(设备文件/dev/binder)
    
    Service组件启动时，会将自己注册到ServiceManager组件中以便Client组件可以通过ServiceManager组件找到它
    
    ServiceManager组件称为Binder进程间通信的上下文管理者，由于它也需要和Service进程、Client进程通信，可将它看做一个特殊的Service组件。

 - Service组件实现原理
![这里写图片描述](https://img-blog.csdn.net/20180422221036591?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 - Client组件实现原理
![这里写图片描述](https://img-blog.csdn.net/20180422221103366?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 

> Binder进程间通信机制涉及了Client、Service、ServiceManager、Binder四个角色；

![这里写图片描述](https://img-blog.csdn.net/20180422205550107?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

 

>  Binder进程间通信机制的四个使用情景：

> * ServiceManager的启动过程
> * ServiceManager代理对象的获取过程
> * Service组件的启动过程
> * Service代理对象的获取过程


----------


 - **ServiceManager的启动过程**

    ServiceManager 是由init进程负责启动的，而init进程是在系统启动时启动的，脚本如下：
![这里写图片描述](https://img-blog.csdn.net/20180422215728882?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
ServiceManager对应的程序文件servicemanager，源代码目录价结构:
![这里写图片描述](https://img-blog.csdn.net/20180422220030878?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
main函数入口在service_manager.c中：
![这里写图片描述](https://img-blog.csdn.net/20180422220220326?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

    ServiceManager的启动过程由三个步骤组成：
- 调用函数binder_open打开设备文件/dev/binder,以及将它映射到本进程的地址空间
- 调用函数binder_become_context_manager将自己注册为Binder进程通信机制的上下文管理者
- 调用函数binder_loop来循环等待和处理Client进程的通信请求

    
   


----------
- **ServiceManager代理对象的获取过程**

    ServiceManager代理对象的关系图
    ![这里写图片描述](https://img-blog.csdn.net/20180422222037637?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    由于ServiceManager的句柄值恒为0，因此获取它的代理对象的过程省去了与binder驱动程序交互的过程
    Android系统在应用程序框架层的Binder库中提供了一个函数defaultServiceManager来获得一个ServiceManager代理对象
    ![这里写图片描述](https://img-blog.csdn.net/20180422222500208?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    

> 有了ServiceManager代理对象，Service组件就可以在启动过程中使用它的函数addService将自己注册到ServiceManager中，Client组件就可以使用它的函数getService来获得一个指定名称的Service组件的代理对象。


----------
- **Service组件的启动过程**

  Servie组件是在Server进程中运行，Service进程在启动时，会首先将它里面的Service组件注册到ServiceManager中，接着再启动一个Binder线程池来等待和处理Client组件的通信请求。
  
  Service组件FregService是运行在FregServer进程中的
  ![这里写图片描述](https://img-blog.csdn.net/20180422223933458?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 注册Service组件
 ![这里写图片描述](https://img-blog.csdn.net/20180422224146745?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 实际调用了BpServiceManager类的addServcie函数
 ![这里写图片描述](https://img-blog.csdn.net/20180422224325285?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
 

> Client进程和Server进程的一次进程间通信可划分5个步骤：

 > * Client进程将进程间通信数据封装成一个Parcel对象，以便可以将进程间通信数据传递给Binder驱动程序.
 > * Client进程向Binder驱动程序发送一个BC_TRANSACTION命令协议，Binder驱动程序根据协议内容找到目标Server进程之后，就会向Client进程发送一个BR_TRANSACTION_COMPLETE返回协议，表示它的进程间通信请求已经接受。Client进程接收到Binder驱动程序发送给它的BR_TRANSACTION_COMPLETE返回协议，并且对它进行处理之后，就会再次进入到 Binder驱动程序中等待目标Server进程返回进程通信结果。
 > * Binder驱动程序在向Client进程发送BR_TRANSACTION_COMPLETE返回协议的同时，也会向目标Server进程发送一个BR_TRANSACTION返回协议，请求目标Server进程处理该进程间通信请求。
 > * Server进程接收到Binder驱动程序发来的BR_TRANSACTION返回协议，并且对它进行处理之后，就会向Binder驱动程序发送一个BC_REPLY命令协议，Binder驱动程序根据协议内容找到目标Client进程之后，就会向Server进程发送一个BR_TRANSACTION_COMLETE返回协议，表示它返回的进程间通信结果已经收到了，Server进程接收到Binder驱动程序发送给它的BR_TRANSACTION_COMPLETE返回协议，并且对它进行处理之后，一次进程通信过程就结束了，接着它会再次进入到Binder驱动程序中等待下一次进程间通信请求。
 > * Binder驱动程序向Server进程发送BR_TRANSACTION_COMPLETE返回协议的同时，也会向目标Client进程发送一个BR_REPLY返回协议，表示Server进程已经处理完成它的进程间通信请求了。并且将进程间通信结果返回给它。
 
 

    Client进程和Server进程其实并不需要对BR_TRANSACTION_COMPLETE返回协议做特殊处理，这样做是为了让Client进程和Server进程在执行通信的过程中，有机会返回到用户空间去做其他事情，从而增加他们的并发处理能力。


----------

- **Service代理对象的获取过程**
FregClient进程获得运行在FregServer进程中的Service组件FregService的代理对象。
![这里写图片描述](https://img-blog.csdn.net/20180422231619689?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
ServiceManager代理对象函数getService的实现
![这里写图片描述](https://img-blog.csdn.net/20180422231858387?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
调用函数checkService来获得一个名称为name的Service组件的代理对象。
![这里写图片描述](https://img-blog.csdn.net/20180422232101414?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2dsZW4xOTQz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)


    








