---
title: Chrome插件开发Demo
date: 2019-03-18 22:13:36
tags:
---
<!-- ![](https://images2015.cnblogs.com/blog/352797/201707/352797-20170711100448275-555008903.png) -->
对于一个前端developer来说，chrome插件的开发非常简单，几个简单的文件就能轻松搞定。
## 插件的目录结构
   ~~~
   demo
    - js 页面所引用的javascript文件。
      - popup.js
      - qrcode.min.js
    - icon.png 显示图标。
    - manifest.json 插件的配置文件，可看作插件的“入口”。
    - popup.html 弹出页面。
   ~~~
---
## 创建一个目录(demo)，并在目录中创建配置文件**manifest.json**
   ~~~ json
    {
        "manifest_version": 2,
        "name": "qrcode generator",
        "description": "二维码生成器",
        "version": "1.0",
        "browser_action": {
            "default_icon": "icon.png",
            "default_popup": "popup.html",
            "default_title": "点这里生成当前页二维码！"
        },
        "content_security_policy": "script-src 'self'; object-src 'self'",
        "permissions": ["activeTab","tabs"]
    }
   ~~~
## 创建弹出页面**popup.html**
   ~~~ html
   <!DOCTYPE html>
    <html lang="zh-CN">
        <head>
            <title>qrcode generator</title>
            <script type="text/javascript" src="js/qrcode.min.js"></script>
            <script type="text/javascript" src="js/popup.js"></script>
        </head>
        <body>
            <div style="width:150px; height:150px;" id="qrcode"> </div>
        </body>
    </html>
   ~~~
## 创建执行脚本**popup.js**
   ~~~ js
   function load() {
        window.console.info('ready.....');

        // 获取到当前激活态的 tab 对象
        chrome.tabs.query({
            "active" : true
        }, function(tabArr) {
            var tab = tabArr[0];
            window.console.info("title=" + tab.title);
            window.console.info("url=" + tab.url);

            var qrcode = new QRCode(document.getElementById("qrcode"), {
                width : 150,
                height : 150,
                colorDark : "#000000",
                colorLight : "#ffffff",
                correctLevel : QRCode.CorrectLevel.H
            });

            qrcode.clear();
            qrcode.makeCode(tab.url);
        });
    };
    window.onload = load();
   ~~~
   **qrcode.min.js**文件下载自[davidshimjs][].
## 验证插件是否可用
   打开chrome扩展程序***chrome://extensions/***，选择***开发者模式***，选择***已解压的扩展程序***，选择demo目录，然后就看到插件**二维码生成器**已经成功加载了
## 打包
   打开chrome扩展程序***chrome://extensions/***，选择***开发者模式***，选择***打包扩展程序***，如果打包失败，请根据错误提示信息修正，如果打包成功，会在与根目录同级的目录中生成CRX插件文件和PEM秘钥文件。
## 安装扩展程序
   打开chrome扩展程序***chrome://extensions/***，拖拽CRX文件进入到当前页面即可完成安装。

## [参考链接](http://www.runoob.com/w3cnote/javascript-qrcodejs-library.html)


[davidshimjs]:http://davidshimjs.github.io/qrcodejs/ 'Cross-browser QRCode generator for javascript'