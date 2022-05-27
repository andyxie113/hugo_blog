---
title: "Navicat激活"                         
author: "雨吁"  
description : "Navicat激活"    
date: 2022-05-23        
lastmod: 2022-05-23             

tags : [                                    
"navicat",
"工具",
"激活工具"
]
categories : [                              
"navicat",
"工具",
"激活工具"
]
keywords : [                                
"navicat",
"工具",
"激活工具"
]
---


## 1.安装

​	打开官网:

https://navicat.com.cn/download/navicat-premium

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

​	教师和学生申请！

https://www.navicat.com.cn/sponsorship/education/student
