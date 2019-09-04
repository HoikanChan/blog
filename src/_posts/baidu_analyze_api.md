因项目运营需求前端需要埋点，统计页面流量。本文指针对H5应用，，若是App可参考[百度Mota](https://link.juejin.im?target=http%3A%2F%2Fmota.baidu.com%2F)。

## 统计几个指标：

- pv:用户每打开一个网站页面就被记录1次。用户多次打开同一页面，浏览量值累计。
- uv:一天之内您网站的独立访客数(以Cookie为依据)，一天内同一访客多次访问您网站只计算1个访客。
- IP/iv(独立IP)：某IP地址的计算机访问网站的次数

首先注册百度统计账号，然后按需求添加统计应用信息

## 1、pv和uv统计（_trackPageview）



![img](https://user-gold-cdn.xitu.io/2018/3/2/161e57955c253f93?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 确定后生成一段代码片段





![img](https://user-gold-cdn.xitu.io/2018/3/2/161e579b5c091d37?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 将代码按提示引入到页面 ，  如果代码安装正确，一般20分钟后，可以查看网站分析数据,



```
// vue router
router.beforeEach((to, from, next) => {
    // 统计代码
    // _hmt.push(['_trackPageview', pageURL]) 必须是以"/"（斜杠）开头的相对路径
    if (to.path) window._hmt.push(['_trackPageview', '/#' + to.fullPath])
    next()
})
复制代码
```

在控制台中查看统计是否发送成功



![img](https://user-gold-cdn.xitu.io/2018/3/2/161e613865f86d9f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

 可以在网站概况中查看（ 生效时间会几十分钟到几个小时的延迟~略坑





![img](https://user-gold-cdn.xitu.io/2018/3/2/161e61a3433bfa69?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 2、 自定义事件统计( _trackEvent)

除了pv和uv两个量之外，最常用的是按钮和其他节点用户事件的处理，因此为了满足需求一般项目需要自定义事件，也有另一个种情况，统计项目中插入用户的第三方页面，事件统计第三方页面的pv，uv

```
_hmt.push(['_trackEvent', category, action, opt_label, opt_value]);
复制代码
```

###          参数

| 名称      | 必选/可选 | 类型   | 功能                         | 备注                      |
| --------- | --------- | ------ | ---------------------------- | ------------------------- |
| category  | 必选      | String | 要监控的目标的类型名称       | 不填、填"-"的事件会被抛弃 |
| action    | 必选      | String | 用户跟网页进行交互的动作名称 | 不填、填"-"的事件会被抛弃 |
| opt_label | 可选      | String | 事件的一些额外信息           | 不填、填"-"代表此项为空   |
| opt_value | 可选      | Number | 跟事件相关的数值             |                           |

### 事件注意点（量大的时候，很少用到）

- 百度统计针对_trackevent API 目前有多样性的限制，即在部署_trackevent API时，事件类型、操作、标签三项的多样性乘积不能超过10000，否则系统会自动抛弃超标的数据。
- 事件分析报告中将展示事件点击总数在前3000名的事件
- 不符合前两条，在报告中会出现提示，请修改代码，使得您代码中传递的参数符合要求

```
// vue component
methods:{
    goUser () {
        window._hmt.push(['_trackEvent', 'loginAction', 'Login', this.name])
    }
}
复制代码
```

在百度统计管理台中查看数据，初次使用时间会有一定延迟



![img](https://user-gold-cdn.xitu.io/2018/3/5/161f4fda7b0f38d5?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 3、追踪用户行为（_setCustomVar）

指定一个自定义变量，用于追踪用户使用行为等。前面两种普通需求够用，这个统计个人没用到过^_^，可能部分业务需求还是需要使用这个方法，可以优化用户体验
 `_hmt.push(['_setCustomVar', index, name, value, opt_scope]);`

###  参数

| 名称      | 必选/可选 | 类型   | 功能                   | 备注                                                         |
| --------- | --------- | ------ | ---------------------- | ------------------------------------------------------------ |
| index     | 必选      | Int    | 自定义变量所占用的位置 | 索引的范围是从1到5                                           |
| name      | 必选      | String | 自定义变量的名字       | 每个索引对应的名字在使用一次后就会固定 以后无法更改          |
| value     | 必选      | String | 自定义变量的值         |                                                              |
| opt_scope | 可选      | Int    | 自定义变量的作用范围   | 1为访客级别（对该访客始终有效） 2为访次级别（在当前访次内生效） 3为页面级别（仅在当前页面生效） 默认为3 |



### opt_Scope参数说明：

setCustomVar的数据报表是基于session的统计，包含了使用jsapi的页面的访次都会在不同的槽位上 (index) 被打上数个形如name=value的标签，而opt_scope参数则定义了这些标签的有效期。opt_scope的合法可选值有3个，分别表示了3不同的级别：

- 访客级别：为当前访客打上标签，此访客以后的所有访问的指定槽位(index)上都会打上这个标签。
- 访次级别：为当前访次打上标签，只有当前访次的指定槽位(index)会打上这个标签，不影响此访客的后续访问
- 页面级别：给当前访次打上第一个有标签页面的标签。只有当前访次的指定槽位(index)会打上这个标签，不影响此访客的后续访问

### opt_scope参数的覆盖规则：

由于对一个访次来说每一个槽位(index)最终只会保留一个标签，所以当一个访次的同一个槽位(index)上出现多个标签时，将出现冲突的情况。对一个访次，统计逻辑会根据以下三步来决定一个访次的最终标签：

- 先判断槽位的级别，判断规则：槽位中有访客级别标签为访客级别槽位，槽位中有访次级别标签且无访客级别标签的为访次级别槽位，槽位中既无访客级别标签又无访次级别标签的为页面级别槽位
- 如果是访次级别槽位，以时序上最后一个访次级别标签为最终标签
- 如果是页面级别槽位，以时序上第一个页面级别标签为最终标签

建议jsapi使用尽量把不同级别的标签设置在不同的槽位(index)上，以免造成标签冲突引起的数据不一致。

### opt_scope参数使用建议

- 在需要对不同类别的访客今后的一系列行为做区分筛选的时候建议使用访客级别，比如“否是VIP会员”等标签；
- 在需要对本访次的用户行为或状态做区分筛选的时候建议使用访次级别，比如“是否登陆”等标签；
- 在需要对本访次的访问内容或访问路径做区分筛选的建议使用页面级别，比如看了“体育频道”还是“财经频道”等标签

在login.vue中进行用户角色属性vip，登录状态，登录事件的jsApi

```
// login.vue
goUser () {
      window._hmt.push(['_trackEvent', 'loginAction', 'Login', 'ld'])
      window._hmt.push(['_setCustomVar', 1, 'vip', 'true', 1])
      window._hmt.push(['_setCustomVar', 2, 'login', 'true', 2])
      this.$router.push({path: '/talk'})
}

// talk.vue

mounted () {
    window._hmt.push(['_setCustomVar', 3, 'talk', 'joy or enjoy', 3])
  },
复制代码
```

在控制台中查看cv参数



![img](https://user-gold-cdn.xitu.io/2018/3/6/161f92aa27b32f46?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



在统计后台中查看



![img](https://user-gold-cdn.xitu.io/2018/3/6/161f9dd7b05a5dcf?imageslim)





![img](https://user-gold-cdn.xitu.io/2018/3/6/161f9dc63af261ff?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



## 4、通过api获取数据（后台迁移数据部分）

一般统计uv和pv是在我们自己的管理后台中进行查看，需要后台调用Tongji api移植数据 [官方文档](https://link.juejin.im?target=https%3A%2F%2Ftongji.baidu.com%2Fopen%2Fapi%2Fmore%3Fp%3Dtongjiapi_usage.tpl)

一、首先您需要使用百度统计的帐号、密码、以及开通数据服务后提供的TOKEN码，通过DR-API的身份验证。

二、然后通过getSiteList获取当前用户下的站点和子目录列表以及对应参数信息，不包括权限站点和汇总网站。

您需要使用正确的站点ID（网站或系统中安装了哪个站点的代码，就选择哪个站点）后，方可使用getData服务导出正确的报告数据。

- 接口地址：

```
https://api.baidu.com/json/tongji/v1/ReportService/getSiteList
```

- 返回参数

| 参数名 | 参数类型          | 是否必需 | 描述     |
| ------ | ----------------- | -------- | -------- |
| list   | array of SiteInfo | 是       | 站点列表 |

- SiteInfo结构

| 参数名       | 参数类型            | 是否必需 | 描述                         |
| ------------ | ------------------- | -------- | ---------------------------- |
| site_id      | uint                | 是       | 应用ID                       |
| domain       | string              | 是       | 站点域名                     |
| status       | uint                | 是       | 0：正常；1：暂停             |
| create_time  | DateTime            | 是       | 日期时间格式，以北京时间表示 |
| sub_dir_list | array of SubDirInfo | 是       | 子目录列表                   |

- SubDirInfo结构

| 参数名       | 参数类型            | 是否必需 | 描述                         |
| ------------ | ------------------- | -------- | ---------------------------- |
| sub_dir_id   | uint                | 是       | 子目录ID                     |
| name         | string              | 是       | 子目录名称                   |
| status       | uint                | 是       | 0：正常；1：暂停             |
| create_time  | DateTime            | 是       | 日期时间格式，以北京时间表示 |
| sub_dir_list | array of SubDirInfo | 是       | 子目录列表                   |

### 三、getData

根据站点ID获取站点报告数据。

确认所需的站点ID后，根据您的数据分析需求指定报告、指标、筛选维度，使用getData获取数据即可。

- 接口名称

```
https://api.baidu.com/json/tongji/v1/ReportService/getData
复制代码
```

- 输入参数

| 参数名      | 参数类型 | 是否必需 | 描述                                                    |
| ----------- | -------- | -------- | ------------------------------------------------------- |
| site_id     | uint     | 是       | 站点id                                                  |
| method      | string   | 是       | 通常对应要查询的报告                                    |
| start_date  | string   | 是       | 查询起始时间,例：20160501                               |
| end_date    | string   | 是       | 查询结束时间,例：20160531                               |
| start_date2 | string   | 否       | 对比查询起始时间                                        |
| end_date2   | string   | 否       | 对比查询结束时间                                        |
| metrics     | string   | 是       | 自定义指标选择，多个指标用逗号分隔                      |
| gran        | string   | 否       | 时间粒度(只支持有该参数的报告): day/hour/week/month     |
| order       | string   | 否       | 指标排序，示例：visitor_count,desc                      |
| start_index | uint     | 否       | 获取数据偏移，用于分页；默认是0                         |
| max_results | uint     | 否       | 单次获取数据条数，用于分页；默认是20; 0表示获取所有数据 |

- ### 所有页面pv、uv、iv查看请求示例：（没有走后台流程那一套直接在百度统计后台截取的请求）

数据格式大体一样，返回值格式也一样，indicators对应metrics



![img](https://user-gold-cdn.xitu.io/2018/3/6/161f9655e2123488?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



返回值



![img](https://user-gold-cdn.xitu.io/2018/3/6/161f968b10715188?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



事件



![img](https://user-gold-cdn.xitu.io/2018/3/6/161f9df630ba14c8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



用户行为 

![img](https://user-gold-cdn.xitu.io/2018/3/6/161f9e42b5ae7d57?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


