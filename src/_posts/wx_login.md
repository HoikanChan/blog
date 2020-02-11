## PC端微信登录

使用第三方微信登录时，仿照知乎小红书等网站采用的的弹窗式扫码登录，可以避免主页面跳转

### 前端主要流程

1. 主页面构造微信扫一扫地址，其中包括

   扫码成功后跳转地址(应该写入我们的扫码成功页)

   随机数（扫码成功后微信会把随机数存储到url的query中，用于检验扫码流程的连续性）

2. 打开新窗口，新窗口地址为步骤一构造的微信地址

3. 主窗口在window下注册全局变量，供新窗口调用

4. 新窗口会出现微信二维码，用户扫一扫后跳转写入的扫码成功页

5. 扫码成功页拿到url的随机数，对比是否在主页面写入的

6. 扫码成功页拿url的code，请求服务器做微信登录，并调用主页面，将请求结果传回主页面

7. 主页面拿到返回结果。用户微信已绑定系统账户时，服务器会返回token，系统直接登录

8. 如果用户微信未绑定系统账户时，弹窗提示需要绑定账户，并且记录服务器返回的openid

### 前端主要代码

1. 点击主页面微信登录

   ```javascript
   @click="openWxAuthWindow"
   ```

   

2. 主页面，将扫码回调页地址注入到url参数中，并且弹窗扫码页面

```javascript
openWxAuthWindow() {
     // 回调页地址
      const REDIRECT_URI = window.encodeURIComponent(
        window.location.origin + '/#/wxLogin'
      )
      // 随机生成state供扫码成功后验证
      const state = parseInt(new Date().getTime() * Math.random())
      // 打开新窗口，地址为微信扫一扫地址，里有跳转地址和随机数
      window.open(
        getWxAuthUrl(appId, REDIRECT_URI, state),
        'newwindow',
        'height=500, width=500, top=50%, left=100'
      )
      //  新窗口回调函数
      //  res[Object]{ 
      //    needBinding[bool]  是否需要绑定系统账户
      //    openid[str]        微信openid
      //    unionid[str]       微信unionid
 	  //    cook[str]          系统token (需绑定账户时返回null)
      //   }
      window.wxLoginComplete = res => {
        if (res.needBinding) {
          this.$alert(
            `<div>
				<img src="${wxTipsPng}" alt="" class="banner">
				<div>微信号首次绑定，请登录或注册</div>
			</div>`,
            {
              confirmButtonText: '确定',
              dangerouslyUseHTMLString: true,
              customClass: 'tips-modal'
            }
          )
          this.wxIds = { openid: res.openid, unionid: res.unionid }
          this.$store.dispatch('wx/saveIds', this.wxIds)
        } else {
          this.$store.dispatch('user/saveToken', res.cook)
          this.checkOrg()
        }
      }
    },
```

3. 扫码回调页面

   ```javascript
   async mounted() {
       this.$loading({
         lock: true,
         text: '微信登录中...',
         spinner: 'el-icon-loading',
         customClass: 'loading',
         background: 'rgba(0, 0, 0, 0.7)'
       })
       // 微信跳转回调地址url带来code和state
       const { code, state } = this.$route.query
       if (code) {
         try {
           const res = await this.$store.dispatch('user/wxLogin', { code, state })
           // 调用主页面的回调
           window.opener.wxLoginComplete(res)
           window.close()
         } catch (err) {
           console.log(err)
           setTimeout(() => {
             window.close()
           }, 1000)
         }
       } else {
         this.$message.error('获取微信授权码失败')
         setTimeout(() => {
           window.close()
         }, 1000)
       }
     }
   ```

   