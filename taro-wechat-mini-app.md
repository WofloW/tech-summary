# Taro写微信小程序

10/10/2019

taro版本：1.3.14

之前写react的写taro还是蛮顺手的

## 建议同时打开
https://developers.weixin.qq.com/miniprogram/dev/framework/subpackages/basic.html

https://taro-docs.jd.com/taro/docs/README.html

https://taro-ui.aotu.io/#/docs/introduction

三个文档并列，1是微信小程序的官方文档，2是taro官方文档，3是taro ui的文档

taro.**很多都是直接转成wx.**，所以遇到这种情况直接查微信小程序文档。

## 遇到的问题和解决方法：

wx.request没有patch method，建议在后端包一个put转patch的

taro.request的statusCode 400 500也算success，所以我看到的都是建议这样包一下
```
Taro.request({
      url,
      data,
      method: 'POST'
    }).then((res) => {
      let {statusCode} = res
      if (statusCode >= 200 && statusCode < 300) {
        return res
      } else {
        throw new Error(`网络请求错误，状态码${statusCode}`)
      }
    })
```

taro的showToast紧跟着路由跳转，就会看不到toast，建议路由跳转success回调里使用showToast

taro的editor没有文档，其实可以在@tarojs/component里直接引用，使用方法如同微信小程序的editor

taro ui的form onsubmit没有内容，官方文档：onSubmit 事件获得的 event 中的 event.detail.value 始终为空对象，开发者要获取数据，可以自行在页面的 state 中获取

先开taro build --type weapp --watch，再开微信小程序wechat devtools，否则会出错

有时候写错代码了，改对了，taro rebuild的时候会说这个文件未被引用到，就再也不编译了，只能手动重新rebuild

fab button也就是浮动按钮并不会固定位置，需要自己手动position: fixed定位

fab button里样式class的at-fab__icon at-icon at-icon-add需要手动开启global class

```
class Index extends Component {
  static options = {
    addGlobalClass: true
  }
}
```

request response header.Content-Range在模拟器和安卓是一样的，在iOS上是content-range，这个还蛮坑的
