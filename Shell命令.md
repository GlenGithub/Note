ִ����rom�汾Ϊuser�汾����ϵ�����ṩroot���boot.img�滻������ˢ��

�򿪷���ģʽ(��ҪROOTȨ��)
settings put global airplane_mode_on 1
am broadcast -a android.intent.action.AIRPLANE_MODE --ez state true

�رշ���ģʽ(��ҪROOTȨ��)
settings put global airplane_mode_on 0
am broadcast -a android.intent.action.AIRPLANE_MODE --ez state false

MTUֵ������鿴(��ҪROOTȨ��)
adb shell
ip link set dev ccmni0 mtu 1300 (ip link set dev X mtu N �ûس� X=�������� N=mtuֵ)
�鿴����������Ϣ
ifconfig ccmni0

mtuֵ����
ping -l ���ݰ���С -f 192.168.1.1

�鿴ĳ��Ӧ�õİ汾��Ϣ
adb shell dumpsys package ����

apkȨ�޲鿴
aapt dmup badging xxx.apk

���CPU��Ϣ
adb shell dumpsys cpuinfo

�鿴ĳ�����ڴ�ʹ����Ϣ
adb shell dumpsys meminfo [processName]

�鿴ĳ����activityʹ�����
adb shell dumpsys activity [processName] | find '�����ַ�'

��ȡ������Ϣ
adb shell getevent

��ȡϵͳ����
adb shell getprop

����ϵͳ����(��ҪROOTȨ��)
adb shell setprop 

�鿴PM������Ϣ
adb shell pm

�鿴�ֻ��ڰ�װ���б�
adb shell pm list packages

����monkey������Ϣ
adb shell ps | find "monkey"

ɱ����
adb shell kill [pid]

����
adb shell reboot

��SVC��������
adb shell svc

��ѯwifi��������
adb shell svc wifi
�ر�wifi
adb shell svc wifi disable
��wifi
adb shell svc wifi enable

�������ݻָ�����
adb shell wipe data

����ָ��activity
adb shell am start -n <package_name>/.<activity_class_name>
�����װ��apk·��
adb shell pm path <package>
ɾ�������ص��������ݣ�������ݺͻ���
adb shell pm clear <package>
����SERVICE -n��ʾ��� -a��ʾservice��action
adb shell am startservice -n <package_name>/.<service_class_name>
adb shell am startservice -a 'android.intent.action.CALL'
���͹㲥 --es String��������  --ei int�������� --ez boolean����
adb shell am broadcast -a <broadcast_action> --es test_string 'this is a testing'


�鿴am������Ϣ
adb shell am

��Ļ¼�� ctrl+cֹͣ¼�� --size 800*600 ������Ƶ�ֱ���
adb shell screenrecord /mnt/sdcard/1.mp4
��SD���е�¼���ļ�����F��
adb pull /mnt/sdcard/1.mp4 f:\

��ͼ
adb shell screencap /mnt/sdcard/1.png

�鿴�ֱ���
adb shell wm size

��ӡlog
adb shell logcat -s TAG
���˵ȼ� 
logcat *:V | *:E
���log���浽�ļ�
adb logcat -v time > log.txt

settings�����÷�
adb shell
settings get secure default_input_method ��ȡĬ�����뷨
settings put secure default_input_method com.sohu.inputmethod.sogouoem/.SogoIME �����ѹ����뷨Ĭ��
settings get system screen_off_timeout ��ȡ��Ļ����ʱ��

ģ�ⰴ��HOME��
adb shell input keyevent 26
ģ�ⰴ�·��ؼ�
adb shell input keyevent 4

����DDMS
ddms

��ʾ��ǰ���е�ȫ��ģ����
adb devices

��ĳһģ����ִ������
adb -s ģ������� ����

��װӦ��
adb install -r apk

ж��Ӧ��
adb uninstall ����

��ȡģ�����е��ļ�
adb pull <remote> <local>

��ģ������д���ļ�
adb push <local> <remote>

�鿴��̨������Ϣ
adb shell service list

�鿴wifi mac��ַ
adb shell cat /sys/class/net/wlan0/address

�ֻ�δroot��Ӧ��debugģʽ��android:debuggable="false"��
adb shell run-as <package> ֱ�ӽ���/data/data/pacakge/Ŀ¼��
cat data.db > /mnt/sdcard/test.db  cat��������ݿ⿽����sd����

����DB
adb shell
sqlite3 test.db

��ȡ�����û�
adb shell dumpsys user
or
adb shell pm list users

�����û�
adb shell pm create-user userName

�л��û�
adb shell am switch-user userId

adb install --user 10 -r '/apk/path/debug.apk'
adb shell am start --user 10 -n com.test/.MainActivity
 
 
 �鿴so��Ϣ
 windows��cmd����E:\software\android-sdk-windows\ndk-bundle\toolchains\arm-linux-androideabi-4.9\prebuilt\windows-x86_64\binִ����������
 arm-linux-androideabi-objdump.exe -d "C:\Users\Administrator\Desktop\masterStaging\release\libzbarjni.so" | findstr "init"



