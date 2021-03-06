---
title: wx
tags: 
    wx
date: 2019-04-25 10:04:49
---

# 公众号类型功能介绍
[微信官方介绍](http://kf.qq.com/faq/170815aUZjeQ170815mU7bI7.html)

公众号，是订阅号和服务号的统称。订阅号是公众号、服务号也是公众号。

订阅号：偏于为用户传达资讯，每天只可以发送一条群发，用户不会收到即时消息提醒，因为所有的订阅号都被收在了订阅号文件夹中。订阅号适用于个人和组织。
想看一个订阅号的消息，有两种方法：
- 打开订阅号文件夹查看订阅号有没有更新消息；
- 打开微信手机通讯录，点开公众号选项，是你关注的所有公众号(包括订阅号和服务号)。

服务号：偏于服务交互(如：招商银行信用卡、微信读书、京东白条)，每月可群发四条消息，收费，不适用于个人，微信支付需要**微信认证服务号**才可以。

选择：
想简单的发送消息，达到宣传效果，建议选择订阅号。
想用公众号获得更多功能，如开通微信支付，建议选择服务号。

服务号、订阅号功能对比：
![](wx/1.jpg)

# 微信开放平台、公众平台区别
- 公众平台
[官网](https://mp.weixin.qq.com/)
[技术文档](https://mp.weixin.qq.com/wiki)
微信公众平台是运营者通过公众号为微信用户提供资讯和服务的平台，登录公众平台账号后，可以看到它有一个不错的交互界面。可以提供给公司的运营人员使用，用来发布消息和提供服务。

- 开放平台
[官网](https://open.weixin.qq.com/)
是为开发者（程序员）提供的一个平台，在这里你可以将你的公众平台下的公众号（订阅号、服务号）绑定到你的开放平台账号下，从而可以基于订阅号、服务号做更多的开发。公众号中的订阅号接口权限是有限的，比如它无法获得网页授权的权限，也就无法通过网页授权获取用户的基本信息（比如openID、unionID等）。

# 微信h5分享
[参考地址](https://blog.csdn.net/change_on/article/details/75264843)

流程图：
![](wx/2.png)


准备：公众号、已经备案好的域名

步骤：
1. 配置JS回调域名和ip白名单
2. 获取appId和secret
3. 从官方代码copy签名函数
4. 获取access_token、ticket
5. 获取url，并进行签名
6. 集成进java web
7. 前端config函数配置

## 获取appId和secret
可以在基本配置中找到AppID和AppSecret。

配置JS回调域名：
登录微信公众平台->公众号设置->功能设置->JS接口安全域名。填写你要分享的网页所在的域名。

配置ip白名单：
登录微信公众平台->基本配置->IP白名单。
必须配置ip白名单，否则无法获取access_token

## 从官方代码copy签名函数
打开[微信分享官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)，在页面最下面找到并下载[示例代码](http://demo.open.weixin.qq.com/jssdk/sample.zip)
我们将java中的`sign.java`直接放到项目中备用。

## 获取access_token、ticket
```java
public static WxShare getWxEntity(String url,StringRedisTemplate template) {
        String access_token = template.opsForValue().get("wx_base_access_token");
        String ticket = template.opsForValue().get("wx_jsapi_ticket");
        if(StringUtils.isEmpty(access_token)){
            System.out.println(">>>>>>>>>>>>>>>>>>>>>>重新获取access_token和ticket>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>");
            //已经超时，重新获取
            access_token = getAccessToken(template);
            //如果重新获取了access_token，ticket也要重新获取
            ticket = getTicket(access_token,template);
        }
        Map<String,String> resultMap = Sign.sign(ticket, url);
        WxShare wx = new WxShare();
        wx.setTicket(resultMap.get("jsapi_ticket"));
        wx.setSignature(resultMap.get("signature"));
        wx.setNoncestr(resultMap.get("nonceStr"));
        wx.setTimestamp(resultMap.get("timestamp"));
        return wx;
    }

    //获取token
    private static String getAccessToken(StringRedisTemplate template) {
        String access_token = "";
        //获取access_token填写client_credential
        String grant_type = "client_credential";
        //第三方用户唯一凭证
        String AppId= PayConstant.WX_H5_APPID;
        //第三方用户唯一凭证密钥，即appsecret
        String secret=PayConstant.WX_H5_APPSECRET;
        //访问链接,这个url链接地址和参数皆不能变
        String url = "https://api.weixin.qq.com/cgi-bin/token?grant_type="+grant_type+"&appid="+AppId+"&secret="+secret;
        try {
            URL urlGet = new URL(url);
            HttpURLConnection http = (HttpURLConnection) urlGet.openConnection();
            http.setRequestMethod("GET"); // 必须是get方式请求
            http.setRequestProperty("Content-Type","application/x-www-form-urlencoded");
            http.setDoOutput(true);
            http.setDoInput(true);
            /*System.setProperty("sun.net.client.defaultConnectTimeout", "30000");// 连接超时30秒
            System.setProperty("sun.net.client.defaultReadTimeout", "30000"); // 读取超时30秒 */
            http.connect();
            InputStream is = http.getInputStream();
            int size = is.available();
            byte[] jsonBytes = new byte[size];
            is.read(jsonBytes);
            String message = new String(jsonBytes, "UTF-8");
            JSONObject demoJson = JSONObject.fromObject(message);
            access_token = demoJson.getString("access_token");
            //缓存到redis，一定要缓存，每天只能请求2000次
            template.opsForValue().set("wx_base_access_token",access_token,2, TimeUnit.HOURS);
            is.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return access_token;
    }

    //获取ticket
    private static String getTicket(String access_token,StringRedisTemplate template) {
        String ticket = null;
        //这个url链接和参数不能变
        String url = "https://api.weixin.qq.com/cgi-bin/ticket/getticket?access_token="+ access_token +"&type=jsapi";
        try {
            URL urlGet = new URL(url);
            HttpURLConnection http = (HttpURLConnection) urlGet.openConnection();
            http.setRequestMethod("GET"); // 必须是get方式请求
            http.setRequestProperty("Content-Type","application/x-www-form-urlencoded");
            http.setDoOutput(true);
            http.setDoInput(true);
            System.setProperty("sun.net.client.defaultConnectTimeout", "30000");// 连接超时30秒
            System.setProperty("sun.net.client.defaultReadTimeout", "30000"); // 读取超时30秒
            http.connect();
            InputStream is = http.getInputStream();
            int size = is.available();
            byte[] jsonBytes = new byte[size];
            is.read(jsonBytes);
            String message = new String(jsonBytes, "UTF-8");
            JSONObject demoJson = JSONObject.fromObject(message);
            ticket = demoJson.getString("ticket");
            //缓存到redis，一定要缓存，每天只能请求2000次
            template.opsForValue().set("wx_jsapi_ticket",ticket,2, TimeUnit.HOURS);
            is.close();
        } catch (Exception e) {
            e.printStackTrace();
        }
        return ticket;
    }
```

## 获取url，并进行签名
url的获取要特别注意，因为微信在分享时会在原来的url上加上一些&from=singlemessage、&from=…的，
所以url必须要动态获取，网上有些用ajax，在网页通过“location.href.split(‘#’)“ 获取url，在用ajax传给后台，
这种方法可行，但是不推荐，一方面用ajax返回，就要访问分享的逻辑，这样后台分享的逻辑增加复杂度，带来不便，
是代码不易于维护，可读性低！另一方面分享是返回页面，而ajax是返回json，又增加了复杂度。所以，
一个java程序员是不会通过ajax从前台获取url的，这里我们用HttpRequest的方法即可，不管微信加多少后缀，都可以获取到完整的当前url。

获取url的controller如下：
```
@PostMapping("/share")
public ResponseUtil getQuestionList(@RequestBody Map<String,String> paramMap){
	LOGGER.info(
	"\n***************************************" + "\n" +
			"start share" + "\n" +
			"分享" + "\n" +
			"\n********************************************"
	);
	ResponseUtil response = ResponseUtil.success();
	CodeEnum code = CodeEnum.FAIL;
	try {
		LOGGER.info("前端传过来的>>>>>>>>>>>>>>>>>>>>>>>>>><<<<<<<<<<<<<<<<<<<<<<<<<<<：" + paramMap.get("strUrl"));
		LOGGER.info("URI解码>>>>>>>>>>>>>>>>>>>>>>>>>><<<<<<<<<<<<<<<<<<<<<<<<<<<：" + URI.create(paramMap.get("strUrl")).getPath());
		response.setData(wxUtil.getWxEntity(URI.create(paramMap.get("strUrl")).getPath(),redisTemplate));
	} catch (Exception e) {
		e.printStackTrace();
		response.setCode(code);
		response.setMessage(e.getMessage());
	}
	return response;
}
```
需要注意：这个url是前端动态获取的，不能写死。如果带参数，或是中文，前端要对url做`encodeURIComponent`转码，后台再使用`URI.create(paramMap.get("strUrl")).getPath()`进行解码。

我们再新建一个存放微信信息的实体类：wx.java
```java
public class wx {

    private String access_token;
    private String ticket ;
    private String noncestr;
    private String timestamp;
    private String str;
    private String signature;

   ...setter and getter...
}
```

sign类里面有一个签名的方法`sign`，传入ticket和url即可。也就是WeinXinUtil的getWinXinEntity方法，并将返回的map的信息读取存入WinXinEntity中。
在调试时，把sign返回的map打印出来，主要看看生成的signature。然后，把jsapi_ticket、noncestr、timestamp、url 复制到微信提供的[微信 JS 接口签名校验工具](https://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign)，
比较代码签名生成的signature与校验工具生成的签名signature是否一致，如果一致，说明前面的步骤都是正确的，如果不一致，仔细检查。

## controller
我们把微信分享分解成3个工具类，现在在处理分享的controller，只要两句话就可以调用微信分享，一句获取url，一句获取WinXinEntity，代码已经在上边给出了。

## 前端config函数配置
下面的代码放在网页js代码的最前面！
”var url = location.href.split(‘#’)[0];“ 页面的url也可以从后台传过来，也可以通过location.href.split(‘#’)[0]获取。
为了一点微不足道的速度，这里才用网页获取方式。（网页的url跟前面的后台签名时得url是一样的，只是绕过了ajax）

```js
<script src="http://res.wx.qq.com/open/js/jweixin-1.2.0.js"></script>
<script>
var url = location.href.split('#')[0];
    wx.config({
    debug: false,
    appId: 'xxxxxxxxxxxxxx',
    timestamp: "${wx.timestamp}",
    nonceStr: "${wx.noncestr}",
    signature: "${wx.signature}",
    jsApiList: [
      // 所有要调用的 API 都要加到这个列表中
       'checkJsApi',
       'onMenuShareTimeline',
       'onMenuShareAppMessage',
    ]
  });
  wx.ready(function () {
    // 在这里调用 API
     wx.onMenuShareTimeline({
    title: 'xxxxxxxxxx', // 分享标题
    link: url, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
    imgUrl: 'xxxxxxxxxxxxxx', // 分享图标
    success: function () {
        // 用户确认分享后执行的回调函数
    },
    cancel: function () {
        // 用户取消分享后执行的回调函数
    }
});
wx.onMenuShareAppMessage({
   title: 'xxxxxxxxxxx', // 分享标题
    desc: 'xxxxxxxxxxx', // 分享描述
    link: url, // 分享链接，该链接域名或路径必须与当前页面对应的公众号JS安全域名一致
    imgUrl: 'xxxxxxxxxx', // 分享图标
    type: '', // 分享类型,music、video或link，不填默认为link
    dataUrl: '', // 如果type是music或video，则要提供数据链接，默认为空
    success: function () {
        // 用户确认分享后执行的回调函数
    },
    cancel: function () {
        // 用户取消分享后执行的回调函数
    }
});
});
</script>
```

# 全局返回码说明
[官方说明](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1433747234)

# 链接带参数
前端的链接如果带参数，`www.xxx.com/xxx?a=1&b=2`，传给后台必须使用`encodeURIComponent`转码，[参考官方文档](https://mp.weixin.qq.com/wiki?t=resource/res_main&id=mp1421141115)，
但是官方文档并没有说，后台要把前端传的url再解码！即：
`URI.create(paramMap.get("strUrl")).getPath()`
不要使用`URLDecoder.decode(paramMap.get("strUrl"))`，该方法将要被废弃。

不带参数可以不转码，若带参数必须转码，后台再解码！！
[参考地址](https://www.cnblogs.com/vipstone/p/6732660.html)

# 微信分享JSSDK-invalid signature签名错误的解决方案
核对官方步骤，确认签名算法。

- 确认签名算法正确，可用 http://mp.weixin.qq.com/debug/cgi-bin/sandbox?t=jsapisign 页面工具进行校验。
- 确认config中nonceStr（js中驼峰标准大写S）, timestamp与用以签名中的对应noncestr, timestamp一致。
- 确认url是页面完整的url(请在当前页面alert(location.href.split('#')[0])确认)，包括'http(s)://'部分，以及'？'后面的GET参数部分,但不包括'#'hash后面的部分。
- 确认 config 中的 appid 与用来获取 jsapi_ticket 的 appid 一致。
- 确保一定缓存access_token和jsapi_ticket。
- 确保你获取用来签名的url是动态获取的，动态页面可参见实例代码中php的实现方式。如果是html的静态页面在前端通过ajax将url传到后台签名，前端需要用js获取当前页面除去'#'hash部分的链接（可用location.href.split('#')[0]获取,而且需要encodeURIComponent，后台decodeURIComponent解码），因为页面一旦分享，微信客户端会在你的链接末尾加入其它参数，如果不是动态获取当前链接，将导致分享后的页面签名失败。

# 微信小程序登录
[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)
登录流程时序图
![](wx/3.jpg)
微信的文档写的像狗屎，所以我抄了一份时序图：
![](wx/4.jpg)

大致流程如下：
- 程序客户端调用wx.login，回调里面包含js_code。
- 然后将js_code发送到服务器A（开发者服务器）,服务器A向微信服务器发起请求附带js_code、appId、secretkey和grant_type参数，以换取用户的openid和session_key（会话密钥）。
- 服务器A拿到session_key后，生成一个随机数我们叫3rd_session,以3rdSessionId为key,以session_key + openid为value缓存到redis或memcached中；因为微信团队不建议直接将session_key在网络上传输，由开发者自行生成唯一键与session_key关联。其作用是：
	+ 将3rdSessionId返回给客户端，维护小程序登录态。
	+ 通过3rdSessionId找到用户session_key和openid。
- 客户端拿到3rdSessionId后缓存到storage。
- 客户端通过wx.getUserIinfo可以获取到用户敏感数据encryptedData。
- 客户端将encryptedData、3rdSessionId和偏移量一起发送到服务器A
- 服务器A根据3rdSessionId从缓存中获取session_key
- 在服务器A使用AES解密encryptedData，从而实现用户敏感数据解密

重点在6、7、8三个环节。
AES解密三个参数：
- 密文 encryptedData
- 密钥 aesKey
- 偏移向量 iv

服务端解密流程：

- 密文和偏移向量由客户端发送给服务端，对这两个参数在服务端进行Base64_decode解编码操作。
- 根据3rdSessionId从缓存中获取session_key，对session_key进行Base64_decode可以得到aesKey，aes密钥。
- 调用aes解密方法，算法为 AES-128-CBC，数据采用PKCS#7填充。

下面结合小程序实例说明解密流程：
1. 微信登录，获取用户信息
```js
var that = this;
wx.login({
success: function (res) {
    //微信js_code
    that.setData({wxcode: res.code});
    //获取用户信息
    wx.getUserInfo({
        success: function (res) {
            //获取用户敏感数据密文和偏移向量
            that.setData({encryptedData: res.encryptedData})
            that.setData({iv: res.iv})
        }
    })
}
})
```

2. 使用code换取3rdSessionId
```js
var httpclient = require('../../utils/httpclient.js')
VAR that = this
//httpclient.req(url, data, method, success, fail)
httpclient.req(
  'http://localhost:8090/wxappservice/api/v1/wx/getSession',
  {
      apiName: 'WX_CODE',
      code: this.data.wxcode
  },
  'GET',
  function(result){
    var thirdSessionId = result.data.data.sessionId;
    that.setData({thirdSessionId: thirdSessionId})
    //将thirdSessionId放入小程序缓存
    wx.setStorageSync('thirdSessionId', thirdSessionId)
  },
  function(result){
    console.log(result)
  }
);
```

3. 发起解密请求
```js
//httpclient.req(url, data, method, success, fail)
httpclient.req(
'http://localhost:8090/wxappservice/api/v1/wx/decodeUserInfo',
  {
    apiName: 'WX_DECODE_USERINFO',
    encryptedData: this.data.encryptedData,
    iv: this.data.iv,
    sessionId: wx.getStorageSync('thirdSessionId')
  },
  'GET',
  function(result){
  //解密后的数据
    console.log(result.data)
  },
  function(result){
    console.log(result)
  }
);
```

4. 通过前端获取的code，获取openid和session_key
```
/**
 * @author: wjy
 * @description: 小程序，通过code获取session_key和openid
 */
@ResponseBody
@PostMapping("getSessionId")
public ResponseUtil getSessionId(@RequestBody Map<String,String> paramMap){
	LOGGER.info(
			"\n***************************************" + "\n" +
					"start getSessionId" + "\n" +
					"通过code获取session_key和openid" + "\n" +
					"paramMap" + paramMap +"\n" +
					"\n********************************************"
	);
	ResponseUtil response = ResponseUtil.success();
	CodeEnum code = CodeEnum.FAIL;
	JSONObject jsonObject;
	try{
		AssertUtil.assertValidate(code,CodeEnum.ERROR_2001.getCode(),"code不能为空",StringUtils.isNotEmpty(paramMap.get("code")));

		//完整地址示例
		//https://api.weixin.qq.com/sns/jscode2session?appid=APPID&secret=SECRET&js_code=JSCODE&grant_type=authorization_code
		String url = "https://api.weixin.qq.com/sns/jscode2session";
		String param =
				"appid=" + PayConstant.XCX_APPID +
				"&secret=" + PayConstant.XCX_APPSECRET +
				"&js_code=" + paramMap.get("code") +
				"&grant_type=authorization_code";
		//向微信服务器发送get请求,获取session_key和openid
		String sessionKeyAndOpenid = HttpUtil.sendGet(url, param);
		jsonObject = JSONObject.parseObject(sessionKeyAndOpenid);

		String uuid = UUID.randomUUID().toString().replaceAll("-","");

		LOGGER.info("session_key是：~~~~~~~~~~~~~~~~" + jsonObject.getString("session_key"));

		//将获取到的session_key存入redis，这里不用设置过期时间
		stringRedisTemplate.opsForValue().set(uuid,jsonObject.getString("session_key"));

		//把uuid返回给前端
		response.setData(uuid);
	}catch (Exception e){
		e.printStackTrace();
		response.setCode(code);
		response.setMessage(e.getMessage());
	}
	return response;
}
```
5. 后端用session_key，encryptedData，iv解密获取用户信息userinfo。
后端解密其实就这么简单，只要流程对了就可以解密，如果解密出错，基本就是流程出错了。不用再去换什么解密算法。
```
public Object getPhoneNumber(String encryptedData, String session_key, String iv) {
		 // 被加密的数据
		byte[] dataByte = Base64.decode(encryptedData);
		// 加密秘钥
		byte[] keyByte = Base64.decode(session_key);
		// 偏移量
		byte[] ivByte = Base64.decode(iv);
		try {
			// 如果密钥不足16位，那么就补足.  这个if 中的内容很重要
			int base = 16;
			if (keyByte.length % base != 0) {
				int groups = keyByte.length / base + (keyByte.length % base != 0 ? 1 : 0);
				byte[] temp = new byte[groups * base];
				Arrays.fill(temp, (byte) 0);
				System.arraycopy(keyByte, 0, temp, 0, keyByte.length);
				keyByte = temp;
			}
			// 初始化
			Security.addProvider(new BouncyCastleProvider());
			Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
			SecretKeySpec spec = new SecretKeySpec(keyByte, "AES");
			AlgorithmParameters parameters = AlgorithmParameters.getInstance("AES");
			parameters.init(new IvParameterSpec(ivByte));
			cipher.init(Cipher.DECRYPT_MODE, spec, parameters);// 初始化
			byte[] resultByte = cipher.doFinal(dataByte);
			if (null != resultByte && resultByte.length > 0) {
				String result = new String(resultByte, "UTF-8");
				return JSONObject.parseObject(result);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	return null;
}
```
上边分别请求了两次后台接口，其实完全可以合并成一次请求，参考下边获取手机号的内容。


[参考1](https://www.cnblogs.com/cangqinglang/p/8944681.html)
[参考2](https://blog.csdn.net/qi923701/article/details/81534820)
[参考3](https://blog.csdn.net/weixin_38429587/article/details/82814325)

# 小程序获取手机号
[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/getPhoneNumber.html)
1. 前端调用 wx.login() 获取code
```
wx.login({
    success:function(res){
        console.log('loginCode:', res.code)
    }
});
```
2. 后端拿到该code发送get请求微信接口获取 session_key和openid，返回结果类似：
```
{
	"session_key": "kI+3ookOAYC9olOcGzYPmQ",
	"openid": "oNZE65Pu0XfTg2yFP-dFks"
}
```
后台拿到数据后，我们把session_key存到redis，然后把session_key存在redis中的key返回给前端。
因为微信官方说：
[为了应用自身的数据安全，开发者服务器不应该把会话密钥下发到小程序，也不应该对外提供这个密钥](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/login.html)

3. 前端调用 getPhoneNumber组件，用户确认授权。拿到encryptedData和iv。并传给后端。
`<button open-type="getPhoneNumber" bindgetphonenumber="getPhoneNumber"> </button>`
```
getPhoneNumber: function(e) { 
    console.log(e.detail.errMsg) 
    console.log(e.detail.iv) 
    console.log(e.detail.encryptedData) 
}
```

4. 后端用session_key，encryptedData，iv解密获取手机号。
后端解密其实就这么简单，只要流程对了就可以解密，如果解密出错，基本就是流程出错了。不用再去换什么解密算法。
```
public Object getPhoneNumber(String encryptedData, String session_key, String iv) {
		 // 被加密的数据
		byte[] dataByte = Base64.decode(encryptedData);
		// 加密秘钥
		byte[] keyByte = Base64.decode(session_key);
		// 偏移量
		byte[] ivByte = Base64.decode(iv);
		try {
			// 如果密钥不足16位，那么就补足.  这个if 中的内容很重要
			int base = 16;
			if (keyByte.length % base != 0) {
				int groups = keyByte.length / base + (keyByte.length % base != 0 ? 1 : 0);
				byte[] temp = new byte[groups * base];
				Arrays.fill(temp, (byte) 0);
				System.arraycopy(keyByte, 0, temp, 0, keyByte.length);
				keyByte = temp;
			}
			// 初始化
			Security.addProvider(new BouncyCastleProvider());
			Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
			SecretKeySpec spec = new SecretKeySpec(keyByte, "AES");
			AlgorithmParameters parameters = AlgorithmParameters.getInstance("AES");
			parameters.init(new IvParameterSpec(ivByte));
			cipher.init(Cipher.DECRYPT_MODE, spec, parameters);// 初始化
			byte[] resultByte = cipher.doFinal(dataByte);
			if (null != resultByte && resultByte.length > 0) {
				String result = new String(resultByte, "UTF-8");
				return JSONObject.parseObject(result);
			}
		} catch (Exception e) {
			e.printStackTrace();
		}
	return null;
}
```

上边是请求了两次接口，其实完全可以只请求一次接口：
```
/**
 * @author: wjy
 * @description: 小程序登录，返回用户信息
 */
@ResponseBody
@PostMapping("xcxRegisterOrLogin")
public ResponseUtil xcxRegisterOrLogin(@RequestBody Map<String,String> paramMap){
	LOGGER.info(
			"\n***************************************" + "\n" +
					"start xcxRegisterOrLogin" + "\n" +
					"小程序登录，返回用户信息" + "\n" +
					"paramMap" + paramMap +"\n" +
					"\n********************************************"
	);
	ResponseUtil response = ResponseUtil.success();
	CodeEnum code = CodeEnum.FAIL;
	JSONObject jsonObject;
	try{
		AssertUtil.assertValidate(code,CodeEnum.ERROR_2001.getCode(),"iv不能为空",StringUtils.isNotEmpty(paramMap.get("iv")));
		AssertUtil.assertValidate(code,CodeEnum.ERROR_2001.getCode(),"code不能为空",StringUtils.isNotEmpty(paramMap.get("code")));
		AssertUtil.assertValidate(code,CodeEnum.ERROR_2003.getCode(),"encryptedData不能为空",StringUtils.isNotEmpty(paramMap.get("encryptedData")));

		//完整地址示例
		//https://api.weixin.qq.com/sns/jscode2session?appid=APPID&secret=SECRET&js_code=JSCODE&grant_type=authorization_code
		String url = "https://api.weixin.qq.com/sns/jscode2session";
		String param = "appid=" + PayConstant.XCX_APPID +
				"&secret=" + PayConstant.XCX_APPSECRET +
				"&js_code=" + paramMap.get("code") +
				"&grant_type=authorization_code";
		//向微信服务器发送get请求,获取session_key和openid
		String wxReturnJson = HttpUtil.sendGet(url, param);
		LOGGER.info("微信返回结果是：~~~~~~~~~~~~~~~~" + wxReturnJson);
		jsonObject = JSONObject.parseObject(wxReturnJson);

		//解密用户信息
		JSONObject userinfo = XcxUtil.decodeUserInfo(paramMap.get("encryptedData"),jsonObject.getString("session_key"),paramMap.get("iv"));
		//用户手机号
		String phoneNum = userinfo.getString("purePhoneNumber");
		LOGGER.info("用户手机号是：~~~~~~~~~~~~~~~~" + phoneNum);

		paramMap.put("mobile",phoneNum);
		//根据手机号，注册或登录
		response = usersService.registerOrLogin(paramMap,response);
	}catch (Exception e){
		e.printStackTrace();
		response.setCode(code);
		response.setMessage(e.getMessage());
	}
	return response;
}
```
这个接口，我接收了code、iv、encryptedData三个参数。
其中code是前端请求`wx.login()`返回的，`iv、encryptedData`是前端请求`wx.userinfo()`返回的。
我的接口中，先通过code请求微信服务器，获取`session_key`和`openid`，然后通过`session_key、iv、encryptedData`进行解密，就得到了用户手机号。
后续是自己业务的逻辑，即通过手机号判断自己的数据库是否有这个用户，有->返回用户信息(即登录)，没有->注册并返回用户信息。

[参考1](https://blog.csdn.net/qq_36466653/article/details/83374485)
[参考2](https://blog.csdn.net/Lonely_Devil/article/details/90024579)

获取手机号和获取用户信息，两套流程的区别，只在于前端第二次请求微信接口的不同。每个步骤，对于后台来说都是一样的。
后台只需要通过前端获取的code，获取session_id和openid，然后再用三个参数`session_key，iv，encryptedData`进行解密即可。
- 获取用户信息，前端调用`wx.getUserInfo(Object object)`[官方文档](https://developers.weixin.qq.com/miniprogram/dev/api/open-api/user-info/wx.getUserInfo.html)
- 获取手机号，前端调用`getPhoneNumber`[官方文档](https://developers.weixin.qq.com/miniprogram/dev/framework/open-ability/getPhoneNumber.html)

所以，获取用户信息和获取用户手机号，后台只需一个接口即可，但是需要注意，解密出来的，一个是手机号，一个是用户信息，所以可能还需要在自己的业务中，对两种不同的参数采取不同的处理。

**后台请求微信授权，在本地即可以测试，只要有code或encryptedData和iv(这三个参数需要前端给你)，不用非拿到线上去测。而且，小程序后台，也无需配置服务器域名啥的。**

# 小程序报错
`{"errcode":40163,"errmsg":"code been used, hints: [ req_id: UGKfDXXBe-rIpWva ]"}`
前端获取code，通过code请求后台接口，后台通过code获取openid和session_key。偶尔会报上边的错。
原因：前端传了重复的code，小程序的code只能用一次，每次使用都要重新获取，不能缓存。