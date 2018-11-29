微信/支付宝监控个人收款，无需签约支付宝、微信支付。为支付宝、微信支付的个人账户提供即时到账服务  

### :musical_score:  功能   
* 收款直接到账你的微信或支付宝，不经过第三方中转  
* 免签约、免备注、免手续费、免提现费  
* 支持PC、手机WAP、公众号、微信内进行支付  
* 自动识别后台上传的二维码图片，自动判断微信或支付宝，同时解析出金额和二维码URL  
* 强大的通知回调，可采用传统的notifyurl实时通知方式  
* 强大的后台可配置选项，可任意自定义提示文字内容、回调地址、通知地址等配置  
* 强大的订单统计图表分析功能，按天、周、月统计订单信息  
* 支持他人收款到账通知，可同时配置多个微信收款账号  
### :bar_chart:  xposed  

使用 xposed 监控收款通知，需要安装[xposed](http://repo.xposed.info/)监控通知。手机的设置很重要，需要把各种权限打开，让你要监控的目标把通知打开，判断标准就是，听到语音播报就可以 。  
###### 安装   

xposed必要安装在已Root的手机或者免 Root的virtualxposed的双开系统中安装。  

###### 原理  

xposed监控到支付宝到账100元，会把该监控到的通知信息推送给Xpay支付系统，推送消息的大体内容如下

```
{
  'type' => 'alipay',
  'token' => 'f75xbeHB+nblekV...AiyPt+eu0SOpGn6o=',
  'money' => '0.01',
  'dt' => '1537408518027',
  'account' => 'tinywan@tinywan.com',
  'sign' => '74d6446c9298bcba3455e5dab056e3a9',
}
```
> 1、验证请求字段的正确性 ，消息类型（转账）、支付类型（微信/支付宝）
>
> 2、验证token（RSA加密/解密）有效性。判断当前token的APP是否登录
>
> 3、根据商户ID（userids）、支付预定单号（mark）、订单金额（money）、未支付订单（status = 0）查询是否有该预订单生成 
>
> 4、否，直接返回给APP相应的错误信息。是，直接处理该订单信息，返回给APP成功信息  

### :game_die:  预支付二维码生成   

###### 环境配置  

配置ngrok本地代理  

###### 固定金额  

通过请求接口返回的金额，生成固定金额的预支付二维码（预设数量进行生成，eg：10）。通过隧道连接 Xpay APP，调用支付宝/微信生成预支付二维码。把预支付二维码的链接地址以及预生成订单号同步返回给Xpay支付系统保存在数据库中。这时候该预支付二维码就会和商户、收款账户以及其他信息帮困在一起了。  

###### 阶梯金额   

生成该金额，必须的现在商户后台配置，金额与生成数量。然后根据收款账户批量生成对应的预支付二维码，为了使预支付二维码能后多次时候，该预支付二维码会在生成订单的同时增加一个预支付二维码的过期时间（如：7分钟之后，该订单将会自动失效或者不可以进行再次支付了），  

### :hotsprings:  收款码金额识别  

二维码的金额需要入数据库才可以在用户订单选择5元的时候,展示5.01元的收款码,如果让客户一张张的手动输入金额.那还用程序员干啥?  
微信支付宝的收款码金额识别:
框架是没有.但是收费API有.可以把收款码上传给API,阿里云/腾讯云/百度云 都有文字识别API.腾讯云一天1000是免费的.可以用.  
这里使用的是腾讯云的OCR,识别二维码的收款金额;调用API即可获取二维码上的所有文字.找到￥符号.正则匹配即可.

### :beginner: 补单  

不怕一万就怕万一.如果哪个订单收到钱而未触发订单完成.那么就需要手动完成一下订单咯.(比如关机,断网,都有可能导致)  
订单管理,列出了自己的所有订单.看看有没有失败的订单(收到钱,没发货).手动点一下完成即可  

### :question:  常见问题   

:speech_balloon:  1、如何设置扫码后免输入金额进行支付？  
>在后台个人收款->产品管理 添加一个我们的产品，并设置一个产品价格，然后我们继续上传这个产品的价格二维码，比如我们产品是50元，我们可以上传50,49.99,49.98,49.97等等二维码,上传的二维码数量越多,越不容易出现让用户手动输入价格  

:speech_balloon:  2、支付完成后页面上未提示成功，后台也显示未到账？  
> 1、检查后台漏单管理中是否有数据(如果有数据则可以排除安卓客户端的配置问题了,下面几项都不用查看了,只需检查支付金额和提示金额是否一致,是否超时支付这两项)
2、手机网络是否畅通，是否能正常访问配置的API地址，收款客户端是否配置成功
3、微信支付宝是否开启到账语音提醒
4、手机通知栏是否有微信或支付到收款到账的通知(手机通知栏有到账通知是必要前提)
5、检查免签收款客户端是否有读取通知栏的权限，微信和支付宝是否正常启动。
6、其次可以尝试将手机设置为不待机，屏幕常亮、设置保留微信、支付宝、免签收款客户端进程。
7、检查你的回调处理是否返回正常
8、收款手机的微信和支付宝请勿停留在应用首屏、二维码页面、微信收款助手页面(此时不会有通知栏通知)
9、检查你是否登录了微信PC版从而设置了微信手机端静音，请务必关闭
10、极少数微信账号会收不到通知，这和你的账号有关，请尝试换个微信账号试下
11、免签收款客户端配置成功后，稍等2~5分钟再进行测试，因为后台服务可能还未启动成功
手机在收到微信或支付宝到账通知时，主页面是否有Toast提示文字


:speech_balloon:  3、二维码有效期是如何设置的？可否设置长一点？  
> 二维码有效期可以在后台插件管理中进行设置，二维码有效期越长，出现优惠价格和手动输入价格的机率越高，请根据自己业务量进行设置，一般建议设置为300秒。  


:speech_balloon:  4、百度OCR的ApiKey和ApiSecret配置有什么用处？  
> 用于在上传收款码时识别图片中的金额和文字。请前往百度AI开放平台申请百度OCR识别，并获取到OCR的APPID、ApiKey和ApiSecret  


:speech_balloon:  5、用户已经支付成功了，但后台状态未变更时该如何操作？  
> 如果用户已经支付成功，但是后台没有处理成功的时候，我们可以在后台个人收款->订单管理中找到相应的订单设置为我已收款
点击我已收款按钮后，系统将会把订单状态设置为已支付的状态，同时执行回调通知请求。


:speech_balloon:  6、漏单管理中的列表是什么数据？  
> 如果用户在二维码有效期外发生的支付记录将会在漏单管理中进行显示  
如果用户在二维码有效期外支付，同时刚好又有会员创建了相同金额的订单，将会导致订单下发错误，此时我们可以在订单管理中手动修正订单状态。
在漏单管理中可手动编辑，编辑时可选定关联的订单号，保存后会自动执行回调通知请求。


:speech_balloon:  7、个人收款插件配置中的识别图片方式中的本地和远程有什么区别？  
> 如果你启用了云储存插件，在上传二维码时图片是直传到云储存的，此时需要设置为远程的识别方式  

:speech_balloon:  8、支付页二维码无法显示？  
> 请确保已经在插件市场安装二维码生成插件，同时确保有启用GD库扩展，然后在插件管理中修改免签插件配置生成二维码的接口地址  

:speech_balloon:  9、为什么支付成功后成功跳转，但后台订单状态显示通知失败？  
>  首先请检查下你的FastAdmin是否开启了app_trace，如果开启了请置为false，其次检查下你的notifyurl的返回是否有其它字段，成功请只返回success这7个字符，不能再返回其它任意字符  

:speech_balloon:  10、创建订单时提示订单已支付成功,请勿重复支付该如何处理？  
>  由于你调用创建订单接口时传的out_order_id已经支付成功了，所以会有此提示，请保证out_order_id的唯一性  

### :book: 原理

>  个人微信支付宝收款插件免签收款的原理是根据金额差来识别是哪一笔订单进行支付的，因此在同一时间有同一金额进行提交支付，后面的金额都会做随机立减(递增)处理(单位为分)，当用户支付成功后此金额将会被释放，后续可继续使用，如果用户占用此金额而没有支付，金额会在订单有效期过期后释放。  