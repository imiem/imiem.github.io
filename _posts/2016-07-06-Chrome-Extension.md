---
layout: post
title:  "小觑Chrome Extension"
date:   2016-07-06 14:06:05
categories: 前端
tags: Chrome Extension
---

* content
{:toc}

最近一朋友问了一个关于Chrome Extension的问题，因为之前从来没有接触过，然后就自己看着文档动手写了一个简单的示例。Chrome Extension除了官方的一个API接口外，其他的就是一些html，css，js的知识了，如果很熟悉这些，写一个简单的自己的插件还是可以的。这里只是做了一个最简单的功能，毕竟我的前端功底不是太好，如果想有更加深入的了解，强烈建议读下[Chrome的官方文档](https://developer.chrome.com/extensions/getstarted)。



## 为何不看看官方文档？
　　[Chrome的官方文档](https://developer.chrome.com/extensions/getstarted)做的相当的赞。如果有梯子并且能看懂一点英文的话，下边的就没必要看了。

## 我要做什么？
　　实现的[代码可以点击查看](https://github.com/MrDebuger/CommonCode/tree/master/js/ChromeExtensionDemo)。如果没有一个自己想要的最终效果，那么做东西就会相当的没有动力。我要做的很简单，就是输入一个图片的url，然后点击加载，会在弹出的页面上加载出一张图片。最终实现的效果如下图所示：
[<img src="{{ site.baseurl }}images/Chrome-Extension-1.png" alt="" style="width: 100%px;"/>]({{ site.baseurl/images/Chrome-Extension-1.png}}/)

## 文件组成

### 基本文件组成

<table boder="1px" cellspacing="0px" style="width:100%">
	<tr>
		<td>文件名</td>
		<td>作用</td>
	</tr>
  <tr>
		<td>manifest.json</td>
		<td>一些配置选项</td>
	</tr>
  <tr>
		<td>icon.png</td>
		<td>插件的图标，官方给出的是19px * 19px</td>
	</tr>
  <tr>
		<td>popup.html</td>
		<td>弹出的主页面</td>
	</tr>
  <tr>
		<td>popup.js</td>
		<td>外部js</td>
	</tr>
  </table>


### manifest.json
  + manifest_version:指定文件格式的版本，目前的默认为2就可以了。
  - 下边的其他的参数配置应该就很容易明白了，当然还有一些其他的配置参数，可以根据自己的需要自行配置。

```json
{
  "manifest_version":2,

  "name":"ChromeExtensionDemo",
  "description":"My First Chrome Extension",
  "version":"1.0",

  "browser_action":{
    "default_icon":"icon.png",
    "default_popup":"popup.html"
  }
}
```

### popup.html
+ 主页面，即点击弹出按钮后弹出的页面，这是一个标准的html页面，**其中不能包含内联事件，所有的js代码最好写在外边**。

```html
<!doctype html>
<html>
  <head>
    <title>My First Chrome Extension</title>
    <style>
      body {
        font-family: "Consolas","Segoe UI", "Lucida Grande", Tahoma, sans-serif;
        font-size: 100%;
        width: 300px;
        height: 400px;
      }
      #status {
        white-space: pre;
        text-overflow: ellipsis;
        overflow: hidden;
        max-width: 600px;
      }
    </style>

    <script src="popup.js"></script>
  </head>
  <body>
    <div style="margin-top:20px">
      <label style="">Please insert a image url:</label>
      <input type="text" id="imgUrl" name="imgUrl" style="width:100%"/>
    </div>

    <div>
      <button>Load Image!</button>
      <img id="myImage" src="?" width="300" height="200" />
    </div>
  </body>
</html>

```

### popup.js

```javascript
function clickHandler(element) {
  loadImg();
}
function loadImg(){
  var imgUrl = document.getElementById("imgUrl").value;
  document.getElementById("myImage").src = imgUrl;
}
document.addEventListener('DOMContentLoaded', function () {
          document.querySelector('button').addEventListener('click', clickHandler);
        });

```

### icon.png
顾名思义，就是该插件的图标名。

## 如何查看效果？

[<img src="{{ site.baseurl }}images/Chrome-Extension-2.png" alt="" style="width: 100%px;"/>]({{ site.baseurl/images/Chrome-Extension-2.png}}/)

## 问题总结
* 由于Chrome Extension**不执行Inline JavaScript**，这说明同时禁止内嵌的```<script>块和内联事件（例如： <button onclick="...">）```,因此请不要在html中编写js代码。
* 代码一次写成的概率很小，因此如果出现错误可以尝试着进行[debug](https://developer.chrome.com/extensions/tut_debugging)一下。
* 出现问题后没必要直接就搜索，有时仔细的看一下错误信息也许会有收获。
