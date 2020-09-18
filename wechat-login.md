# 微信开放平台 网页微信扫码登录（OAuth2.0）

参考这个网页做的
https://www.cnblogs.com/0201zcr/p/5133062.html

## 需要的前提
- 申请网站应用
- 申请开发者资质认证

两个可以同时进行

### 申请网站应用
在线填表：

- 网站应用名称
- 简介
- 应用官网
- 下载《微信开放平台网站信息登记表》填写盖章签字（需要备案的域名）
- 网站应用图标

### 申请认证
- 对公账户打钱
- 营业执照
- 最后认证费￥300

两个申请都通过后即可拿到AppID和AppSecret

## 授权流程

参考[官方文档](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)
![流程图](https://res.wx.qq.com/op_res/D0wkkHSbtC6VUSHX4WsjP5ssg5mdnEmXO8NGVGF34dxS9N1WCcq6wvquR4K_Hcut "流程图")

简洁说流程
1. 拿code
```
GET https://open.weixin.qq.com/connect/qrconnect?appid=APPID&redirect_uri=REDIRECT_URI&response_type=code&scope=snsapi_login&state=STATE#wechat_redirect
```
redirect_uri记得UrlEncode一下

只有state不是必填的，用于保持请求和回调的状态，授权请求后原样带回给第三方。该参数可用于防止csrf攻击（跨站请求伪造攻击），建议第三方带上该参数，可设置为简单的随机数加session进行校验

这一步请求发出去后，会返回一个页面，页面中有二维码，扫码后在手机微信，点确认登陆，然后当前页面会redirect去<REDIRECT_URI>，如果刚才确认登陆了，就会带着code和state，如果拒绝登陆了，只会带着state跳转

如果想提高体验，不跳到微信域名，也可以内嵌微信提供的js脚本，然后iframe来显示二维码

详情见[官方文档](https://developers.weixin.qq.com/doc/oplatform/Website_App/WeChat_Login/Wechat_Login.html)

2. 拿token
```
GET https://api.weixin.qq.com/sns/oauth2/access_token?appid=APPID&secret=SECRET&code=CODE&grant_type=authorization_code
```

返回值示例
{ 
  "access_token":"ACCESS_TOKEN", 
  "expires_in":7200, 
  "refresh_token":"REFRESH_TOKEN",
  "openid":"OPENID", 
  "scope":"SCOPE",
  "unionid": "o6_bmasdasdsad6_2sgVt7hMZOPfL"
}


3. refresh token
    1. 若access_token已超时，那么进行refresh_token会获取一个新的access_token，新的超时时间；
    2. 若access_token未超时，那么进行refresh_token不会改变access_token，但超时时间会刷新，相当于续期access_token。

```
https://api.weixin.qq.com/sns/oauth2/refresh_token?appid=APPID&grant_type=refresh_token&refresh_token=REFRESH_TOKEN
```

4. 通过access_token调用接口

授权作用域（scope）	接口	接口说明

snsapi_base	/sns/oauth2/access_token	通过code换取access_token、refresh_token和已授权scope

snsapi_base	/sns/oauth2/refresh_token	刷新或续期access_token使用

snsapi_base	/sns/auth	检查access_token有效性

snsapi_userinfo	/sns/userinfo	获取用户个人信息





