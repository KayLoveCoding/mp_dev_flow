***登录篇***
*1.客户端*
1.调用wx.checkSession->success则表示session没有过期，fail表示session已过期。将结果放到header的Authorization中，告诉服务端是否需要重新调用微信接口还是直接查库返回用户数据
2.调用wx.login，将code传给客户端的**登录接口**，记得传入上个流程知道的Authorization标识。
*2.服务端*
1.**登录接口**：获取code和Authorization，通过Authorization是否有值判断是直接查库返回用户信息，还是调用微信
`https://api.weixin.qq.com/sns/jscode2session?appid=${WECHAT_APPID}&secret=${WECHAT_APPSECRET}&js_code=${code}&grant_type=authorization_code`;
这个接口去获取用户信息，需要返回一个微信的token给前端，前端储存起来作为Authorization，再次进入时再校验。
*3.客户端*
1.走完以上流程，调用wx.getUserInfo方法，并设置withCredentials: true,成功后将res.userInfo和Authorization传给服务端**获取用户信息**接口
*4.服务端*
1.**获取用户信息**：本接口要从客户端传来的userInfo结合自身逻辑，把微信用户加入内部用户。
客户端返回的明文内容将不包含openid,unionid等敏感数据。开发者如需要获取敏感数据，需要对接口返回的加密数据(encryptedData) 进行对称解密。参考文档如下（有demo）：
https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/signature.html
解密后，服务端能获取用户的openid等数据，根据自身后台逻辑，查出该用户在本平台是否绑定过手机号，返回给客户端用户数据及token等（同公众号）
*5.客户端*
通过**获取用户信息**接口的返回结果，判断当前用户是否绑定过手机号。如果没有，强制登录。登录绑定接口可以调用公众号的登录接口,
所需参数mobile，code，openId，nickname，unionId等.
