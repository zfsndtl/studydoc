# izpack打包

## 1. izpack打包

1、编译安装izpack。

 ①进入文件夹uxdb-ng/install/linux/izpack，使用命令mvn clean install编译izpack。

 ②编译完成后，进入uxdb-ng/install/linux/izpack/izpack-dist/target文件夹，运行命令：java -jar izpack-dist***.jar安装izpack。

2、环境确认：   打包前需确定本机已安装python，并且配置了python环境变量，使用izpack工具打包时需要调用python脚本。

D:\Program Files\Python\Python38\Scripts

D:\Program Files\Python\Python38\

![image-20201028104731807](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201028104731807.png)

D:\Program Files\Python\Python38



3、自动打包：   运行工程目录下，自动打包脚本izpack_pkg_cmd.bat，命令格式： izpack_pkg_cmd.bat version version_type   示例：

注意izpack_pkg_cmd.bat脚本几个目录格式

set src_dir=C:\uxdb-release
set izpackdir=D:\Program Files\IzPack
set python_pkg_dir=%izpackdir%\utils\wrappers\izpack2exe
set pkg_src_dir=%bat_path%packSource
set uxfs_config_targetpath=D:\UXDB
set uxfs_config_sourcepath=%src_dir%\uxfs\etc\xos\uxfs_conf



python打包

```bat
python "D:\Program Files\IzPack"\utils\wrappers\izpack2exe\izpack2exe.py --file=uxdbee.jar --output=uxdb-win-x86_64-ver%version%-EE.exe
```

工程目录：E:\uxfs-2.1.1.3-izpack\uxdb-ng\install\windows

exe资源源目录：C:\uxdb-release

![image-20201028114505604](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201028114505604.png)

```bat
@echo off
setlocal enabledelayedexpansion

set fn=%1

REM (for /f "delims=" %%i in (%fn%) do (
REM set s=%%i
REM set s=!s:/home/uxdb/uxdbinstall=%2!
REM set s=!s:/etc/xos/xtreemfs/policies=%2\uxfs\etc\policies!
REM set s=!s:/=\!
REM echo !s!))>%1_temp.properties
REM move /y %1_temp.properties "%fn%"

```

```shell
@echo off
setlocal enabledelayedexpansion

set fn=%1

rem source file directory
set dir=%1
rem file suffix
set suffix=*.properties




cd %dir%

REM for %%f in ('dir /b %suffix%') do 
REM (   
REM set s=%%f
REM echo %s%
REM )

REM for /f "delims=" %%b in ('dir /b %suffix%') do 
REM (   
REM echo %%~dpnxb
REM )


rem founction 1 pi liang

for /r %%i in (%suffix%) do (
echo %%~dpnxi
echo %%~nxi
rem 此处文件%%i没有办法被赋值
rem set x=%%i



for /f "delims=" %%j in (%%~dpnxi) do (
set s=%%j
set s=!s:/home/uxdb/uxdbinstall=%2!
set s=!s:/etc/xos/xtreemfs/policies=%2\uxfs\etc\policies!
set s=!s:/home/uxdb/ngdb/objs=%2\uxfs\ngdb\objs!
set s=!s:/=\!
echo !s!))>%%i_temp.properties
rem 此处文件没有办法被move替换,找不到源文件
rem move /y %%i_temp.properties "%%~dpnxi"
)



rem founction 2
REM echo %dir%
REM dir /b  %suffix%

rem founction 3  one by one
REM (for /f "delims=" %%i in (%fn%) do (
REM set s=%%i
REM set s=!s:/home/uxdb/uxdbinstall=%2!
REM set s=!s:/etc/xos/xtreemfs/policies=%2\uxfs\etc\policies!
REM set s=!s:/home/uxdb/ngdb/objs=%2\uxfs\ngdb\objs!
REM set s=!s:/=\!
REM echo !s!))>%fn%_temp.txt
REM timeout /t 5
REM move /y %fn%_temp.txt "%fn%"
pause

```



izpack_pkg_cmd.bat 加入如下内容：
注意bat文件调用外部bat文件时，使用call命令，否则外部命令执行完会退出

![image-20201028123430945](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201028123430945.png)

```bat
call replace.bat %uxfs_config_sourcepath%\dirconfig.properties  %uxfs_config_targetpath% 
call replace.bat %uxfs_config_sourcepath%\osdconfig.properties  %uxfs_config_targetpath% 
call replace.bat %uxfs_config_sourcepath%\mrcconfig.properties  %uxfs_config_targetpath%
echo replace uxfs_config is ok!
    
```



## 2.其他

查看当前目录.md文件

dir .\ /b /s .\\|findstr /i .md

![image-20201029095336857](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029095336857.png)

dir .\ /b  |findstr /i .bat

![image-20201029095934860](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029095934860.png)

dir  /b *.bat

![image-20201029102257515](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029102257515.png)



bat中%和%%有什么区别，怎么用？

![image-20201029114419745](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114419745.png)

![image-20201029114432022](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114432022.png)

![image-20201029114443174](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114443174.png)

![image-20201029114605763](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114605763.png)

![image-20201029114644620](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114644620.png)

![image-20201029114712877](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114712877.png)

![image-20201029114732128](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114732128.png)

![image-20201029114745729](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114745729.png)

![image-20201029114807872](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114807872.png)

![image-20201029114821009](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114821009.png)

![image-20201029114939646](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029114939646.png)

![image-20201029115018291](C:\Users\Lenovo\AppData\Roaming\Typora\typora-user-images\image-20201029115018291.png)