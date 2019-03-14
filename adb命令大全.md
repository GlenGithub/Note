执法仪rom版本为user版本，联系厂家提供root版的boot.img替换，重新刷机

打开飞行模式(需要ROOT权限)
settings put global airplane_mode_on 1
am broadcast -a android.intent.action.AIRPLANE_MODE --ez state true

关闭飞行模式(需要ROOT权限)
settings put global airplane_mode_on 0
am broadcast -a android.intent.action.AIRPLANE_MODE --ez state false

MTU值设置与查看(需要ROOT权限)
adb shell
ip link set dev ccmni0 mtu 1300 (ip link set dev X mtu N 敲回车 X=网卡名称 N=mtu值)
查看网卡配置信息
ifconfig ccmni0

mtu值测试
ping -l 数据包大小 -f 192.168.1.1

查看某个应用的版本信息
adb shell dumpsys package 包名

apk权限查看
aapt dmup badging xxx.apk

输出CPU信息
adb shell dumpsys cpuinfo

查看某个包内存使用信息
adb shell dumpsys meminfo [processName]

查看某个包activity使用情况
adb shell dumpsys activity [processName] | find '过滤字符'

获取按键信息
adb shell getevent

获取系统属性
adb shell getprop

设置系统属性(需要ROOT权限)
adb shell setprop 

查看PM帮助信息
adb shell pm

查看手机内安装包列表
adb shell pm list packages

查找monkey进程信息
adb shell ps | find "monkey"

杀进程
adb shell kill [pid]

重启
adb shell reboot

打开SVC帮助界面
adb shell svc

查询wifi操作帮助
adb shell svc wifi
关闭wifi
adb shell svc wifi disable
打开wifi
adb shell svc wifi enable

擦除数据恢复出厂
adb shell wipe data

启动指定activity
adb shell am start -n <package_name>/.<activity_class_name>
输出安装包apk路径
adb shell pm path <package>
删除与包相关的所有数据；清除数据和缓存
adb shell pm clear <package>
启动SERVICE -n表示组件 -a表示service的action
adb shell am startservice -n <package_name>/.<service_class_name>
adb shell am startservice -a 'android.intent.action.CALL'
发送广播 --es String参数类型  --ei int参数类型 --ez boolean类型
adb shell am broadcast -a <broadcast_action> --es test_string 'this is a testing'


查看am帮助信息
adb shell am

屏幕录像 ctrl+c停止录像 --size 800*600 设置视频分辨率
adb shell screenrecord /mnt/sdcard/1.mp4
将SD卡中的录像文件导出F盘
adb pull /mnt/sdcard/1.mp4 f:\

截图
adb shell screencap /mnt/sdcard/1.png

查看分辨率
adb shell wm size

打印log
adb shell logcat -s TAG
过滤等级 
logcat *:V | *:E
输出log缓存到文件
adb logcat -v time > log.txt

settings命令用法
adb shell
settings get secure default_input_method 获取默认输入法
settings put secure default_input_method com.sohu.inputmethod.sogouoem/.SogoIME 设置搜狗输入法默认
settings get system screen_off_timeout 获取屏幕休眠时间

模拟按下HOME键
adb shell input keyevent 26
模拟按下返回键
adb shell input keyevent 4

启动DDMS
ddms

显示当前运行的全部模拟器
adb devices

对某一模拟器执行命令
adb -s 模拟器编号 命令

安装应用
adb install -r apk

卸载应用
adb uninstall 包名

获取模拟器中的文件
adb pull <remote> <local>

向模拟器中写入文件
adb push <local> <remote>

查看后台服务信息
adb shell service list

查看wifi mac地址
adb shell cat /sys/class/net/wlan0/address

手机未root，应用debug模式（android:debuggable="false"）
adb shell run-as <package> 直接进入/data/data/pacakge/目录下
cat data.db > /mnt/sdcard/test.db  cat命令把数据库拷贝到sd卡下

操作DB
adb shell
sqlite3 test.db
 



