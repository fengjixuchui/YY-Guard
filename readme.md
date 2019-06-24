﻿
# YY-Guard——有效缓解DLL劫持攻击

## 1. 关于YY-Guard
我们知道，加载dll时是有搜索顺序的，具体顺序可以参考[MSDN —— dynamic-link-library-search-order](https://docs.microsoft.com/zh-cn/windows/desktop/Dlls/dynamic-link-library-search-order)。

这个顺序给我们提供便利的同时也引入了安全隐患，比如的程序根目录放上修改版的 version.dll、msimg32.dll或者其他。如果你的程序直接或者间接的依赖了这些dll，那么极有可能被劫持。

Dism++在设计之初就考虑到了这些安全问题，现在正式从Dism++中分离，并命名为——YY-Guard。我热切的希望在大家一起努力下，让程序的安全性更上一层楼！

[ [YY-Guard交流群 633710173](https://shang.qq.com/wpa/qunwpa?idkey=21d51d8ad1d77b99ea9544b399e080ec347ca6a1bc04267fb59cebf22644a42a) ]

### 1.1. 原理
* 支持 LOAD_LIBRARY_SEARCH_SYSTEM32 特性的系统中直接使用 LOAD_LIBRARY_SEARCH_SYSTEM32加载dll以及相关依赖项。
* 针对不支持 LOAD_LIBRARY_SEARCH_SYSTEM32 特性的系统，显式指定dll搜索顺序目录为System32。

### 1.2. 亮点
* 使用YY-Guard后，对于直接依赖的dll，我们只需要吧设置为延迟加载即可免疫相关劫持行为。
* 对于代码显式 LoadLibrary 的行为，我们也提供了 `YY::LoadLibraryFormSystem32` 导出函数。

### 1.3. 劫持攻击示例

很多人在编写代码时可能会编写出如下代码：
```C++
LoadLibraryW(L"wimgapi.dll");
```

这样其实存在一些安全隐患，因为应用程序目录优先级是最高的，如果攻击者在你的程序根目录放一个篡改后的`wimgapi.dll`，那么你就中枪了。
假设你的程序路径在`D:\Dism++`，那么你的程序就会去加载`D:\Dism++\wimgapi.dll`。

OK！或许你会说，我直接按全路径，就能避免这个问题，你那个什么鸟库就是个多余的，比如这样：
```C++
//演示之用，暂时用个死路径，请勿照搬照抄！
LoadLibraryW(L"C:\\Windows\\System32\\wimgapi.dll");
```

这样难道就真的没有问题了吗？实际结果会让你大吃一惊，众所周知wimgapi.dll依赖于version.dll，具体自行去System32，用Depend工具查看wimgapi.dll的导入表。

如果攻击者在你的程序根目录放一个加料版——version.dll，你的程序就会在加载wimgapi.dll时加载上加料版version.dll。

假设你的程序路径在`D:\Dism++`，那么你的程序就会去加载`C:\Windows\System32\wimgapi.dll`的同时顺道加载上`D:\Dism++\version.dll`。


不要害怕，YY-Guard能够解决此类问题，保障你的代码安全，你只需要这样做：
```C++

//加载System32中加载wimgapi.dll以及他的依赖项。
YY::LoadLibraryFormSystem32(L"wimgapi.dll");


//如果你需要加载你自己程序下的DLL，比如说 sites.dll
//从 C:\Program Files (x86)\XXXX 目录加载 sites.dll，但是依赖项依然从 System32目录加载。
YY::LoadLibraryFormSystem32(L"C:\\Program Files (x86)\\XXXX\\sites.dll");

```

## 2. 使用YY-Guard
1. 下载[YY-Guard-Binary](https://github.com/Chuyu-Team/YY-Guard/releases)，然后解压到你的工程目录。<br/>
2. 【链接器】-【输入】-【附加依赖项】，添加`objs\$(PlatformShortName)\YY_Guard.obj`。<br/>
3. 所有代码显式 LoadLibrary 的行为尽可能的替换为 YY::LoadLibraryFormSystem32（需要 #include "YY-Guard.h"）。
4. 重新编译代码。


## 3. YY-Guard兼容性
### 3.1. 支持的编译器
全平台ABI兼容。
* 所有Visual Studio版本均支持（比如：VC6.0、VS2008、VS2010、VS2015、VS2017、VS2019等等）。
* 所有运行库模式均支持（比如：`/MD`、`/MT`、`/MDd`、`/MTd`）。


## 更改日志

### 1.0.0.1 - 第一版（2019-06-24 18:00）
* 第一正式版。