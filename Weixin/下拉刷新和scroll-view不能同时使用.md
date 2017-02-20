## 下拉刷新和scroll-view不能同时使用问题 <br>
今天在开发小程序中，使用scroll-view组件对后台新闻列表数据进行刷新。本来在电脑端使用14x版本的微信开发工具能正常运行:下拉列表进行刷新，滑动到底部，上拉对下一页数据进行获取，还有返回顶部的图标。结果列，扫二维码用苹果手机运行，are you fucking kidding me ?居然不能下拉刷新，也不能上拉发送请求获取数据。这个心塞啊...>_<||||  <br>
结果列，用度娘查了下，看到了知乎的[这篇文章](https://zhuanlan.zhihu.com/p/24739728?refer=oldtimes),发现小程序现在TM的确实有这个问题啊，我要哭死在厕所了，这个坑为毛又被我踩了。<br>
简而言之，就是scroll-view组件和onPullDownRefresh这下拉刷新不能同时使用问题。无语中，然后可以使用view结合onReachButton这个方法替换scroll-view组件。下面来看看代码的具体如何实现。<br>
<br>

  * news.js文件<br>
  该文件包含新闻列表的方法和属性。注意到onPullDownRefresh已被注释掉，使用onReachButton方法。<br>
```javascript
// pages/news/news.js
Page({
  data: {
    newslist: [],
    scrolltop: null, //滚动位置
    page: 0 //分页
  },
  onLoad: function (options) {
    //后台查询新闻列表数据、分页
    this.fetchNewsData();
  },
  fetchNewsData: function () { //获取新闻列表
    var that = this
    const perpage = 5;
    this.setData({
      page: this.data.page + 1
    })
    const page = this.data.page;
    var nnews = [];
    // for (var i = (page - 1) * perpage; i < page * perpage; i++) {
    //每次查询一条新闻信息数据
    //wx.request
    // }
    //或者每次直接查询10条新闻
    wx.showNavigationBarLoading()
    wx.request({
      url: 'https://wx.excecard.com/xcxdemo/xcx/news/newslist.do',
      data: {
        "page": page,
        "perpage": perpage
      },
      method: 'GET', // OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT
      // header: {}, // 设置请求的 header
      success: function (res) {
        // success
        nnews = res.data
        // console.log('nnews:' + JSON.stringify(nnews))
      },
      fail: function () {
        // fail
      },
      complete: function () {
        wx.hideNavigationBarLoading()
        // complete
        if (nnews.length > 0) {
          that.setData({
            newslist: that.data.newslist.concat(nnews)
          })
          console.log('this.data.newslist>> ' + that.data.newslist)
        }

      }
    })
  },
  scrollLoading: function () {   //滚动加载
    this.fetchNewsData();
  },
  scrollHandle: function (e) {  //滚动事件
    this.setData({
      scrolltop: e.detail.scrollTop
    })
  },
  goToTop: function () { //回到顶部
    this.setData({
      scrolltop: 0
    })
  },
  // onPullDownRefresh: function () {  //下拉刷新
  //   console.log('onPullDownRefresh invoke..')
  //   //清空重置已加载的数据
  //   this.setData({
  //     page: 0,
  //     newslist: []
  //   })
  //   this.fetchNewsData();
  //   //设置请求时间
  //   setTimeout(() => {
  //     wx.stopPullDownRefresh();
  //   }, 1000)

  // },
  
  onReachBottom: function (e) {
    this.fetchNewsData();
  },

})
```
<br>

  * news.wxml文件<br>
  包含原始的使用scroll-view组件和替换后的view组件。
```xml
<view class="container">
      <view class="container-body" >
        <view class="news-list">
          <navigator wx:for="{{newslist}}" wx:key="{{item.id}}" url="../newdetail/newdetail?id={{item.id}}">
            <view class="news-list-item-title">{{item.title}}</view>
            <view></view>
            <view class="news-list-item-time">
              <text class="news-list-item-time-text">{{item.time}}</text>
            </view>
            <view class="news-list-item-profile">
              <text>{{item.profile}}</text>
            </view>
            <image src="{{item.imgurl}}" mode="widthFix" class="news-img"></image>
          </navigator>
        </view>
        <view class="gototop {{scrolltop>200?'active':''}}" bindtap="goToTop"></view>
      </view>
</view>

<!--
  由于现在微信小程序对scroll-view组件和下拉刷新事件不能同时时候，出现问题效果：
  在web开发工具上可以体验scroll-view的下拉刷新和上拉获取新的数据，还能直接返回顶部这效果，但是在手机上实现不了，所以，用view替换scroll-view。
  下面是使用scroll-view在模拟器上成功实现的代码：
  <view class="container">
      <scroll-view class="container-body"  scroll-y="true" scroll-top="{{scrolltop}}" bindscroll="scrollHandle" lower-threshold="50" bindscrolltolower="scrollLoading">
        <view class="news-list">
          <navigator wx:for="{{newslist}}" wx:key="{{item.id}}" url="../newdetail/newdetail?id={{item.id}}">
            <view class="news-list-item-title">{{item.title}}</view>
            <view></view>
            <view class="news-list-item-time">
              <text class="news-list-item-time-text">{{item.time}}</text>
            </view>
            <view class="news-list-item-profile">
              <text>{{item.profile}}</text>
            </view>
            <image src="{{item.imgurl}}" mode="widthFix" class="news-img"></image>
          </navigator>
        </view>
        <view class="gototop {{scrolltop>200?'active':''}}" bindtap="goToTop"></view>
      </scroll-view>
</view>
-->
```
<br>
再使用手机扫码测试的时候，就发现可以进行上拉进行下一页数据加载和顶部下拉刷新功能了。也可以试试使用scroll-view可以看到再电脑端可以运行成功，使用苹果手机扫码就不行了，不知道安卓机子效果怎么样。
