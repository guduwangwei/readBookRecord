#### adb常用命令 

* 1  查看当前Activity
  *  shell中输入：**logcat | grep ActivityManager**   
  * cmd命令中输入：**adb shell dumpsys activity activities **

adb shell pm list packages -f 查看已安装app的包名

adb push foo.txt /sdcard/foo.txt 向手机sdcard 推送文件  

