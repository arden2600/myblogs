## xcx-tabBar与navigateTo同url页面无法跳转问题<br>
最近在玩微信小程序开发，突然发现了一个问题：<br>
若是在全局app.json中配置了`tabBar`。那么在开发中调用`wx.navigateTo`接口时候，若是**跳转的url与tabBar中list页面中引用同样的页面路径**，那么结果会是tabBar页面可以访问，但是**navigateTo页面无法跳转**。<br>
web开发工具版本:`0.12.130400`, 下面看看代码内容:<br>
其中日志对应tabBar对应的页面为路径`pages/logs/logs`。
<br>
  * app.json<br>
```json
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#fff",
    "navigationBarTitleText": "WeChat",
    "navigationBarTextStyle":"black"
  },
  "tabBar": {
    "selectedColor": "#aaaaaf",
    "borderStyle": "white",
    "list": [{
      "pagePath": "pages/index/index",
      "text": "首页"
    }, {
      "pagePath": "pages/logs/logs",
      "text": "日志"
    }]
  }
}
```
<br>
  * pages/index/index.js中通过modal进行页面跳转<br>
  下面代码中，通过wx.navigateTo()页面跳转，注意到其中的url页面地址与app.json中日志页面路径相同。
```javascript
//index.js
//获取应用实例
var app = getApp()
Page({
  data: {
    motto: 'Hello World',
    userInfo: {}
  },
  //事件处理函数
  bindViewTap: function () {
    wx.showModal({
      title: '提示',
      content: '这是一个测试弹窗',
      confirmText: "yes",
      cancelText: "no",
      showCancel: false,
      success: function (res) {
        if (res.confirm) {
          wx.navigateTo({
            url: '../logs/logs',  //与tabBar中日志页面相同
            success: function (res) {
              // success
              console.log('success')
            },
            fail: function () {
              // fail
              console.log('fail')
            }
          })
        }
      }
    })

  },
  onLoad: function () {
    console.log('onLoad')
    var that = this
    //调用应用实例的方法获取全局数据
    app.getUserInfo(function (userInfo) {
      //更新数据
      that.setData({
        userInfo: userInfo
      })
    })

    wx.showToast({
      title: '加载中',
      icon: 'loading',
      duration: 10000
    })

    setTimeout(function () {
      wx.hideToast()
    }, 2000)

  }
})
```
<br>
  * 当点击头像进行测试<br>
  当点击头像时候，会在调试页面中输出：fail (即调用wx.navigateTo这个api失败)<br>
  若是任意修改app.json中tabBar中日志栏的pagePath(只要不与navigateto跳转页相同)，那么可以成功跳转。只是现在小程序版本有这个问题，后面估计还有很多坑等着去踩，疼啊。~~
