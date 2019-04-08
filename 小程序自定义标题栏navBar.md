# 小程序自定义标题栏navBar

------
### 业务场景：
小程序自带标题栏只有返回按钮，我们的业务需求是返回按钮和返回首页按钮根据场景组合出现，方便用户直达首页，于是我们便封装了自定义标题栏navBar，以下代码示例是基于mpvue开发的。

> * 第一步：全局配置小程序导航栏样式为custom
> * 第二步：设计拆分好要自定义标题栏的样式和属性
> * 第三步：自定义标题栏代码如下
> * 第四步：使用自定义标题栏示例，可以根据所需业务需求灵活设置属性值

### 第一步：全局配置小程序导航栏样式为custom
main.js里更改小程序导航栏样式(原生小程序开发在app.json内更改)，custom自定义导航栏，只保留右上角胶囊按钮：([点此查看对应小程序官方文档相关链接](https://developers.weixin.qq.com/miniprogram/dev/framework/config.html#全局配置) )

![article_navstyle](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/article_navstyle.png)
代码如下：
```css
window: {
  navigationStyle: 'custom'
}
```
设置custom后小程序所有的页面都需要自定义标题栏，只显示右上角胶囊按钮，当前页面效果如下：
![navbar_custom](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/navbar_custom.png)

### 第二步：设计拆分好要自定义标题栏的样式和属性以及功能
如图所示：
自定义标题栏总高度 = 系统statusBarHeight + 标题栏titleBarHeight
自定义内容：当前页是否是首页，是否显示返回键及其颜色，是否显示home键及其颜色，标题名称及其颜色，整个标题栏的背景颜色

![navbar-logo](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/navbar_logo.png)

### 第三步：代码如下
HTML:
```html
<template>
  <!-- paddingBottom: navHeight+'px' - 距离底部一个标题栏的高度，解决自定义标题栏遮挡页面显示的问题 -->
  <div class="nav-bar" :style="{paddingBottom: navHeight+'px'}">
    <div class="nav-con" :style="{backgroundColor: navBackgroundColor}">
    
      <!-- 状态栏 -->
      <div :style="{height: statusBarHeight+'px'}"> </div>
      
      <!-- 标题栏 -->
      <div class="flex-container" :style="{height: titleBarHeight+'px'}">
        <!-- 返回键和home键图标 -->
        <div class="icons marginl20 flex-container " v-if="(hasBack || hasHome) && !isHome">
          <text class="icon-arrow-left paddingh30 marginv15" :style="{color: backColor}" @click="backClick" v-if="hasBack"></text>
          <p class="text-lighter" style="width:1rpx;" v-if="hasBack && hasHome">|</p>
          <text class="icon-home-slim paddingh30 marginv15" :style="{color: homeColor}" @click="homeClick" v-if="hasHome"></text>
        </div>
        <!-- 标题文字 -->
        <p class="title-name text-limit1 fs-36" :style="{color: titleColor}">{{titleName}}</p>
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
    titleName: { // 标题文字内容
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
    homePath: { // 点击home键的返回路径
      default: '/pages/home/main'
    },
    isHome: { // 当前页是否是首页 - true不显示[返回键]和[home键]
      default: false
    },
    isTransparent: { // 是否是沉浸式标题栏
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
    // 点击[返回键]返回上一页
    backClick () {
      wx.navigateBack({
        delta: 1
      })
    },
    // 点击[home键]回到首页的路径地址
    homeClick () {
      wx.reLaunch({
        url: this.homePath
      })
    }
  },
  onLoad () {
    // 当前页为第一个页面时，不显示返回键(例如：落地页，首页等)
    this.hasBack = getCurrentPages().length !== 1

    const that = this
    // 获取手机系统信息，用来计算及设置自定义标题栏的高度
    wx.getSystemInfo({
      success: res => {
        that.statusBarHeight = res.statusBarHeight // 状态栏的高度
        
        // ios和android的标题栏高度不同，需要单独设置
        if (/ios/i.test(res.platform)) {
          that.titleBarHeight = 44 // ios标题栏高度
        } else {
          that.titleBarHeight = 48 // android及其他客户端标题栏高度
        }
        
        // 最终的自定义标题栏高度 = 状态栏高度 + 标题栏高度
        that.navHeight = res.statusBarHeight + that.titleBarHeight
        console.log(that.navHeight)
      }
    })
    
    // 设置沉浸式标题栏，自定义标题栏变为透明
    if (this.isTransparent) {
      this.navHeight = 0
      this.navBackgroundColor = 'transparent'
    }
  }
}
</script>
```
CSS:
```css
<style lang="scss" scoped>
.nav-bar{
  width: 100vw;
  .nav-con{
    width: 100%;
    position: fixed; // 注意-固定自定义标题栏，解决下拉小程序时自定义标题栏也被拉下来的问题
    top: 0;
    z-index: 999999; // 注意-解决保持标题栏在页面最顶层不被覆盖的问题
    .icons{
      border: 2rpx solid #F6F6F6;
      border-radius: 34rpx;
    }
    .title-name{
      position: absolute;
      left: 105px;
      right: 105px;
      margin: 0 auto;
      text-align: center;
      font-size: 32rpx；
    }
  }
}
</style>
```
### 第四步：直接使用自定义组件，根据所需自定义内容灵活设置属性值
##### 1.非首页使用示例代码：
```html
<template>
    <nav-bar title-name='商品'></nav-bar>
</template>
<script >
import navBar from 'lemon/src/navBar'
export default {
  compontents: {navBar},
  data () {
    return {
    }
  }
}
</script>
```
页面效果如图所示：

![navbar_default](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/navbar_default.png)

##### 2.首页及tabbar页面使用示例代码：
（不显示左上角返回键和home键的页面isHome均设置为true，如:小程序底部tabBar页面）
```html
<nav-bar :is-home="true"></nav-bar>
```
页面效果如图所示：

![navbar_home](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/navbar_home.png)

##### 3.沉浸式标题栏使用示例代码：
```html
<nav-bar title-name='员工中心' is-transparent='true' title-color='white' home-color='white'></nav-bar>
```
页面效果如图所示：

![navbar-transparent](https://zens-pic.oss-cn-shenzhen.aliyuncs.com/static/gift/navbar-transparent.png)


参考文章：[https://www.jianshu.com/p/d67ee748445b](https://www.jianshu.com/p/d67ee748445b)
