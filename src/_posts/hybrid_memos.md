---
title: APP技术选型备忘录
vssue-title: Hybrid APP(H5app)前端脱坑指南
date: 2018-11-08
tags:
  - 大前端
  - hybrid-app
categories: [大前端]
---

[[toc]]

## 类型对比

| 方案   | 开发 | 推广     | 入口       | 体验 | 优点                         | 缺点                                               | 建议                               |
| ------ | ---- | -------- | ---------- | ---- | ---------------------------- | -------------------------------------------------- | ---------------------------------- |
| Native | 很难 | 链路长   | 独立入口   | 最好 | 最佳体验，最优实现           | 开发难度大，周期长                                 | 提前做好人才储备                   |
| Hybrid | 较难 | 链路长   | 独立入口   | 较好 | 快速迭代，可实现几乎所有功能 | 体验接近网页                                       | 需要提前设计好策略和界限           |
| 跨端   | 较难 | 链路长   | 独立入口   | 较好 | 接近原生体验                 | 开发难度不低，坑多                                 | 没原生工程师不要碰                 |
| 服务号 | 容易 | 点击使用 | 微信深入口 | 最差 | 最快开发                     | 入口不好找，体验较差                               | 使用好的前端框架和 UI 框架很有必要 |
| 小程序 | 容易 | 点击使用 | 微信浅入口 | 较差 | 微信中最佳体验               | 体积限制导致按模块分裂，需要多个小程序满足一套业务 | 切割好业务模块，做好合理的跳转策略 |

## 原生方案策略

- 小成本下慎重尝试
- 各端至少需要一个 10 年工作经验的人
- 需要拿出和预期进度相符的研发阵容方可立项

## 混合方案策略

- 各端至少需要一个 10 年工作经验的人
- 需要准备靠谱的前端阵容
- 这个方案只需要人靠谱并没有对数量有要求，所以这个方案最有可能实现少人数开发大项目
- 方式 1：提前制定好渲染策略，制定出简单的 sdk 级别的库，极限目标是尽量少使用原生
- 方案 2：每个界面都由 webview 组成，当然 webview 和原生如何占比根据页面业务，极限情况是整个页面都是 webview。新页面永远由新的页面带着 webview 打开，每个页面保持独立，每个页面都是由 webview 来负责渲染。每个页面的配合策略跟方案 1 不同的是，方案一的目标是抽象出所有页面的动作和桥接，方案 2 是每个页面单独制定策略，所以页面单独定制占位背景，网页打开之前就这个背景，网页打开之前顶部右侧只有一个刷新按钮，页面载入完毕顶部业务按钮出现，当然业务按钮也是定制的，页面刷新可由原生在 webview 的顶部做下拉事件来做动画，刷新就是重新摘入一遍页面，重复先显示占位背景那个过程。每个页面的桥接都是独立定制的。实现这个方案之前首先需要确定的两点是，1 栈能 push 多少个带着 webview 的页面，2 刷新策略和摘入流程策略。这个方案还有一个好处就是岗位人员使用比较灵活，如果原生想要付出更多原生可以给底层实现做的更好，如果原生使不上劲了 web 可以补位，极限情况是有些地方不想复杂做了就可以让一个 web 在 webview 里面不断的打开到底就好了，

## 跨端方案策略

- 真正用到生产环境实际的开发门槛不低
- 必须需要有原生工程师参与
- 目前只有 RN 和 Flutter 靠谱
- 只有特定类型适合偏展示类多的 app，例如电商平台，资讯类等

## 小程序方案策略

- 需要必要的前端团队
- 设计好模块分裂策略
- 建议配合服务号 H5 一起上
- 使用成熟的 UI 框架

## 服务号方案策略

- 需要必要的前端团队
- 微信中更推荐用服务号
- 可同步搞出小工具补充使用
- 使用成熟的前端框架例如 vue 等
- 使用成熟的前端 UI 框架
- 利用特殊宿主环境进行二次创作，例如有赞的类微信菜单

## UI 框架对比

![React，Vue、Angular](https://images.bjshanbang.com/test/vue.jpg)

| UI 框架     | 环境                | Github Star | 团队        | 平台         | 最后提交       | 推荐 |
| ----------- | ------------------- | ----------- | ----------- | ------------ | -------------- | ---- |
| Element     | Vue，React，Angular | 31K         | 饿了么      | PC，H5       | 22 minutes ago | 是   |
| Zent        | React               | 1K          | 有赞        | PC           | 5 hours ago    | 是   |
| Vant        | Vue                 | 5K          | 有赞        | H5           | 4 days ago     | 是   |
| Vant Weapp  | 小程序              | 6K          | 有赞        | 小程序       | 3 hours ago    | 是   |
| Cube-UI     | Vue                 | 4K          | 滴滴        | H5           | a day ago      | 是   |
| iView       | Vue                 | 17K         | TalkingData | PC，小程序   | 3 days ago     | 是   |
| Ant Design  | React，RN           | 33K         | 蚂蚁金服    | PC，H5，原生 | 18 hours ago   | 是   |
| NG-ZORRO    | Angular             | 3K          | 蚂蚁金服    | PC           | 2 days ago     | 是   |
| Vue Antd    | Vue                 | 1K          | 蚂蚁金服    | PC           | 3 years ago    | 否   |
| VUX         | Vue                 | 14K         | 个人        | H5           | 2 months ago   | 是   |
| Material-UI | React               | 40K         | 国外        | PC，H5       | a day ago      | 是   |
| Vuetify     | Vue                 | 13K         | 国外        | PC，H5       | 2 days ago     | 是   |