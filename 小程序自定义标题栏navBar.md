# 小程序自定义标题栏navBar

------
由于小程序默认提供的navigationBar在一些业务需求上有一定的局限性，自定义标题栏便可以给我们带来很大的拓展空间，本文的自定义标题栏组件基于mpvue开发，并使用了我们自己维护的zens-ui库及z-icon字体库，以下为开发及使用流程。

> * 第一步：全局配置小程序导航栏样式为custom
> * 第二步：设计拆分好要自定义标题栏的样式和属性以及功能
> * 第三步：主要代码：获取系统信息，用来计算最终标题栏高度
> * 第四步：注意事项及遇到问题
> * 第五步：直接使用自定义组件，根据所需自定义内容灵活设置属性值

### 第一步：全局配置小程序导航栏样式为custom
main.js里更改小程序导航栏样式(原生开发小程序在app.json内更改)，custom自定义导航栏，只保留右上角胶囊按钮：([点此查看对应小程序官方文档相关链接](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#全局配置) )

![navstyle-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_navstyle.png)
代码如下：
```css
window: {
  navigationStyle: 'custom'
}
```

### 第二步：设计拆分好要自定义标题栏的样式和属性以及功能
如图所示：
自定义标题栏总高度 = 系统statusBarHeight + 标题栏titleBarHeight
自定义内容：当前页是否是首页，是否显示返回键及其颜色，是否显示home键及其颜色，标题名称及其颜色，整个标题栏的背景颜色
![navbar-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_navbar_logo.png)

###属性 Properties
| 参数       | 说明     | 类型      | 可选值       | 默认值   |
|---------- |-------- |---------- |-------------  |-------- |
| navBackgroundColor | 菜单栏颜色 | string | — | white |
| titleColor      | 标题文字颜色 | string | — | black |
| titleName | 标题名称 | string | — | 首页 |
| backColor      | 返回键颜色 | string | — | black |
| hasBack | 是否显示返回键 | boolean | — | true |
| homeColor      | home键颜色 | string | — | black |
| hasHome | 是否显示home键 | boolean | — | true |
| homePath      | home键的路径 | string | — | /pages/home/main |
| isHome      | 当前页是否是首页 | boolean | — | false |

### 事件 Events
| 事件名称 | 说明 | 回调参数 |
|---------- |-------- |---------- |
| backClick  | 点击返回键,返回上一页 | — |
| homeClick  | 点击home键盘,返回首页homePath | — |
| checkFIrstPage  | 是否是第一个页面 | — |
### 第三步：主要代码：获取系统信息，用来计算最终标题栏高度
HTML:
```html
<template>
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
</template>
```
JS：([点此查看wx.getSystemInfo小程序官方文档链接](https://developers.weixin.qq.com/miniprogram/dev/api/wx.getSystemInfo.html) )
```js
<script>
export default {
  name: 'navBar',
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
  },
  data () {
    return {
      navHeight: 0,
      statusBarHeight: 0,
      titleBarHeight: 0
    }
  },
  methods: {
    backClick () {
      wx.navigateBack({
        delta: 1
      })
    },
    homeClick () {
      wx.reLaunch({
        url: this.homePath
      })
    },
    checkFIrstPage () {
      this.hasBack = getCurrentPages().length !== 1
    }
  },
  onLoad () {
    this.checkFIrstPage()
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
}
</script>
```
CSS:
```css
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
### 第四步：注意事项及遇到问题
1.下拉小程序的时候发现，自定义组件也被下拉下来，没有固定在顶部不动。
解决方式：给`nav-con`添加`position: fixed;top: 0;`
2.又发现自定义标题栏在顶部定住之后，遮挡页面的顶部内容，如图所示：
![navbar-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_cover_page.png)
解决方式：给`nav-bar`添加` :style="{paddingBottom: navHeight+'px'}"`,距离底部一个标题栏的高度。
3.标题栏被页面z-index高的组件遮挡
解决方式：给`nav-con`添加z-index: 999; // 提示-保持标题栏在最顶，页面最大z-index小于999
4.标题名称过长进行优化处理，超过最大宽度范围进行省略隐藏

### 第五步：直接使用自定义组件，根据所需自定义内容灵活设置属性值
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

参考文章：[https://www.jianshu.com/p/d67ee748445b](https://www.jianshu.com/p/d67ee748445b)
