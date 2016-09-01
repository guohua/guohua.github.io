---
layout:     post
category:   "Visual C++"
title:      "禁止链接警告4197的方法"
subtitle:   ""
date:       2016-08-24
author:     "郭华"
tags:
    - Visual C++
---

# VC6
打开 StdAfx.h 文件，查找：
```cpp
#pragma once
```
替换为：
```cpp
#pragma once

// 禁止链接警告4197，适用于VC6(VC7及其以上版本将 /ignore:4197 加入工程设置的链接器附加选项中)
#if _MSC_VER < 1300
#pragma comment(linker, "/ignore:4197")
#endif
```
# VC7-VC9
修改工程设置，在链接器附加选项中加入 /ignore:4197，下面说明手工修改工程文件的步骤。
打开工程文件，查找：
```xml
Name="VCLinkerTool"
```
替换为：
```xml
Name="VCLinkerTool"
AdditionalOptions="/ignore:4197"
```
# VC10 以上
查找：
```xml
<TargetMachine>MachineX86</TargetMachine>
```
替换为：
```xml
<TargetMachine>MachineX86</TargetMachine>
<AdditionalOptions>/ignore:4197 %(AdditionalOptions)</AdditionalOptions>
```
查找：
```xml
<TargetMachine>MachineX64</TargetMachine>
```
替换为：
```xml
<TargetMachine>MachineX64</TargetMachine>
<AdditionalOptions>/ignore:4197 %(AdditionalOptions)</AdditionalOptions>
```