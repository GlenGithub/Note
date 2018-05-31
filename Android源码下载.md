# 环境准备

安装ubuntu系统，或者虚拟机

#源码下载
- **首先下载 repo 工具**

```
mkdir ~/bin
PATH=~/bin:$PATH
curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
## 如果上述 URL 不可访问，可以用下面的：
## curl https://storage-googleapis.lug.ustc.edu.cn/git-repo-downloads/repo > ~/bin/repo
chmod a+x ~/bin/repo
```
- **然后建立一个工作目录**

```
mkdir android
cd android
```

- **初始化仓库命令：**
```
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest
## 如果提示无法连接到 gerrit.googlesource.com，可以编辑 ~/bin/repo，把 REPO_URL 一行替换成下面的：
## REPO_URL = 'https://gerrit-googlesource.proxy.ustclug.org/git-repo'
```
> 编辑 ` ~/bin/repo`文件，执行命令 `gedit ~/bin/repo`：


- **如果需要某个特定的 Android 版本**

```
repo init -u git://mirrors.ustc.edu.cn/aosp/platform/manifest -b android-8.1.0_r9
```

- **同步源码树（以后只需执行这条命令来同步）**

```
repo sync
```

### 编译相关

```
cd android  //切到android目录下
lunch       //选择编译版本类型
23    		//针对不同类型选择序号，我这里选择23 user版
make -j32   //全编，-j加快编译速度
mmm packages/apps/Helloword/ //单模块编译
切换到要编译的目录下 mm -B
make clean  //清除out目录文件
```

