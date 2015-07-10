---
layout: post
title: "主流跨平台媒体库 FFmpeg Live555 MobileVLCKit 简介"
description: ""
category:
tags: [FFmpeg,iOS,VLC,Live555,MobileVLCKit]
---


### 下面是主流的媒体解码或播放库清单，它们都是跨平台支持的：

库名 | 简介 | 需要的能力 | 官网链接
---- | ---- | ---- | ----
FFmpeg | [FFmpeg是一套可以用来记录、转换数字音频、视频，并能将其转化为流的开源计算机程序。采用LGPL或GPL许可证。它提供了录制、转换以及流化音视频的完整解决方案。它包含了非常先进的音频/视频编解码库libavcodec，为了保证高可移植性和编解码质量，libavcodec里很多codec都是从头开发的。][ffmpeg-info] | 了解视频编解码原理和流程、了解图像压缩技术、音频压缩技术等 | [http://ffmpeg.org][ffmpeg]
Live555 | [Live555实现了对多种音视频编码格式的音视频数据的流化、接收和处理等支持，包括MPEG、H.263+、DV、JPEG视频和多种音频编码。同时由于良好的设计，Live555非常容易扩展对其他格式的支持。][live555-info] | 了解视频编解码原理和流程、了解图像压缩技术、音频压缩技术等 | [http://www.live555.com][live555]
MobileVLCKit | 鼎鼎大名的播放器`VLC`，优秀的封装，源码中最核心的部分，被封装成了独立的库，基于`ffmpeg``live555`提供完整的媒体播放库，你只需要定制自己的界面，支持[CocoaPods][cocoapods]导入库，开发一个简单界面的播放器，你只需要几行代码，几乎覆盖所有媒体格式！| 你只需要定制好自己的界面，它的API看起来就是一个播放器 | [http://www.videolan.org][vlc]


### 项目选用
如果是播放标准协议的视频流（http,rtsp,ftp等），建议使用`VLC`，这个库有600M左右，兼容`armv7 armv7s arm64`的情况下，编译后大约会增加`15～20M`左右的体积。

使用pod引入的话，非常简单，省去了编译烦恼。

```
pod 'MobileVLCKit'
```

如果你要做精细的视频解码控制，或者要优化程序的体积，或者说你喜欢折腾！那么可以选用`ffmpeg`或者`live555`。
要做视频编码的话，使用`ffmpeg`会得到很好的支持！

### 总结
上面提到的库都是非常优秀的媒体库！各有优势，选用的时候看需求吧。

[ffmpeg]: http://ffmpeg.org
[live555]: http://www.live555.com
[vlc]: http://www.videolan.org/vlc/download-ios.html
[live555-info]: http://baike.baidu.com/link?url=moZzIWKn-UCO3DkQLDKC8N2iVzMMy8I6Zp1nWPMk0oYLYunXgxdZXsM4QMkcMUgXNsuPekumV8uPImicBm1KzK "live555_百度百科"
[ffmpeg-info]: http://baike.baidu.com/link?url=54pxUOZ324UlLmL3Ucul8F9LSw5np6adCj6JJWtERWfQd6Atps0Ybg_XLTwnTq2yzd5OSKdiKNyeD6yZ5jVCxK "ffmpeg_百度百科"
[cocoapods]: http://cocoapods.org/


{% include JB/setup %}
