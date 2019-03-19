# 小程序自定义标题栏navBar

------
由于小程序默认提供的navigationBar在一些业务需求上有一定的局限性，自定义标题栏便可以给我们带来很大的拓展空间，本文的自定义标题栏组件基于mpvue开发，并使用了我们自己维护的zens-ui库及z-icon字体库，以下为开发及使用流程。

> * 第一步：全局配置小程序导航栏样式为custom
> * 第二步：设计拆分好要自定义标题栏的样式和属性以及功能
> * 第三步：获取系统信息，用来计算最终标题栏高度
> * 第四步：直接使用自定义组件，根据所需自定义内容灵活设置属性值

### 第一步：全局配置小程序导航栏样式为custom
app.json里更改小程序导航栏样式：([点此查看对应小程序官方文档相关链接](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#全局配置) )
![navstyle-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_navstyle.png)
代码如下：
```css
window: {
  navigationStyle: 'custom'
}
```

### 第二步：设计拆分好要自定义标题栏的样式和属性以及功能
自定义标题栏总高度 = 系统statusBarHeight + 标题栏titleBarHeight
自定义内容：当前页是否是首页，是否显示返回键及其颜色，是否显示home键及其颜色，标题名称及其颜色，整个标题栏的背景颜色
![navbar-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_navbar_logo.png)
HTML:
```html
<div class="nav-bar" :style="{paddingBottom: navHeight+'px'}">
    <div class="nav-con" :style="{backgroundColor: navBackgroundColor}">
      <!-- 状态栏 -->
      <div :style="{height: statusBarHeight+'px'}"> </div>
      <!-- 标题栏 -->
      <div class="flex-container" :style="{height: titleBarHeight+'px'}">
        <div class="icons">
          <text class="icon-arrow-left paddingl30" :style="{color: backColor}" @click="backClick" v-if="hasBack && !isHome"></text>
          <text class="icon-home-slim paddingl30" :style="{color: homeColor}" @click="homeClick" v-if="hasHome && !isHome"></text>
        </div>
        <p class="title-name text-limit1 " :style="{color: titleColor}">{{titleName}}</p>
      </div>
    </div>
  </div>
```
自定义属性：
```javaScript
props: {
    navBackgroundColor: { // 标题栏-颜色
      default: 'white'
    },
    titleColor: { // 标题文字-颜色
      default: 'black'
    },
    titleName: { // 标题文字
      default: '首页'
    },
    backColor: { // 返回键-颜色
      default: 'black'
    },
    hasBack: { // 是否显示[返回键]
      default: true
    },
    homeColor: { // home键-颜色
      default: 'black'
    },
    hasHome: { // 是否显示[home键]
      default: true
    },
    homePath: { // home键的返回路径
      default: '/pages/home/main'
    },
    isHome: { // 当前页是否是首页 - true不显示任何图标
      default: false
    }
  }
```
### 第三步：获取系统信息，用来计算最终标题栏高度
JS:([点此查看wx.getSystemInfo小程序官方文档链接](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getSystemInfo.html) )
```js
onLoad () {
    // 落地页不显示[返回键]
    this.hasBack = getCurrentPages().length !== 1
    const that = this
    // 获取手机系统信息
    wx.getSystemInfo({
      success: res => {
        that.statusBarHeight = res.statusBarHeight
        if (/ios/i.test(res.platform)) {
          that.titleBarHeight = 44 // ios
        } else {
          that.titleBarHeight = 48 // android及其他
        }
        // 自定义标题栏高度 = 状态栏高度 + 标题栏高度
        that.navHeight = res.statusBarHeight + that.titleBarHeight
      }
    })
  }
```
CSS:
```
.nav-bar{
  width: 100vw;
  .nav-con{
    width: 100%;
    position: fixed;
    top: 0;
    z-index: 999; // 注意 - 保持标题栏在最顶
    .title-name{
      position: absolute;
      left:70px;
      right: 70px;
      margin:auto;
      text-align: center;
      font-size: 32rpx；
    }
  }
}
```
### 第四步：直接使用自定义组件，根据所需自定义内容灵活设置属性值
![navbar-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_title.png)
本文的自定义标题栏navBar在我们自己开发的组件库[lemon(点击查看组件库文档地址)](http://120.77.37.44:83/#/)内，直接使用组件需要先安装[lemon](http://120.77.37.44:83/#/)，`欢迎交流使用～`操作如下：
##### 安装组件库`lemon`：
```
npm i --save lemon
```
##### 使用示例代码：
```html
<template>
    <nav-bar :title-name="titleName"></nav-bar>
</template>
<script >
import navBar from 'lemon/src/navBar'
export default {
  compontents: {navBar},
  data () {
    return {
     titleName: '我是标题名称'
    }
  }
}
</script>
```
