---
title: 纯H5套壳APP实现扫一扫
vssue-title: 纯H5套壳APP实现扫一扫
date: 2019-09-02
tags:
  - 大前端
  - hybrid-app
categories: [大前端]
---


由于公司需求需要快速上线一版 app，试验一下用纯 h5 套在 app 壳里，用 jsbridge 的方式和原生 app 层进行交互的可行性

<!-- more -->

[[toc]]

## 产品需求

- 新增 webview 套入 H5
- 引入 jsbridge，实现与原生 app 层交互
- 引入原生层扫一扫实现库，与 js 实现交互


## 放入 webview

在 res/layout/activity_main.xml 中加入 webview 组件

```xml
<WebView
  android:id="@+id/tuotuo_webview"
  android:layout_width="match_parent"
  android:layout_height="match_parent">
</WebView>
```

在 MainActivity 中，初始化 webview，并进行一系列配置

```java
webView = findViewById(R.id.my_webview);
WebSettings webSettings = webView.getSettings();
//处理http和https混源
webView.getSettings().setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
// 开启webview Storage能力
webSettings.setDomStorageEnabled(true);
// 支持zoom
webSettings.setSupportZoom(true);
// !关键 开启js
webSettings.setJavaScriptEnabled(true);
webSettings.setBlockNetworkImage(false);
webSettings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.SINGLE_COLUMN);
webSettings.setUseWideViewPort(true);
//加载一个网页：
webView.loadUrl("https://msaas.tuotuoltd.com");
```

## 引入 dsbridge

要赋予 h5 与原生层交流的能力，app 需要引入 dsbridge 库，让 js 和原生层架起一道桥梁。

[dsbridge 安卓github地址]: https://github.com/wendux/DSBridge-Android

1. 安装

```groovy
   allprojects {
     repositories {
      ...
      maven { url 'https://jitpack.io' }
     }
   }
```

2. 添加依赖

   ```groovy
   dependencies {
   	//compile 'com.github.wendux:DSBridge-Android:3.0-SNAPSHOT'
   	//support the x5 browser core of tencent
   	//compile 'com.github.wendux:DSBridge-Android:x5-3.0-SNAPSHOT'
   }
   ```

3. 添加 API 类实例到 DWebView

```xml
<wendu.dsbridge.DWebView
 android:id="@+id/tuotuo_webview"
 android:layout_width="match_parent"
 android:layout_height="match_parent">
</wendu.dsbridge.DWebView>

```

```java

import wendu.dsbridge.DWebView
...
DWebView dwebView= (DWebView) findViewById(R.id.my_webview);
dwebView.addJavascriptObject(new JsApi(), null);

```

3. 在 Javascript 中调用原生 (Java/Object-c/swift) API ,并注册一个 javascript API 供原生调用.

```javascript
//npm方式安装初始化代码
//npm install dsbridge@3.1.3
var dsBridge = require('dsbridge')
```

4. js 中调用原生

```javascript
if (dsbridge.hasNativeMethod('startScan')) {
  dsbridge.call('startScan', 'js调用扫一扫', async data => {
    // 获取结果
  })
}
```

5. java 定义 js 所调用的方法

   ```java
   public class JsApi {
       @JavascriptInterface
       public void startScan(Object msg, CompletionHandler<String> handler) {
           // handler 返回结果
       }
   }
   ```

## 实现扫一扫

引入扫一扫库

[bgaqrcode-android]: https://github.com/bingoogolapple/BGAQRCode-Android

完善 js 所调用原生的扫一扫方法，调用 MainActivity 的方法，跳转去扫一扫页面

```java
public class JsApi {
    MainActivity activity;

    public JsApi(MainActivity context) {
        this.activity = context;
    }

    @JavascriptInterface
    public void startScan(Object msg, CompletionHandler<String> handler) {
        activity.startScan(handler);
    }
}
```

```java
public void startScan(CompletionHandler<String> handler) {
   this.handler = handler;
   startActivityForResult(new Intent(MainActivity.this, TestScanActivity.class), 1);
}
```

等待扫一扫页面返回的结果

```java
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    String result = data.getStringExtra("result");
    handler.complete(result);
}
```
