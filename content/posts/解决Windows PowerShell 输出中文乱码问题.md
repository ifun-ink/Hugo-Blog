---
title:  WindowsPowerShell中文乱码问题解决
subtitle:
date: 2024-04-06T21:41:18+08:00
slug: Ki3SYG5b
tags:
  - PowerShell
  - Windows
categories:
  - 系统管理
---

乱码原因：当文本的字符编码与终端或文本编辑器使用的字符编码不匹配时，就会出现乱码。比如，文本是以 UTF-8 编码保存的，但终端或编辑器使用了其他编码（如 GB2312、GBK 等）来解析显示文本，就会导致中文乱码。

#### 查看默认配置

打开终端分别输入以下两条命令;

```
[console]::OutputEncoding

[console]::InputEncoding
```

得到结果如下

```
PS D:\Project> [console]::OutputEncoding
BodyName          : gb2312
EncodingName      : 简体中文(GB2312)
HeaderName        : gb2312
WebName           : gb2312
WindowsCodePage   : 936
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.InternalEncoderBestFitFallback
DecoderFallback   : System.Text.InternalDecoderBestFitFallback
IsReadOnly        : False
CodePage          : 936


PS D:\Project> [console]::InputEncoding


BodyName          : gb2312
EncodingName      : 简体中文(GB2312)
HeaderName        : gb2312
WebName           : gb2312
WindowsCodePage   : 936
IsBrowserDisplay  : True
IsBrowserSave     : True
IsMailNewsDisplay : True
IsMailNewsSave    : True
IsSingleByte      : False
EncoderFallback   : System.Text.InternalEncoderBestFitFallback
DecoderFallback   : System.Text.InternalDecoderBestFitFallback
IsReadOnly        : True
CodePage          : 936
```

可以看到默认编码是gb2312

我们需要把默认编码改成utf-8

#### 修改默认配置

输入命令此命令，使用记事本打开配置文件：

```
notepad $PROFILE
```

写入配置代码如下：

```
$OutputEncoding = [console]::InputEncoding = [console]::OutputEncoding = [Text.UTF8Encoding]::UTF8
```

或者以下代码： 也可以

```
[console]::OutputEncoding = [Text.Encoding]::UTF8
[console]::InputEncoding = [Text.Encoding]::UTF8
```

保存即可。

关闭终端重新打开，验证效果，问题应当得到解决。
