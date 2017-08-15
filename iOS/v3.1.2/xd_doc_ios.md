## <center> 心动SDK（iOS）对接文档 </center>

### 目录
* [1.心动SDK介绍](#1心动sdk介绍)
	* [1.1登录流程](#11登录流程)
	* [1.2支付流程](#12支付流程)
	* [1.3SDK集成流程](#13sdk集成流程)
* [2.申请心动AppID](#2申请心动appid)
* [3.需要遵守的规则](#3需要遵守的规则)
	* [3.1.回调依赖](#31回调依赖)
* [4.接入SDK](#4接入sdk)
	* [4.1获取SDK](#41获取sdk)
	* [4.2添加系统依赖库](#42添加系统依赖库)
	* [4.3设置URL Types](#43设置url-types)
	* [4.4配置info.plist](#44配置infoplist)
	* [4.5处理第三方应用跳回事件](#45处理第三方应用跳回事件)
* [5.接口调用](#5接口调用)
	* [5.1配置SDK登录选项](#51配置sdk登录选项)
	* [5.2实现SDK协议](#52实现sdk协议)
	* [5.3初始化SDK](#53初始化sdk)
	* [5.4登录](#54登录)
	* [5.5获取Access Token](#55获取access-token)
	* [5.6获取当前登录状态](#56获取当前登录状态)
	* [5.7打开用户中心](#57打开用户中心)
	* [5.8发起支付](#58发起支付)
	* [5.9登出](#59登出)
	* [5.10游客升级](#510游客升级)
	* [5.11微信分享](#511微信分享)
* [6服务端对接](#6服务端对接)
	* [6.1获取用户信息](#61获取用户信息)
	* [6.2处理支付回调](#62处理支付回调)

<span id="1心动sdk介绍">

### 1.心动SDK介绍

心动SDK主要为游戏提供登录，支付等功能。登录流程和支付流程都需要客户端和服务端同时参与。

<span id="11登录流程">

#### 1.1.登录流程

<image src="https://static.tapdb.net/web/res/img/upload/2017/06/27/01.png"></image>

<span id="12支付流程">

#### 1.2.支付流程

<image src="https://static.tapdb.net/web/res/img/upload/2017/06/27/02.png"></image>

<span id="13sdk集成流程">

#### 1.3.SDK集成流程

<image src="https://static.tapdb.net/web/res/img/upload/2017/07/07/01.png"></image>

<span id="2申请心动appid">

### 2.申请心动AppID

<p>参阅“心动AppID申请介绍”文档，申请心动AppID，获得心动AppID、心动AppKey、微信AppID、QQ AppID。其中心动AppID主要是客户端对接时使用，AppKey主要是服务端对接支付回调时使用。</p>

<p style="color:red">注意，如果游戏需要使用微信分享功能，必须使用心动提供的微信AppID，否则会导致微信登录失败。
</p>

<span id="3需要遵守的规则">

### 3.需要遵守的规则

<p>对接过程中，为了避免出现一些奇怪的问题，无特殊需求情况下，请尽量遵守下面的规则。
</p>

<span id="31回调依赖">

#### 3.1.回调依赖

<p>游戏调用SDK的功能，SDK通常会以回调形式通知到游戏，除了必须依赖的回调（如登录成功回调），其它回调尽可能不依赖。此场景适用于登录、打开用户中心、支付。</p>

<p>比如游戏登录，通常是游戏提供一个登录按钮，用户点击后调用SDK登录，此时有两种处理方式。A、点击后，游戏隐藏登录按钮或使用loading图标盖住，等收到SDK登录失败回调时，才会再次展现登录按钮或移除loading。B、无论是否收到回调，都保证按钮处理可点状态。</p>

<p>建议游戏使用后者，防止意外情况下，流程无法走通，用户无法继续操作的情况。另外也可以采用后者的优化版，即用户点击后，仅屏蔽按钮1秒钟，这样可以防止用户反复点击导致SDK接口被反复调用。</p>

<p>总之游戏尽可能保证功能一直处于可用状态，而不依赖SDK的状态。</p>

<span id="4接入sdk">

### 4.接入SDK

<span id="41获取sdk">

#### 4.1.获取SDK

从心动平台处获取SDK，其中主要的文件或目录用途如下。

目录或文件 | 用途
--- |---
XdComPlatform.framework | 心动SDK的主要库文件，需要添加到项目依赖中 
XDStore.framework | 心动SDK支付组建，需要添加到项目依赖库中
resource | 心动SDK需要或依赖的资源文件，需要保证所有文件都被添加到了Xcode的“Copy Bundle Resources”中
libs | 心动SDK依赖的其它库文件，需要添加到项目依赖中

<span id="42添加系统依赖库">

#### 4.2.添加系统依赖库

```
Security.framework 
CFNetwork.framework
UIKit.framework
QuartzCore.framework
Foundation.framework
CoreGraphics.Framework
CoreTelephony.framework
SystemConfiguration.framework
libiconv.tbd
libsqlite3.tbd
libstdc++.tbd
libz.tbd
libicucore.tbd
```
<span id="43设置url-types">

#### 4.3.设置URL Types

需要在Xcode中设置多个URL Types，URL Types主要是需要设置URL Schemes，其它选项可任意填写。按照下面表格的内容填写，注意替换其中的各项AppID。

URL Schemes | 用途 |示例 |备注
---|---|---|---|
XD-{心动AppID}|用于支付宝支付后跳回|XD-ci2dos1ktzsca4f
{微信AppID}| 用于微信授权登录后跳回|wx19f231d77ac408d9
tencent{QQ AppID}|用于QQ授权登录后跳回|tencent317081|如果给到的心动AppID没有对应的QQ AppID，可以不配置该项

<span id="44配置infoplist">

#### 4.4.配置info.plist

修改项目的info.plist，在<dict>节点中添加下列内容。修改的内容主要为了保证QQ和微信登录能够正常运行。

```
<key>LSApplicationQueriesSchemes</key>
<array>
<string>mqq</string>
<string>mqqapi</string>
<string>wtloginmqq2</string>
<string>mqqopensdkapiV4</string>
<string>mqqopensdkapiV3</string>
<string>mqqopensdkapiV2</string>
<string>mqqwpa</string>
<string>mqqOpensdkSSoLogin</string>
<string>mqqgamebindinggroup</string>
<string>mqqopensdkfriend</string>
<string>mqzone</string>
<string>weixin</string>
<string>wechat</string>
</array>
<key>NSAppTransportSecurity</key>
<dict>
<key>NSAllowsArbitraryLoads</key>
<true/>
</dict>

```

<span id="45处理第三方应用跳回事件">

#### 4.5.处理第三方应用跳回事件

在AppDelegate.m中增加如下两个方法，如果已经存在这些方法，在其中追加相应的处理代码即可。

```
- (BOOL)application:(UIApplication *)application
openURL:(NSURL *)url
sourceApplication:(NSString *)sourceApplication
annotation:(id)annotation {
return [XDCore HandleXDOpenURL:url];
}

- (BOOL)application:(UIApplication *)app
openURL:(NSURL *)url
options:(NSDictionary<UIApplicationOpenURLOptionsKey,id> *)options {
return [XDCore HandleXDOpenURL:url];
}
```

<span id="5接口调用">

### 5.接口调用

<span id="51配置sdk登录选项">

#### 5.1.配置SDK登录选项

<p>心动SDK提供配置QQ、微信登录、游客登录的显示与隐藏以及登录方式。</p>

<p>如不进行配置，SDK将默认显示QQ、微信、游客登录。QQ和微信的登录方式默认为App授权登录，心动SDK也会根据实际情况，将QQ和微信登录方式切换为Web登录，或者不提供对应的登录功能。</p>

<p style="color:red">请勿直接复制以下代码，根据游戏实际需求选择调用以下API</p>

```
[XDCore hideGuest]; //隐藏游客登录
[XDCore hideWX]; //隐藏微信登录
[XDCore hideQQ]; //隐藏QQ登录
[XDCore showVC];//显示VeryCD登录（此接口供老游戏兼容，新游戏不建议调用）
[XDCOre setQQWeb]; //设置QQ为Web登录方式
[XDCore setWXWeb];//设置微信为Web 扫码登录方式

```

<span id="52实现sdk协议">

#### 5.2.实现SDK协议

游戏调用SDK接口后，结果以Protocol方式回调给游戏，游戏需要实现XDCallback.h中声明的方法。
<br/>
代码示例：

```

[XDCore setCallBack:[YourCallBack new]];

#import "YourCallBack.h"
@interface YourCallBack()<XDCallback>
@end

@implementation YourCallBack

/*初始化成功*/
- (void)onInitSucceed{
}

/*初始化失败*/
- (void)onInitFailed:(nullable NSString*)error_msg{
}

/*登录成功*/
/*access_token:登录成功，获取到用户的token。游戏根据token获取用户信息。*/
- (void)onLoginSucceed:(nonnull NSString*)access_token{
}

/*登录失败*/
/*error_msg: 登录失败的报错信息*/
- (void)onLoginFailed:(nullable NSString*)error_msg{
}

/*登录取消*/
- (void)onLoginCanceled{
}

/*游客绑定成功*/
/*access_token: 游客升级成功，获取新的token。
  后续如需使用 token，务必使用回调给的新 token。但已生效的会话无须处理。
  */
- (void)onBindGuestSucceed:(nonull NSString*)access_token{
}

/*登出成功*/
- (void)onLogoutSucceed{
}

/*支付完成*/
- (void)onPayCompleted{
}

/*支付失败*/
/*error_msg:支付失败的错误信息*/
- (void)onPayFailed:(nullable NSString*)error_msg{
}

/*支付取消*/
- (void)onPayCanceled{
}

```

<span id="53初始化sdk">

#### 5.3.初始化SDK

初始化心动SDK，调用该接口是调用其它功能接口的必要条件。

```
/**
* @param app_id 心动AppID
* @param aOrientation 屏幕方向，0表示横屏，1表示竖屏
*/
+ (void)init:(NSString *)app_id orientation:(int)aOrientation;

```

<p>调用该接口会触发下列回调。</p>
<p style="color:red">其他接口请在获取到初始化成功回调之后进行调用。</p>

类别 | 回调方法
--- |---
初始化成功 | -(void)onInitSucceed
初始化失败 | -(void)onInitFailed:(nullable NSString*)error_msg

<span id="54登录">

#### 5.4.登录

调用该接口进行登录。

```
/*登录*/
+ (void)login;
```

<p>调用该接口会触发下列回调。</p>
<p style="color:red">获取、查看用户信息以及支付接口请在获取到登录成功回调之后调用。</p>

类别 | 回调方法
--- | ---
登录成功 | - (void)onLoginSucceed:(nonnull NSString*)access_token;
登录失败 | - (void)onLoginFailed:(nullable NSString*)error_msg;
登录取消 | - (void)onLoginCanceled;

<span id="55获取access-token">

#### 5.5.获取Access Token

调用该接口获取当前登录用户的access token。

```
/**
* @return 当前登录用户的access token，无登录用户时，返回空字符串
*/
+ (nullable NSString *)getAccessToken;
```

<span id="56获取当前登录状态">

#### 5.6.获取当前登录状态

调用该接口获取当前登录状态。

```
/**
* @return YES表示已经登录，NO表示未登录
*/
+ (BOOL)isLoggedIn;

```

<span id="57打开用户中心">

#### 5.7.打开用户中心

调用该接口打开用户中心界面，用户可以在该界面进行登出和登录操作，游戏注意正确处理回调。在未登录状态，无法打开用户中心。在用户中心中，用户可进行登出操作，此时交互界面将消失。游戏需要提供引导用户重新进行登录的操作界面。

```
/*
* @return YES,开启成功 NO，开启失败
*/
+ (BOOL)openUserCenter;
```

<span id="58发起支付">

#### 5.8.发起支付
调用该接口发起支付。当前版本SDK仅支持AppStore支付。

```
/**
* @param info 支付相关信息，注意key和value都是字符串类型
*/
+ (void)pay:(nonull NSDictionary*)info

```
其中info的字段如下。

参数 | 必须 |说明
--- | --- |--- 
Product_Name | 是 |商品名称，建议以游戏名称开头，方便财务对账
Product_Id | 是 | 商品ID，到AppStore购买的商品
Product_Price | 是 | 商品价格（单位分），对于AppStore支付，该字段没有用处，但是需要传递真实金额，有多处需要用到
Sid | 是 |所在服务器ID，不能有特殊字符，服务端支付回调会包含该字段
Role_Id | 否 | 支付角色ID，服务端支付回调会包含该字段
OrderId | 否 | 游戏侧订单号，服务端支付回调会包含该字段
EXT | 否 |额外信息，最长512个字符，服务端支付回调会包含该字段。

调用该接口会触发下列回调。

类别 | 回调方法
--- | ---
支付完成 | - (void)onPayCompleted;
支付失败 | - (void)onPayFailed:(nullable NSString*)error_msg;
支付取消 | - (void)onPayCanceled;

<span id="59登出">

#### 5.9.登出
需要注销当前登录用户时调用，该操作不会出现登录界面。

```
+ (void)logout;

```
调用该接口会触发下列回调

类别 | 回调方法
--- | ---
登出成功 | - (void)onLogoutSucceed;

<span id="510游客升级">

#### 5.10.游客升级

当游客账号升级成功时,会触发下列回调。<br/>
后续如需使用token，务必使用回调给的新token。但已生效的会话无需处理。

类别 | 回调方法
--- | ---
游客升级成功 | - (void)onGuestBindSucceed:(nonull NSString*)access_token;

<span id="511微信分享">

#### 5.11.微信分享

调用XDWXShare文件中的以下方法，使用微信分享功能。

```
+ (void)WXShareWithObject:(XDWXShareObject*)shareObject;

```

XDWXShareObject详细参数说明

通用参数 | 是否必须 | 类型| 说明
--- | --- | --- | ---
text | 否 | NSString | 分享文字内容
bText | 是 | BOOL | 分享内容是否为多媒体内容（YES文本内容，NO多媒体内容）
scene | 是 | int | 分享场景（0对话，1朋友圈，2收藏）
type | 是 | int | 分享类型（0文本，1图片，2音乐，3视频，4网页）
title | 否 | NSString | 分享标题
descriptionStr | 否 | NSString | 分享描述信息
thumbData | 否 | NSData | 缩略图数据
thumbPath | 否 | NSString | 缩略图路径
thumbImage | 否 | UIImage | 缩略图对象（优先级thumbImage>thumbPath>thumbData,若都未进行设置，则不显示缩略图）
meidaTagName | 否 | NSString |
messageExt | 否 | NSString |
messageAction | 否 | NSString |

以下参数，请根据不同的分享类型（type参数）进行设置

图片分享 | 是否必须 | 类型| 说明
--- | --- | --- | ---
imageData | 否| NSData | 分享图片数据
imageUrl | 否| NSString | 分享图片Url（imageData、imageUrl不可同时为空）
<br/>


音乐分享 | 是否必须 | 类型| 说明
--- | --- | --- | ---
musicUrl | 是 | NSString | 音乐链接
musicLowBandUrl | 否 | NSString | 低带宽音乐链接
musicDataUrl | 否 | NSString | 音乐数据链接
musicLowBandDataUrl | 否 | NSString | 低带宽状音乐数据链接
<br/>

视频分享 | 是否必须 | 类型| 说明
--- | --- | --- | ---
videoUrl | 是 | NSString | 视频链接
videoLowBandUrl | 否 | NSString | 低带宽视频链接
<br/>


网页分享参数 | 是否必须 | 类型| 说明
--- | --- | --- | ---
webpageUrl | 是 | NSString | 网页链接


API调用示例
在分享之前，需要设置分享回调

```
#import <XdComPlatform/XDWXShare.h>

@interface YourCallBack()<XDWXShareCallBack>

[XDWXShare setWXShareCallBack:YourCallBack];

XDWXShareObject * obj = [XDWXShareObject shareObject];

obj.text = "text";
obj.title = "title";
...
...

[XDWXShare WXShareWithObject:obj];

```

调用该接口会触发下列回调。

类别 | 回调方法
--- | ---
分享成功 | - (void)onWXShareSucceed;
分享失败 | - (void)onWXShareFailed:(nullable NSString*)error_msg;

<p style = "color:red">
注意: 在进行分享时，若用户选择留在微信，则无法收到分享回调，此处不可以完全依赖上述回调事件。
</p>

<span id="61服务端对接">

### 6.服务端对接

<span id="61获取用户信息">

#### 6.1.获取用户信息
游戏服务端使用客户端获取的access token，按照下面的方式获取用户信息。

```
接口：https://api.xd.com/v1/user
method：GET
参数：access_token
请求示例：https://api.xd.com/v1/user?access_token=1234
成功判断：返回的HTTP Code为200时表示成功，否则失败
返回数据格式：application/json
返回值示例：
{"id":"1""name":"xdname","friendly_name":"xdfriendly_name","client_id":"abc","phone":"1","safety":false,"site":"1"}
id：用户的ID，注意类型是字符串
name：用户的账号名称
friendly_name：用户的昵称，如果游戏想要展现用户名称，建议使用该字段
client_id：该用户在该游戏登录时使用的心动AppID
phone： 绑定的手机号码
safety： 账号是否安全/通过设备二次验证（true：安全，false：不安全）
site：账号类型, 0 => vc账号，1 => 心动账号，3 => qq账号，8 => 微信账号，注意类型是字符串
```
<span id="62处理支付回调">

#### 6.2.处理支付回调
游戏服务端需要提供一个能够处理支付回调的接口，这个接口是申请心动AppID时需要的。处理逻辑中，需要使用一个密钥进行加密验证，该密钥即为心动AppKey。
当心动平台处有充值成功时，心动服务端会通知到支付回调接口，信息如下。

```
method：POST
数据格式：application/x-www-form-urlencoded

```
字段如下。


字段 | 类型 | 描述
--- | --- | ---
order_id | number | 心动平台的订单号，相同订单号表示是同一笔支付
payment | string | 支付方式，appstore或其它（若回调无该字段，则默认为appstore）
user_id | string | 充值用户ID，注意类型是字符串
client_id | string | 充值的心动AppID
app | string | 同client_id
app_id | string | 游戏客户端调用充值时传递的Sid字段
app\_order_id | string | 游戏客户端调用充值时传递的OrderId字段
role_id | string | 游戏客户端调用充值时传递的Role_Id字段
product_id | string | 支付购买的商品ID
gold | number | 支付实际所付金额，单位元。（仅在客户端使用非AppStore支付方式支付时才有该字段）
ext | string | 游戏客户端调用充值时传递的EXT字段
timestamp | number | 时间戳，1990年到当前时间的秒数
sign | string | 签名校验字段，按照下面的方式进行校验

签名算法示例，使用php语言。

```
/**
* @param params 类型array，支付回调时收到的参数
* @param appKey 类型string，心动AppKey
*/
function verify_sign($params, $appKey) {
$tmp = $params;
$sign = $tmp['sign'];
unset($tmp['sign']);
ksort($tmp);

return strcasecmp($sign, md5(http_build_query($tmp) . $appKey)) == 0;
}
```

<p style="color:red">
需要注意
<br/>
1、游戏服务端应该按照order_id进行排重，相同order_id仅生效一次。
<br/>
2、游戏服务端成功处理了支付回调后，应当返回字符串“success”，如果是一笔已经处理的重复的订单，也应该返回“success”。
<br/>
3、只要通过签名校验的回调，都应该视为合法数据，按照如下逻辑发放道具。A.如果payment字段为appstore，即AppStore支付，直接按照product_id字段进行道具发放；B.如果payment字段为其它值，需要验证gold字段和 product_id 字段是否相符，如果相符，按照product_id发放道具，如果不相符，直接按照gold字段折算成对应的游戏货币发放。
</p>
