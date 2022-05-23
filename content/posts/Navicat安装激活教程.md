# navicat安装激活教程

## 1.安装

打开官网:

https://navicat.com.cn/download/navicat-premium

然后如图选择你需要的版本下载即可:

![image-20220313212756871](https://billy.taoxiaoxin.club//img/202203132128787.png)

安装就是一路下一步.

![image-20220313213804497](https://billy.taoxiaoxin.club//img/202203132138428.png)

选择**同意**,下一步.

![image-20220313213954045](https://billy.taoxiaoxin.club//img/202203132139099.png)

选择**安装目录**,点击下一步.

![image-20220313214038925](https://billy.taoxiaoxin.club//img/202203132140091.png)

![image-20220313214137703](https://billy.taoxiaoxin.club//img/202203132141759.png)

等待安装完成.

![image-20220313214204498](https://billy.taoxiaoxin.club//img/202203132142586.png)

好了,到此安装结束

## 2.激活



**支持Windows和Mac!**

**windows:**

```powershell
@echo off

echo Delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[version and language]
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium" /s | findstr /L Registration"') do (
    reg delete %%i /va /f
)
echo.

echo Delete Info folder under HKEY_CURRENT_USER\Software\Classes\CLSID
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\Classes\CLSID" /s | findstr /E Info"') do (
    reg delete %%i /va /f
)
echo.

echo Finish

pause
```

**mac**:

```shell
@echo off

echo Delete HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium\Registration[version and language]
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\PremiumSoft\NavicatPremium" /s | findstr /L Registration"') do (
    reg delete %%i /va /f
)
echo.

echo Delete Info folder under HKEY_CURRENT_USER\Software\Classes\CLSID
for /f %%i in ('"REG QUERY "HKEY_CURRENT_USER\Software\Classes\CLSID" /s | findstr /E Info"') do (
    reg delete %%i /va /f
)
echo.

echo Finish

pause
```



## 3.其他路子

教师和学生申请！

https://www.navicat.com.cn/sponsorship/education/student