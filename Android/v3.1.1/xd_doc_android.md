## <center> 心动SDK（Android）对接文档 </center>

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
	* [4.2添加Android支持库](#42添加android支持库)
	* [4.3添加需要的权限](#43添加需要的权限)
	* [4.4声明相关的Activity](#44声明相关的activity)
	* [4.5接收返回值](#45接收返回值)
	* [4.6接入微信分享处理](#46接入微信分享处理)
* [5.接口调用](#5接口调用)
	* [5.1配置SDK登录选项](#51配置sdk登录选项)
	* [5.2设置回调](#52设置回调)
	* [5.3初始化SDK](#53初始化sdk)
	* [5.4登录](#54登录)
	* [5.5获取Access Token](#55获取access-token)
	* [5.6获取当前登录状态](#56获取当前登录状态)
	* [5.7打开用户中心](#57打开用户中心)
	* [5.8发起支付](#58发起支付)
	* [5.9登出](#59登出)
	* [5.10游客升级](#510游客升级)
* [6服务端对接](#6服务端对接)
	* [6.1获取用户信息](#61获取用户信息)
	* [6.2处理支付回调](#62处理支付回调)

### 1.心动SDK介绍

心动SDK主要为游戏提供登录，支付等功能。登录流程和支付流程都需要客户端和服务端同时参与。

#### 1.1.登录流程


<image src="img/01.png"></image>


#### 1.2.支付流程

<image src="img/02.png"></image>

#### 1.3.SDK集成流程

<image src="img/workflow.png"></image>


### 2.申请心动AppID

<p>参阅“心动AppID申请介绍”文档，申请心动AppID，获得心动AppID、心动AppKey、微信AppID、QQ AppID、Ping++ ID。其中心动AppID主要是客户端对接时使用，AppKey主要是服务端对接支付回调时使用。</p>

<p style="color:red">注意，如果游戏需要使用微信分享功能，必须使用心动提供的微信AppID，否则会导致微信登录失败。
</p>

### 3.需要遵守的规则

<p>对接过程中，为了避免出现一些奇怪的问题，无特殊需求情况下，请尽量遵守下面的规则。
</p>

#### 3.1.回调依赖

<p>游戏调用SDK的功能，SDK通常会以回调形式通知到游戏，除了必须依赖的回调（如登录成功回调），其它回调尽可能不依赖。此场景适用于登录、打开用户中心、支付。</p>

<p>比如游戏登录，通常是游戏提供一个登录按钮，用户点击后调用SDK登录，此时有两种处理方式。A、点击后，游戏隐藏登录按钮或使用loading图标盖住，等收到SDK登录失败回调时，才会再次展现登录按钮或移除loading。B、无论是否收到回调，都保证按钮处理可点状态。</p>

<p>建议游戏使用后者，防止意外情况下，流程无法走通，用户无法继续操作的情况。另外也可以采用后者的优化版，即用户点击后，仅屏蔽按钮1秒钟，这样可以防止用户反复点击导致SDK接口被反复调用。</p>

<p>总之游戏尽可能保证功能一直处于可用状态，而不依赖SDK的状态。</p>

### 4.接入SDK

#### 4.1.获取SDK

<p>从心动平台处获取SDK，其中SDKLib目录就是心动SDK项目，可以直接导入eclipse中作为Library项目。如果不是使用eclipse作为IDE的项目，需要保证相关的文件都被正确导入到项目中。SDKLib中的主要文件或目录用途如下。</p>

目录 | 用途
--- |---
libs | 包含心动SDK的库和其它依赖库
res | 包含心动SDK需要的资源文件

#### 4.2.添加Android支持库

<p>添加Android v4支持库android-support-v4.jar到项目中。</p>

#### 4.3.添加需要的权限

```
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
    <uses-permission android:name="android.permission.ACCESS_WIFI_STATE" />
    <uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
    <uses-permission android:name="android.permission.READ_PHONE_STATE" />
    <!-- Ping++ -->
    <uses-permission android:name="android.permission.NFC" />
    <uses-permission-sdk-23 android:name="android.permission.READ_PHONE_STATE" />
    <uses-permission-sdk-23 android:name="android.permission.NFC" />
```


#### 4.4.声明相关的Activity

```
<application>
    <activity
        android:name="com.xd.sdklib.helper.XDStartView"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
    <activity
        android:name="com.xd.sdklib.helper.XDPayActivity"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
    <activity
        android:name="com.xd.sdklib.helper.XDWebView"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />
    <activity
        android:name="com.xd.sdklib.helper.WXEntryActivity"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />

    <!-- 微信登录，仅在游戏不接入其它微信分享SDK时使用该方法 -->
    <activity-alias
        android:name=".wxapi.WXEntryActivity"
        android:exported="true"
        android:targetActivity="com.xd.sdklib.helper.WXEntryActivity"/>

    <!-- 微信登录和分享，游戏接入了其它微信分享SDK时必须使用该方法，并按照文章后面的内容进行处理
    <activity
        android:name=".wxapi.WXEntryActivity"
        android:label="@string/app_name"
        android:exported="true"/>
    -->

    <!-- Ping++ SDK -->
    <activity
        android:name="com.pingplusplus.android.PaymentActivity"
        android:configChanges="orientation|screenSize"
        android:launchMode="singleTop"
        android:theme="@android:style/Theme.Translucent.NoTitleBar.Fullscreen" />

    <!-- 银联支付 sdk -->
    <activity
        android:name="com.unionpay.uppay.PayActivity"
        android:configChanges="orientation|keyboardHidden|navigation|screenSize" />

    <!-- 支付宝 -->
    <activity
        android:name="com.alipay.sdk.app.H5PayActivity"
        android:configChanges="orientation|keyboardHidden|navigation"
        android:exported="false"
        android:screenOrientation="portrait" />
    <activity
        android:name="com.alipay.sdk.auth.AuthActivity"
        android:configChanges="orientation|keyboardHidden|navigation"
        android:exported="false"
        android:screenOrientation="portrait" />

    <!-- 微信支付 -->
    <activity-alias
        android:name=".wxapi.WXPayEntryActivity"
        android:exported="true"
        android:targetActivity="com.pingplusplus.android.PaymentActivity" />

    <!-- QQ登录 -->
    <activity
        android:name="com.tencent.tauth.AuthActivity"
        android:noHistory="true"
        android:launchMode="singleTask" />
    <activity
        android:name="com.tencent.connect.common.AssistActivity"
        android:theme="@android:style/Theme.Translucent.NoTitleBar"
        android:configChanges="orientation|keyboardHidden|screenSize" />
</application>


```

#### 4.5.接收返回值

在初始化XDSDK的Activity中，增加如下处理。

```
@Override
protected void onActivityResult(int requestCode, int resultCode, Intent data) {
    XDSDK.onActivityResultData(requestCode, resultCode, data);
    super.onActivityResult(requestCode, resultCode, data);
}
```

#### 4.6.接入微信分享处理

<p>心动SDK目前仅提供微信登录功能，如果游戏需要使用微信分享功能，需要自行接入微信分享功能。需要注意下面几点。</p>
<p style="color:red">微信分享的微信AppID必须使用心动提供的微信AppID，否则会导致微信登录失败。</p>
<p>接入其它SDK提供的微信分享功能时，会被要求在项目中增加一个类“{游戏包名}.wxapi.WXEntryActivity”，这个类可能是复制SDK提供的一个类，或者继承SDK的一个类。无论如何，将其修改为另一个名字，比如“{游戏包名}.wxapi.MYWXEntryActivity”。</p>
<p>需要按照文章前面Activity配置中的注释说明，修改Activity配置内容，并且需要在游戏项目中增加“{游戏包名}.wxapi.WXEntryActivity”类，内容如下，注意替换代码中花括号括起来的需要游戏自定的内容。</p>

```
package {游戏的包名(AndroidManifest.xml中的package)}.wxapi;

import android.app.Activity;
import android.content.Intent;
import android.os.Bundle;
import android.util.Log;

import com.tencent.mm.sdk.modelbase.BaseReq;
import com.tencent.mm.sdk.modelbase.BaseResp;
import com.tencent.mm.sdk.modelmsg.SendAuth;
import com.tencent.mm.sdk.modelmsg.SendMessageToWX;
import com.tencent.mm.sdk.openapi.IWXAPI;
import com.tencent.mm.sdk.openapi.IWXAPIEventHandler;
import com.tencent.mm.sdk.openapi.WXAPIFactory;
import com.xd.xdsdk.XDCore;

public class WXEntryActivity extends Activity implements IWXAPIEventHandler {

    private IWXAPI api;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        this.api = WXAPIFactory.createWXAPI(this, XDCore.getInstance().getWXAppId(),false);
        this.api.handleIntent(getIntent(), this);
    }
    
    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        
        setIntent(intent);
        api.handleIntent(intent, this);
    }
    
    @Override
    public void onReq(BaseReq arg0) {
    }

    @Override
    public void onResp(BaseResp arg0) {
        if (arg0 instanceof SendMessageToWX.Resp) {
            Intent intent = new Intent();
            String str = getPackageName() + ".wxapi.{被改名的分享类名}" // 替换后的例子"wxapi.MYWXEntryActivity"
            intent.setClassName(getPackageName(), str);
            if (intent.resolveActivity(getPackageManager()) != null) {
                intent.putExtras(getIntent());
                startActivity(intent);
            }
            finish();
            return;
        }

        if (arg0 instanceof SendAuth.Resp) {
            switch (arg0.errCode) {
            case BaseResp.ErrCode.ERR_OK:
                String code = ((SendAuth.Resp)arg0).code;
                Log.d("微信code",code);
                //调用平台登录
                XDCore.getInstance().wxLogin(code);
                break;
            case BaseResp.ErrCode.ERR_USER_CANCEL:
                Log.d("微信登录","ERR_USER_CANCEL");
                break;
            case BaseResp.ErrCode.ERR_AUTH_DENIED:
                Log.d("微信登录","ERR_AUTH_DENIED");
                break;
            default:
                break;
            }
        }
        finish();
    }
}
```

### 5.接口调用

<p>所有接口设计为静态方法，在com.xd.xdsdk.XDSDK中调用。</p>

#### 5.1.配置SDK登录选项

<p>心动SDK提供配置QQ、微信登录、游客登录的显示与隐藏以及登录方式。</p>

<p>如不进行配置，SDK将默认显示QQ、微信、游客登录。QQ和微信的登录方式默认为App授权登录，心动SDK会根据QQ和微信的安装情况，将QQ和微信登录方式切换为Web登录，或者不提供对应的登录功能。</p>

<p style="color:red">请勿直接复制以下代码，根据游戏实际需求选择调用以下API</p>

```
XDSDK.hideGuest()	//隐藏游客登录
XDSDK.hideWX()		//隐藏微信登录
XDSDK.hideQQ()		//隐藏QQ登录
XDSDK.showVC()		//显示VeryCD登录（此接口供老游戏兼容，新游戏不建议调用）
XDSDK.setQQWeb()	//设置QQ为Web登录方式
XDSDK.setWXWeb()	//设置微信为Web 扫码登录方式
```

#### 5.2.设置回调

<p>游戏调用心动SDK的接口后，大部分情况都是通过回调的形式将结果返回给游戏，所以需要先设置对应的回调函数。</p>

代码示例：

```
XDSDK.setCallback(new XDCallback() {
            //初始化成功
            @Override
            public void onInitSucceed() {

            }
            //初始化失败
            @Override
            public void onInitFailed(String msg) {

            }
            //登录成功
            @Override
            public void onLoginSucceed(String token) {
                
            }

            //登录失败
            @Override
            public void onLoginFailed(String msg) {
				
            }

            //登录取消
            @Override
            public void onLoginCanceled() {

            }

            //游客绑定成功
            @Override
            public void onGuestBindSucceed(String token) {

            }
            //登出成功
            @Override
            public void onLogoutSucceed() {

            }
            //支付完成
            @Override
            public void onPayCompleted() {

            }
            //支付失败
            @Override
            public void onPayFailed(String msg) {

            }
            //支付取消
            @Override
            public void onPayCanceled() {

            }
        });
```

#### 5.3.初始化SDK

初始化心动SDK，调用该接口是调用其它功能接口的必要条件。

```
/**
 * @param activity 游戏的主activity
 * @param appid 心动AppID
 * @param aOrientation 屏幕方向，0表示横屏，1表示竖屏
 */
public static void initSDK(Activity activity, String appid, int aOrientation)
```

示例代码
```
XDSDK.initSDK(this, "xxxxxxxxxxxxxx", 0);
```
<p>调用该接口会触发下列回调。</p>
<p style="color:red">其他接口请在获取到初始化成功回调之后进行调用。</p>

类别 | 回调方法
--- |---
初始化成功 | public void onInitSucceed()
初始化失败 | public void onInitFailed(String msg)

#### 5.4.登录

调用该接口进行登录。

```
public static void login()
```

示例代码
```
XDSDK.login();
```

<p>调用该接口会触发下列回调。</p>
<p style="color:red">获取、查看用户信息以及支付接口请在获取到登录成功回调之后调用。</p>

类别 | 回调方法
--- | ---
登录成功 | public void onLoginSucceed(String token)
登录失败 | public void onLoginFailed(String msg)
登录取消 | public void onLoginCanceled()

#### 5.5.获取Access Token

调用该接口获取当前登录用户的access token。

```
public static String getAccessToken()
```

代码示例
```
XDSDK.getAccessToken()
```

#### 5.6.获取当前登录状态

调用该接口获取当前登录状态。

```
public boolean isLoggedIn()
```
代码示例
```
XDSDK.isLoggedIn()
```

#### 5.7.打开用户中心

调用该接口打开用户中心界面，用户可以在该界面进行登出和登录操作，游戏注意正确处理回调。在未登录状态，无法打开用户中心。在用户中心中，用户可进行登出操作，此时交互界面将消失。游戏需要提供引导用户重新进行登录的操作界面。

```
/**
* @return false表示尚未登录，重复调用默认为成功
*/
public static boolean openUserCenter()
```
代码示例
```
XDSDK.openUserCenter()
```

#### 5.8.发起支付
调用该接口发起支付。
<p style='color:red'>不保证在任何情况下都能收到回调，请勿直接使用SDK返回的支付结果作为最终判定订单状态的依据。</p>

```
/**
* @param info 支付相关信息，注意key和value都是字符串类型
*/
public static boolean pay(Map<String, String> info) 
```
其中info的字段如下。

参数 | 必须 |说明
--- | --- |--- 
Product_Name | 是 |商品名称，建议以游戏名称开头，方便财务对账
Product_Id | 是 | 商品ID
Product_Price | 是 | 商品价格（单位分）
Sid | 是 |所在服务器ID，不能有特殊字符，服务端支付回调会包含该字段
Role_Id | 否 | 支付角色ID，服务端支付回调会包含该字段
OrderId | 否 | 游戏侧订单号，服务端支付回调会包含该字段
EXT | 否 |额外信息，最长512个字符，服务端支付回调会包含该字段。

调用该接口会触发下列回调。

类别 | 回调方法
--- | ---
支付完成 | public void onPayCompleted()
支付失败 | public void onPayFailed(String msg) 
支付取消 | public void onPayCanceled()

示例代码
```
Map<String, String> info = new HashMap<String, String>();
info.put("OrderId", "1234567890123456789012345678901234567890");
info.put("Product_Price", "1");
info.put("EXT", "abcd|efgh|1234|5678");
info.put("Sid", "2");
info.put("Role_Id", "3");
info.put("Product_Id", "4");
info.put("Product_Name", "648大礼包");
XDSDK.pay(info);
```

#### 5.9.登出
需要注销当前登录用户时调用，该操作不会出现登录界面。

```
public static void logout() 
```
调用该接口会触发下列回调

类别 | 回调方法
--- | ---
登出成功 | public void onLogoutSucceed() 

示例代码
```
XDSDK.logout();
```

#### 5.10.游客升级

当游客账号升级成功时,会触发下列回调。<br/>
后续如需使用token，务必使用回调给的新token。但已生效的会话无需处理。

类别 | 回调方法
--- | ---
游客升级成功 | public void onGuestBindSucceed(String token)

#### 5.11.退出

调用该方法时，弹出确认框供用户选择是否退出。

```
public static void exit(ExitCallback callback)
```

示例代码

```
XDSDK.exit(new ExitCallback() {
    @Override
    public void onConfirm() {
    	super.onConfirm();
    }

    @Override
    public void onCancle() {
    	super.onCancle();
    }
});
```

### 6.服务端对接

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
{"id":”a13”,"name":"xdname","friendly_name":"xdfriendly_name","client_id":"abc"}
id：用户的ID，注意类型是字符串
name：用户的账号名称
friendly_name：用户的昵称，如果游戏想要展现用户名称，建议使用该字段
client_id：该用户首次在该游戏登录时使用的心动AppID
```         
#### 6.2.	处理支付回调
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





