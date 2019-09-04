---
title: 钉钉扫一扫登录
vssue-title: 钉钉扫一扫登录
date: 2019-09-04
tags:
  - 前端
  - 第三方交互
categories: [前端]
---

[[toc]]

记录一下如何在 WEB 端实现钉钉扫一扫登录，并且与 vue 框架结合

文档参考

[钉钉开发文档]: https://ding-doc.dingtalk.com/doc#/serverapi2/kymkv6

## 引入依赖

```xml
<script src="https://g.alicdn.com/dingding/dinglogin/0.0.5/ddLogin.js"></script>
```

## 实例化钉钉 sdk

```js
/*
 * 解释一下goto参数，参考以下例子：
 * var url = encodeURIComponent('http://localhost.me/index.php?test=1&aa=2');
 * var goto = encodeURIComponent('https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=appid&response_type=code&scope=snsapi_login&state=STATE&redirect_uri='+url)
 */
var obj = DDLogin({
  id: 'login_container', //这里需要你在自己的页面定义一个HTML标签并设置id，例如<div id="login_container"></div>或<span id="login_container"></span>
  goto: '', //请参考注释里的方式
  style: 'border:none;background-color:#FFFFFF;',
  width: '365',
  height: '365'
})
```

```Tips
!!!
goto参数需要对url decode, 再对整个goto参数decode
```

## 注册扫码回调函数

```js
// 定义扫码回调函数
const handleMessage = async event => {
  const origin = event.origin
  if (origin == 'https://login.dingtalk.com') {
    //判断是否来自ddLogin扫码事件。
    const loginTmpCode = event.data //拿到loginTmpCode后就可以在这里构造跳转链接进行跳转了
    // 存储state，用于校验
    const state = new Date().getTime() * Math.random()
    window.sessionStorage.setItem('dingding_state', state)
    // 跳转到dingding oauth2地址来获取code
    window.location.href = `https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=APPID&response_type=code&scope=snsapi_login&state=${state}&redirect_uri=REDIRECT_URI&loginTmpCode=${loginTmpCode}`
  }
}
// 注册扫码回调函数
if (typeof window.addEventListener != 'undefined') {
  window.addEventListener('message', handleMessage, false)
} else if (typeof window.attachEvent != 'undefined') {
  window.attachEvent('onmessage', handleMessage)
}
```

## 判断跳转得到的 code 和 state

```js
// 经过扫码跳转回来，带code时直接走登录
if (this.$route.query && this.$route.query.code) {
  if (
    window.sessionStorage.getItem('dingding_state') === this.$route.query.state
  ) {
    this.login(this.$route.query.code)
    return
  } else {
    this.$message('钉钉状态验证失败')
  }
}
```

## 完整代码

在 login 页面的 monted()钩子里写入即可

```js
async mounted() {
    // dingding扫码登录
    // doc: https://ding-doc.dingtalk.com/doc#/serverapi2/kymkv6
    // 经过扫码跳转回来，带code时直接走登录
    if (this.$route.query && this.$route.query.code) {
      if (
        window.sessionStorage.getItem('dingding_state') ===
        this.$route.query.state
      ) {
        this.login(this.$route.query.code)
        return
      } else {
        this.$message('钉钉状态验证失败')
      }
    }
    // 没code初始化钉钉二维码
    let { appid, REDIRECT_URI } = config
    // 本地调试，写死地址
    // REDIRECT_URI = 'http://ttjadmin.tuotuoltd.com/#/login'
    const state = new Date().getTime() * Math.random()
    window.DDLogin({
      id: 'qrcode', //这里需要你在自己的页面定义一个HTML标签并设置id，例如<div id="login_container"></div>
      goto: window.encodeURIComponent(
        `https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=${appid}&response_type=code&scope=snsapi_login&state=${state}&redirect_uri=${window.encodeURIComponent(
          REDIRECT_URI
        )}`
      ),
      style: 'border:none;background-color:rgb(255, 255, 255,0);',
      width: '365',
      height: '365'
    })
    // 定义扫码回调函数
    const handleMessage = async event => {
      const origin = event.origin
      if (origin == 'https://login.dingtalk.com') {
        //判断是否来自ddLogin扫码事件。
        const loginTmpCode = event.data //拿到loginTmpCode后就可以在这里构造跳转链接进行跳转了
        // 存储state，用于校验
        window.sessionStorage.setItem('dingding_state', state)
        // 跳转到dingding oauth2地址来获取code
        window.location.href = `https://oapi.dingtalk.com/connect/oauth2/sns_authorize?appid=APPID&response_type=code&scope=snsapi_login&state=${state}&redirect_uri=REDIRECT_URI&loginTmpCode=${loginTmpCode}`
      }
    }
    // 注册扫码回调函数
    if (typeof window.addEventListener != 'undefined') {
      window.addEventListener('message', handleMessage, false)
    } else if (typeof window.attachEvent != 'undefined') {
      window.attachEvent('onmessage', handleMessage)
    }
  },
```
