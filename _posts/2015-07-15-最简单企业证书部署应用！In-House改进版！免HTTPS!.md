---
layout: post
title: "最简单企业证书部署应用！In-House改进版！免HTTPS!"
description: ""
category:
tags: [iOS,Xcode,AppStore]
---


# 简介
`iOS`平台发布应用，想绕过`AppStore`，最好的方式就是`In-House`发布了（由于越狱用户覆盖不全，一般不考虑越狱方式）。
网上搜索`In-House`的教程也很多，怎么申请企业证书，怎么对`ipa`包进行大包签名我就不再复述，文章最后附了两个链接，不了解的童鞋可以看一看。

如果有做过`In-House`部署，应该知道，需要准备一个描述应用信息的`*.plist`文件上传到服务器，并且从`iOS7`及以后版本，此文件必须部署在`HTTPS`服务器上才能正常安装。这一步非常容易出错不能成功部署。

# 出错原因：
* 签名错误或者打包方式不对。
* 是因为对配置文件不了解，出错了也不知道错在哪里。
* 没有条件部署`HTTPS`服务器

# 最简单的方式（上传`ipa`包到`http`服务器，调用一个`js`方法）
* 安装页面引入这个`JS`

```javascript
<script src="http://iosinstall.sinaapp.com/ios/install.js"></script>
```
* 在安装按钮的位置调用`openInstallURL`方法，可以使用任意HTML标签！

```javascript
<div onclick="openInstallURL({
    'title'   : '我是App标题',
    'ipa'     : 'http://www.xxx.com/app.ipa',
    'version' : '1.0',
    'ident'   : 'cn.xxx.xxx',
    'icon'    : ''});">点击安装</div>
```

* 参数说明

参数 | 说明 | 备注
---- | ---- | ----
title | 标题 | Safari弹出安装提示时提示的标题
ipa | 你猜 | ipa包需要你上传到自己的服务器上，然后将可以下载到这个ipa包的URL填写到这里，可以使用`HTTP`协议！
version | 你再猜 | 可以随意填写，也可以安装实际填写
ident | App唯一标识符 | 你可以在项目配置的`Bundle Identifier`下看到他
icon | 安装加载过程中的图标 | 如果传入空字符串，会有一个默认图标：![](http://iosinstall.sinaapp.com/ios/install.png)


# Demo

* 对HTML不熟悉的同学可以直接用下面的代码，样式已经写好了，将其保存成`*.html`文件即可


```javascript
<html>
<head>
  <meta name="generator" content="HTML Tidy for Mac OS X (vers 31 October 2006 - Apple Inc. build 15.12), see www.w3.org" />
  <meta charset="utf-8" />
  <meta http-equiv="X-UA-Compatible" content="IE=edge" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>APP测试</title>
  <link rel="stylesheet" href="http://iosinstall.sinaapp.com/ios/install.css">
  <script src="http://iosinstall.sinaapp.com/ios/install.js"></script>
</head>
<body>
  <h5 style="text-align: center; color: red; padding: 20px;">⚠️注意！以下安装包仅用于测试！<br/></h5>
  <install-btn onclick="openInstallURL({
    'title' 	: '我是标题',
	  'ipa'     : 'http://www.xxx.com/ipa/xxxx.ipa',
	  'version'	: '1.0',
	  'ident'   : 'com.xxx.xxx',
	  'icon'    : ''});">点击我安装</install-btn>
</body>
</html>
```

如果你想完全自己提供这些，请看下面的内容。
-------------
# 实现原理
我曾经为了解决`In-House`部署问题，也走了很多弯路，为了解决`HTTPS`的问题，使用私有证书，利用`Dropbox`的HTTPS服务，又或是使用Github的HTTPS服务，这些方式都是可行的，但是都有不同程度的麻烦，于是有了今天这个帖子。

实现逻辑：客户端根据自己的软件需求，传参到服务器，服务器动态生成`*.plist`，因为`iOS`会检测`*.plist`的URL，不能带有参数，所以将参数用`Base64`加密后加到URL路径中，服务器截取路径中的参数部分解密获得参数。由于路径变化的特殊性，需要配置好服务器的重定向。
这样，就不需要每个新的应用都去配置一次`*.plist`文件了！
我现在提供的动态`*.plist`运行在新浪云稳定快速，可以放心使用！

* `js`实现的逻辑：收集参数，将参数加密成Base64字符串，插入到访问URL里面。


```javascript
// http://iosinstall.sinaapp.com/ios/install.js
var base64EncodeChars = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/";
var base64DecodeChars = new Array(
	-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1,
	-1, -1, -1, -1, -1, -1, -1, -1, -1, -1, -1, 62, -1, -1, -1, 63,
	52, 53, 54, 55, 56, 57, 58, 59, 60, 61, -1, -1, -1, -1, -1, -1,
	-1,  0,  1,  2,  3,  4,  5,  6,  7,  8,  9, 10, 11, 12, 13, 14,
	15, 16, 17, 18, 19, 20, 21, 22, 23, 24, 25, -1, -1, -1, -1, -1,
	-1, 26, 27, 28, 29, 30, 31, 32, 33, 34, 35, 36, 37, 38, 39, 40,
	41, 42, 43, 44, 45, 46, 47, 48, 49, 50, 51, -1, -1, -1, -1, -1);

// Base64编码的方法
function base64encode(str) {
	var out, i, len;
	var c1, c2, c3;
	len = str.length;
	i = 0;
	out = "";
	while(i < len) {
		c1 = str.charCodeAt(i++) & 0xff;
		if(i == len) {
			out += base64EncodeChars.charAt(c1 >> 2);
			out += base64EncodeChars.charAt((c1 & 0x3) << 4);
			out += "==";
			break;
		}
		c2 = str.charCodeAt(i++);
		if(i == len) {
            out += base64EncodeChars.charAt(c1 >> 2);
            out += base64EncodeChars.charAt(((c1 & 0x3)<< 4) | ((c2 & 0xF0) >> 4));
            out += base64EncodeChars.charAt((c2 & 0xF) << 2);
            out += "=";
            break;
        }
        c3 = str.charCodeAt(i++);
        out += base64EncodeChars.charAt(c1 >> 2);
        out += base64EncodeChars.charAt(((c1 & 0x3)<< 4) | ((c2 & 0xF0) >> 4));
        out += base64EncodeChars.charAt(((c2 & 0xF) << 2) | ((c3 & 0xC0) >>6));
		out += base64EncodeChars.charAt(c3 & 0x3F);
	}
	return out;
}

/**
  Demo:
  var info = {	'title'		: '我是标题', // app name
				'ipa'		: 'http://www.xxx.com/ipa/xxx.ipa', // ipa url
				'version'	: '1.0',
				'ident'		: 'cn.xxx.xxx',
				'icon'		: '' // icon url
  };
  openInstallURL(info);
 */
function openInstallURL(info) {
	if (info.ident == null || info.ident.length == 0) {
		info.ident = 'cn.ineva.cn';
	}
	if (info.icon == null || info.icon.length == 0) {
		info.icon = 'http://iosinstall.sinaapp.com/ios/install.png';
	}
	if (info.version == null || info.version.length == 0) {
		info.version = '1.0.0';
	}
	var url = info.title + '@' + info.ipa + '@' + info.version + '@' + info.ident + '@' + info.icon;
	url = base64encode(encodeURI(url));
	url = 'https://iosinstall.sinaapp.com/ios/' + url.substr(0, url.length-2) + '/install.plist';
	console.log(url);
	url = 'itms-services://?action=download-manifest&url=' + url;
	window.self.location = url;
}
```

* 服务器PHP实现：从URL中截获参数，使用参数拼接好`*.plist`文件内容，将拼接好的内容当文件返回。

```php
<?php


//-------------------------------------------
//	解决 ios8 初期版本安装问题
	$sel = array(
        '12A365',
    	'12A366',
        '12A402'
    );

	function auto_file_name() {

        return round(time()/(60*10), 0);

        $randNum = rand(100,9999);
        $fileName =  iconv('gb2312','utf-8', $randNum.time());
        return $fileName;
    }

	$auto = '';
	foreach ($sel as $s) {
        if	( strpos($_SERVER["HTTP_USER_AGENT"], $s) ) {
            $auto = '.'.auto_file_name();
            break;
        }
	}


    // 获取参数
	$url = 'http://'.$_SERVER['SERVER_NAME'].':'.$_SERVER["SERVER_PORT"].$_SERVER["REQUEST_URI"];
    $tmp = explode("/", $url);
	$tmp = $tmp[4];
	$tmp = $tmp.'==';

	$tmp = base64_decode($tmp);
    $arr = explode("@", $tmp );

	$title = urldecode( $arr[0] );
	$url = urldecode( $arr[1] );
	$version = urldecode( $arr[2] );
	$ident = urldecode( $arr[3].$auto );
	$icon = urldecode( $arr[4] );

	$data = '<?xml version="1.0" encoding="UTF-8"?>
    		<plist version="1.0">
            <dict>
            <key>items</key>
            <array>
            <dict>
            <key>assets</key>
            <array>
            <dict>
            <key>kind</key>
            <string>software-package</string>
            <key>url</key>
            <string>'.$url.'</string>
            </dict>
            <dict>
            <key>kind</key>
            <string>display-image</string>
            <key>needs-shine</key>
            <true/>
            <key>url</key>
            <string>'.$icon.'</string>
            </dict>
            </array>
            <key>metadata</key>
            <dict>
            <key>bundle-identifier</key>
            <string>'.$ident.'</string>
            <key>bundle-version</key>
            <string>'.$version.'</string>
            <key>kind</key>
            <string>software</string>
            <key>title</key>
            <string>'.$title.'</string>
            </dict>
            </dict>
            </array>
            </dict>
            </plist>';


	$file_size = strlen($data);
	$filename = 'appinstall.plist';

	$filename = iconv("UTF-8", "GB2312", $filename);
	ob_clean();
	header("Content-type:application/octet-stream");
	header("Server:nginx/1.4.4");
	header("Content-type:text/plain");
	header("Accept-Ranges:bytes");
	header("Accept-Length:".$file_size);
	header("Content-Disposition: attachment; filename=".$filename);

	echo $data;
```

# 相关推荐
非常详尽的`In-House`部署教程：[http://blog.csdn.net/yangxt/article/details/7998762](http://blog.csdn.net/yangxt/article/details/7998762)

利用`Github`的`HTTPS`服务部署：[http://my.oschina.net/qixiaobo025/blog/321050](http://my.oschina.net/qixiaobo025/blog/321050)
