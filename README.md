pk接口文档(v.200310)
=
文档内容最后更新于：2020-03-10 

特别注意：
-
1.**时间戳为秒级，非毫秒级，毫秒级请/1000**

2.**returnUrl/notifyUrl 为完整地址,含有协议+端口。如果回调通知地址（notifyUrl）不传，平台不会发起异步回调，需要调用查询接口确认订单状态。**

3.**金额为整数，非小数，以分为单位，不能包含有“·”号**,例：123 即 1.23 元。

4.**商户编号需要从商户后台首页获取，并非登陆账号**，商户密钥（apikey）每次刷新都会重新随机生成，保存好最后一次刷新的密钥进行对接即可。

5.**商户接收异步通知时，不要写死固定参数接收，请使用通用的json/map 对象接收，这样可接收完整参数，然后对json/map 里面的参数进行签名校验。如果只接收固定参数，会导致签名验证失败。后期如果通知增加参数，也可以不用修改代码**

6.**商户测试时如果要确认能不能回调以及验证签名是否成功，可生成订单后直接取消，取消后系统便会有通知。测试完取消通过后，再联系客服测试成功订单**

接口规范
-
1.字符编码：UTF-8

2.Content-Type：application/json

3.URL 传输参数需要对参数进行 UrlEncode

接口调用所需条件
-
1.网关地址：请联系客服

2.商户编号(merchantNo)

3.商户密钥(apiKey)

签名（sign）算法
-
DigestUtils.md5Hex(originalStr + "key=" + apiKey)

1.originalStr: 除sign参数外其他参数值非空（空值或者空字符串）的参数按参数名称字母正序排序然后以name=UrlEncode(value)形式组合， 通过&拼接得到结果将apiKey拼接在最后。<br>
i.注：空值（空值或者空字符串）不参与签名。<br>
ii.注：value需要进行UrlEncode编码

示例:
amount=100&merchantNo=20200113185052721173545318&notifyUrl=https%3A%2F%2Fwww.baidu.com%2F&orderNo=123456789000&payMode=ebank&returnUrl=https%3A%2F%2Fwww.baidu.com%2F&ts=1581920707&key=06f231e8483243e28296229


2.DigestUtils.md5Hex(originalStr + "key=" + apiKey) <br>
i.用DigestUtils.md5Hex算法将“originalStr + "key=" + apiKey”进行加密得到签名信息

3.[c# demo](https://github.com/passpay/passpay/tree/master/C%23-demo)

4.[java demo](https://github.com/passpay/passpay/tree/master/java)

5.[php demo](https://github.com/passpay/passpay/tree/master/php)

同步通知 （returnUrl）
-
当创建订单时传入返回地址，订单结束后，用户点击“返回商户”，会在返回链接带上参数（returnUrl?urlparams）。参数内容参考统一返回参数，可通过签名算法计算签名的正确性。例：<br>
returnUrl?<br>

    amount=100&
    
    bankName=广发银行&
    
    bankNo=62146202210026980&
    
    merchantNo=20200113185052721173545318&
    
    name=王五&
    
    orderNo=o-1008614&
    
    orderStatus=50&
    
    payMode=ebank&
    
    payNo=20200213173023981153464943&

    payStatus=30&
    
    payTime=1581586702&
    
    sign=3aff08ebde950423acbc267e363588ec&
    
    ts=1581585888
    
 异步回调 （notifyUrl）
 -
当创建订单时传入异步回调地址时，订单结束后（用户取消订单(-30)、用户支付超时（-40）、订单失败（-50）、订单已完成（50））进行通知，总共通知3次，每次间隔10 分钟，超时时间为10s，处理成功后返回 success，返回其他字符表示处理失败，会继续进行后续通知。通知内容参考统一返回参数，可通过签名算法计算签名的正确性 例：<br>
curl -X POST "回调地址"<br>
  -H 'content-type: application/json' <br>
  -d '{<br>
  
    amount=100&
    
    bankName=广发银行&
    
    bankNo=62146202210026980&
    
    merchantNo=20200113185052721173545318&
    
    name=王五&
    
    orderNo=o-1008614&
    
    orderStatus=50&
    
    payMode=ebank&
    
    payNo=20200213173023981153464943&
    
    payStatus=30&
    
    payTime=1581586702&
    
    sign=3aff08ebde950423acbc267e363588ec&
    
    ts=1581585888

}'

1.订单接口内容
-
1.创建订单接口

i.使用场景：当商户创建时，根据下面参数，生成订单信息。<br>
ii.请求方式：POST <br>
iii.请求地址：网关地址+/pk-order/#/create?urlparams  <br>
iv.请求参数

参数名称  | 必须  | 数据类型 | 示例| 参数说明
 ---- | ----- | ------  | ------    | ------
 amount  | 是 | 整数 | 100 | 金额,以分为单位；最小值100，即1元
 merchantNo  | 是 | 字符串 | 20200113185052721173545318 | 商户编号
 orderNo  | 是 | 字符串(<50) | 123456789000 | 商户订单编号
 payMode  | 是 | 字符串 | ebank | 支付模式，请登陆商户后台获取
 ts  | 是 | 整数 | 1575948756 | 商户订单时间戳（秒级）
 notifyUrl  | 否 | 字符串 | https://www.baidu.com/notify | 后台通知地址
 returnUrl  | 否 | 字符串 | https://www.baidu.com | 支付完成用户返回地址
 sign  | 是 | 字符串 | 2A1FEB481909CBE0CA823D6FA31... | 参数签名，请按照签名算法生成

2.查询订单接口

i.使用场景：查询订单信息。<br>
ii.请求方式：POST<br>
iii.请求地址：网关地址+ /cat-pay/open/orde/query  <br>
iv.请求参数

参数名称  | 必须  | 数据类型 | 示例| 参数说明
 ---- | ----- | ------  | ------    | ------
 merchantNo  | 是 | 字符串 | 20200113185052721173545318 | 商户编号
 orderNo  | 是 | 字符串(<50) | 123456789000 | 商户订单编号
 ts  | 是 | 整数 | 1575948756 | 商户订单时间戳（秒级）
 sign  | 是 | 字符串 | 2A1FEB481909CBE0CA823D6FA31... | 参数签名，请按照签名算法生成

统一返回参数
-
1.参数内容

 参数名称  | 必须  | 数据类型 | 示例| 参数说明
 ---- | ----- | ------  | ------    | ------
 amount  | 是 | 整数 | 100 | 金额,以分为单位
 bankName  | 否 | 字符串 | 广发银行 | 银行名称
 bankNo  | 是 | 字符串(<50) | 62146202210026980 | 银行卡号
 merchantNo  | 是 | 字符串 | 20200113185052721173545318 | 商户编号
 name  | 是 | 字符串 | 王五 | 产品名称
 orderNo  | 是 | 字符串 | 201912081855183951ab02e | 商户订单编号
 orderStatus  | 是 | 整数 | 50 | 订单状态，请参考订单状态枚举
 payMode  | 是 | 字符串 | ebank | 支付模式
 payNo  | 否 | 字符串 | 20191209194326631108714792 | 支付订单编号
 payStatus  | 否 | 整数 | 30 | 支付状态，请参考支付状态枚举
 payTime  | 否 | 整数 | 1575948756 | 支付成功时间（秒级）
 sign  | 是 | 字符串 | $2a$10$JwOX9nmVHrE6o8vcoSmyd.T6... | 参数签名，使用DigestUtils.md5Hex校验方法校验
 ts  | 是 | 整数 | 1575948756 | 商户订单时间戳（秒级）
 
 
2.订单状态（orderStatus）枚举

值  | 说明  
 ---- | -----   
 30  | 支付等待中
 -30  | 用户取消订单
 -40  | 用户支付超时
 -50  | 订单失败
 50  | 订单已完成
    
3.支付状态（payStatus）枚举
 
  值  | 说明  
 ---- | -----  
 10  | 等待支付 
 -10  | 支付超时
 -20  | 支付取消
 30  | 支付成功
 -30  | 支付失败


**以订单状态为主进行判断，支付超时后状态可能会收到支付成功状态通知，请注意处理**

2.下发api内容
-
1.银行卡下发

i.使用场景：商户银行卡下发<br>
ii.请求方式：POST <br>
iii.请求地址：网关地址+/any-pay/open/merchant/withdraw-apply  <br>
iv.请求参数

参数名称  | 必须  | 数据类型 | 示例| 参数说明
 ---- | ----- | ------  | ------    | ------
 merchantNo  | 是 | 字符串 | 20200221093008088186315303 | 商户编号
 bankName  | 是 | 字符串 | 平安银行 | 银行名称
 bankNo  | 是 | 字符串(<50) | 6230580000270144210 | 银行卡号
 bankUser  | 是 | 字符串 | 张三 | 持卡人
 passwd  | 是 | 字符串 | MD5(密码) | 提现密码（需要md5）
 amount  | 是 | 整数 | 1000 | 金额（分）
 ts  | 是 | 整数 | 1586345315063 | 商户订单时间戳（秒级）
 sign  | 是 | 字符串 | $2a$10$GLvMhH7Vr9zSP7CRE... | 参数签名，请按照签名算法生成



