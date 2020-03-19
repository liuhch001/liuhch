
!> **更新时间：** {docsify-updated} 


## 云设备  

### 简介
作为第三方云服务设备接入的依据和输入。

### 术语和定义  

| **名称** | **解释** |  
| :-----: |:------:|  
|Haier U+云平台|为物联网云平台，提供设备和应用的接入服务、数据分析服务等|   

### 公共说明  

##### 整体架构与接入流程介绍  

Iot云平台支持第三方设备通过云云对接的方式接入，整体架构如下图  

![架构图][framework]  


设备通过第三方云服务接入，首先需要通过以下流程对第三方云服务进行注册或配置：  
  **1.	线上申请使用的SystemID和SystemKey (海极网申请)；**  
  **2.	线上申请设备使用的uPlusID与DeviceKey（海极网申请）；**  
  **3.	线上在Iot平台配置第三方云服务回调地址（海极网申请）；**  
  

第三方云服务注册配置完成后，设备即可通过本接口文档接入，设备接入及通信流程如下图：  

![流程图][flow]  


第三方云服务需要实现并线下注册的回调接口如下表：

<table>
<tr>
    <td bgcolor="#3399FF"><b>回调类型</b></td>
 	<td bgcolor="#3399FF"><b>必须实现</b></td>
	<td bgcolor="#3399FF"><b>接口定义</b></td>
	<td bgcolor="#3399FF"><b>说明</b></td>
</tr>
<tr>
    <td>设备信息查询</td>
	<td>是</td>
    <td>设备信息查询</td>
	<td>Iot平台通过本接口来主动向第三方云服务定期同步设备信息。</td>

</tr>
<tr>
    <td>设备属性读</td>
	<td>否</td>
    <td>设备属性读</td>
	<td rowspan="3">Iot平台通过本接口将APP或APPServer对设备的控制指令发送给第三方云服务。如果不实现本接口，则对所有第三方云服务发送的远程控制请求会全部失败。<font color="#FF0000">特别注意：设备标准模型文档中定义的组操作（getAllProperty，getAllAlarm）必须实现。</font></td>
</tr>
<tr>
    <td>设备属性写</td>
	<td>否</td>
    <td>设备属性写</td>
</tr>
<tr>
    <td>设备操作</td>
	<td>是</td>
    <td>设备操作</td>
</tr>
<table>

Iot平台回调第三方云服务接口时，第三方云服务可通过以下两种方式对Iot平台身份进行验证：  
  **1.	将IOT平台访问IP配置为白名单；**  
  **2.	通过HTTPS双向认证，验证IOT平台证书；**  


##### HTTP接入地址定义  

本文档提供的所有接口仅支持https协议请求，接入地址如下：  
`https://uws.haier.net:443/cloudgw/接口地址`   
访问非生产环境时，需要配置第三方云服务DNS，以便是域名指向对应的环境。  

##### HTTP Header参数定义  

第三方云服务调用Iot平台HTTPS接口时，需要携带以下HTTP Header参数：  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
systemId|String|Header|是|系统ID。40位以内字符，系统全局唯一标识  
sequenceId|String|Header|是|报文流水(客户端唯一)客户端交易流水号。6-32 位。由客户端自行定义，自行生成。建议使用不带中横线的UUID，例如：e3480efdef6a410f9f60801707671bcc  
sign|String|Header|是|对请求进行签名运算产生的签名 [签名算法][sign]   
timestamp|String|Header|是|Unix时间戳，当前UTC标准时间，精确到毫秒，例如：1557900146503    
Content-Type|String|Header|是|本接口Payload内容仅支持UTF-8编码的Json格式数据，值必须为application/json;charset=UTF-8  


Iot平台回调第三方接口时，也会同样携带以上HTTP Header参数。
   

请求头信息样例（其中#为占位符，不代表具体含义）：  


	Connection: keep-alive  
	systemId: ######    
	deviceId: ######  
	sequenceId: e3480efdef6a410f9f60801707671bcc  
	sign: bb2a5c1e432eac8bea8eecb89b408937382e7e95486ee0a60944a46504fa0015 （此字段需计算）  
	timestamp: 1557900146503  
	Content-Type: application/json;charset=UTF-8  


##### 错误码定义  

###### 公共错误码  

错误码W00001~W10000及部分其他错误码，为公共错误码，及Iot平台的接口与第三方云服务实现的回调接口都会出现的错误码，定义如下：  

| **错误码** | **描述** |
| :-------: |:-----:|
|00000|成功|
|W00001|通用错误，表示暂时未定义或无法区分的错误。|
|W00002|内部错误，表示系统内部发生且不对外开放的错误。正常业务下不会返回该错误。|
|W00003|格式错误，即Payload不是UTF-8编码的Json格式|
|W00004|参数错误，即接口Payload中某必填字段不存在、某字段值不合法等|
|W01001|sequenceId错误，即HTTP Header中的sequenceId字段为空或者超长|
|W01002|时间戳错误，即时间戳与当前时间相差较大|
|C00001|systemId错误，即HTTP Header中的systemId字段为空或在Iot平台不存在|
|C00002|无访问授权，指设备云接入网关未在UWS接口注册|
|D00001|HTTP Header签名校验错误|

以上错误码在后续每一个接口中均可能返回，后面章节接口中，不再单独描述。

###### Iot平台接口错误码  

错误码W10000~W20000，为仅Iot平台的接口出现的错误码，定义如下：  

| **错误码** | **描述** |
| :-------: |:-----:|
|W10001|设备未注册|
|W10002|设备注册时，U+产品编号不存在|
|W10003|设备注册时，设备产品信息签名错误，通常为deviceKey错误导致|
|W10004|设备绑定时，用户token无效|
|W10005|设备绑定时，用户绑定时间窗未开启|
|W10006|设备绑定时，用户已达到绑定设备数量上限|
|W10007|设备控制应答sn错误|

###### 第三方云服务回调接口错误码 

错误码W20000~W30000，为仅第三方云服务实现的回调接口出现的错误码，定义如下：  


| **错误码** | **描述** |
| :-------: |:-----:|
|W20010|设备不存在|

###### <span id="jump1">第三方云设备控制异步应答错误码</span>   
<!--  <h5 id="1">第三方云设备控制异步应答错误码</h5>--> 

设备控制异步应答错误码为整数类型，该错误码会直接返回给App处理，定义如下：  

| **错误码** | **描述** |
| :-------: |:-----:|
|0 |成功。|
|1 |通用错误码，表示暂时未定义或无法区分的错误。|
|3 |格式错误，表示PayLoad消息体格式错误，例如不是合法的Json格式。|
|4 |缺少参数，表示PayLoad消息体中缺少部分必填字段。|
|11|控制指令执行超时。|
|12|无效命令。主要指命令执行逻辑错误。例如关机状态下再关机等。|
|13|非法值。主要指设备操作的操作值非法。例如温度设置的温度值越界。|
|17|不支持的命令。主要指读属性或写属性请求时，设备无此属性，或者操作时，设备不支持此操作。|


### 设备配网  

设备配网有两种方式：设备集成配网SDK(推荐方式)，设备自身具备联网能力。


#### 设备集成配网SDK(推荐)   
  
设备底板无需改动，模块集成配网SDK，支持U+ APP配置绑定。  
设备云端按本文档API的定义调用，实现设备控制与状态数据同步。  
  
 

在海极网下载SDK申请表，申请配网SDK：

![设备集成配网sdk][configuration_sdk1]   

![设备集成配网sdk][configuration_sdk2]   

注：配网SDK大小为2KB。

#### 设备自身具备联网能力  

设备无法集成U+配网SDK且自身具备联网能力，需要设备(如带屏设备)生成绑定二维码，U+ APP扫描绑定二维码，或U+ APP生成二维码，设备(如摄像头类设备)解析并通过云设备回传U+云平台，实现设备发起绑定。  
设备云端按本文档API的定义调用，实现设备控制与状态数据同步。    
  
请联系平台王世腾(wangshiteng@haier.com)。

### 设备接入  

#### 设备注册   
> 调用Iot平台。  
> 设备接入Iot平台，需要先在Iot平台注册，否则后续对该设备的所有动作均会失败。设备可以重复注册，machineId是设备物理唯一标识，对于同一个systemId的同一个machineId，重复注册时返回的deviceId始终是相同的，设备信息（uPlusID、online）也会直接覆盖该设备的原有信息。


##### 1、接口定义
?> **接入地址：** `/v1/dev/reg` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
machineId|String|Body|是|要注册的设备的物理唯一标识，该标识在同一systemId下需唯一。
字符串长度范围：1~32，（字符集可取值[0-9][A-Z] [a-z]）。对于WIFI类设备，建议使用MAC地址，格式为全大写无连接符十六进制字符串，例如CC1122334455。对于2G或NB类设备，建议使用IMEI地址，格式为全数据字符串，例如860123456789876。
uPlusID|String|Body|是|U+产品编号，接入U+产品的唯一标识，由U+统一定义。字符串长度范围：64。
devSign|String|Body|是|设备产品信息签名，用于设备身份认证，全小写十六进制字符串，字符串长度范围：64。例如：996a4a51cfc228f7d5044319767d6f792f106460e86a9050b79dcee0a6dd31d9 算法如下（+表示字符串拼接）：SHA256（machineId+uPlusID+deviceKey+ timestamp）其中：deviceKey为海极网创建uPlusID时分配；timestamp为HTTP Header中的时间戳字段；
online|Boolean|Body|是|设备当前在线状态。true为在线；false为离线；  
uPlusCodeT|String|Body|否|U+产品整机型号编码（即成品编码）。设备上报的型号编码必须是在海极网注册或录入的，否则设备注册会失败。设备上报型号信息后，会直接覆盖平台该设备现有型号信息。字符串长度范围：9~11




**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10002、W10003
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。
deviceId|String|Body|是|注册成功后Iot平台分配的设备ID。字符串长度范围：1~32。



##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/reg
Body：
	{
    "deviceMac" : "***",
    "uPlusID" : "***",
    "devSign" : "***",
    "online" : true | false
	}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***",
    "deviceId" : "***"
}
```

<!-- 注释开始
#### 设备绑定
> 调用Iot平台。  
> 设备在Iot平台注册，即可将该设备绑定在一个优家用户下。调用本接口绑定设备时，设备绑定必须先由uSDK对要绑定的用户开启绑定时间窗（每次开启，20分钟内有效，且只能绑定一次），否则即绑定失败。如果绑定成功，本设备与之前其他用户的绑定关系会被自动解除。一个优家用户，最多可以绑定300个设备。  


##### 1、接口定义
?> **接入地址：** `/v1/dev/bind` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  
token|String|Body|否|设备要绑定的用户Token。字符串长度范围：1~32。



**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001、W10004、W10005、W10006  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。



##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/bind  
Body：
{
    "deviceId" : "***",
    "token" : "***"
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```
注释结束  -->

#### 设备上线  
> 调用Iot平台。  
> Iot平台会维护设备的在线状态，设备注册到Iot平台时，会携带设备初始在线状态，随后可以通过设备上线与设备下线更新设备在线状态。
Iot平台仅被动维护设备在线状态，任何设备在线状态变化，都需要第三方云服务主动维护。
本接口会直接更新设备在Iot平台的在线状态，即不论设备当前是否在线，均会直接更新为在线。



##### 1、接口定义
?> **接入地址：** `/v1/dev/online` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  


**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。



##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/online  
Body：
{
    "deviceId" : "***"
}


```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```  

#### 设备下线  
> 调用Iot平台。  
> Iot平台会维护设备的在线状态，设备注册到Iot平台时，会携带设备初始在线状态，随后可以通过设备上线与设备下线更新设备在线状态。
Iot平台仅被动维护设备在线状态，任何设备在线状态变化，都需要第三方云服务主动维护。
本接口会直接更新设备在Iot平台的在线状态，即不论设备当前是否离线，均会直接更新为离线。



##### 1、接口定义
?> **接入地址：** `/v1/dev/offline` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  


**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。



##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/offline  
Body：
{
    "deviceId" : "***"
}


```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```

<!-- 注释开始
#### 开启设备绑定时间窗  
> 调用Iot平台。  
> 设备也可以通过优家APP直接进行绑定，但需要在优家APP绑定之前，必须调用本接口对设备开启绑定时间窗，否则即会绑定失败。
对设备开启绑定时间窗后，20分钟内有效，如果调用本接口时当前设备的绑定时间窗已开启，则重置时间窗有效期为20分钟；


##### 1、接口定义
?> **接入地址：** `/v1/dev/setBindTimeWindow` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  


**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。



##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/setBindTimeWindow  
Body：
{
    "deviceId" : "***"
}


```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```
注释结束  -->

#### 设备信息查询
> Iot平台回调。    
> Iot平台通过本接口来主动向第三方云服务定期同步设备信息，以便应对由于服务或网络中断导致的设备信息或在线状态不同步问题。  


##### 1、接口定义
?> **回调类型：** `设备信息查询` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  
machineId|String|Body|是|设备注册时使用的设备物理唯一标识。字符串长度范围：1~32。    
callbackType|String|Body|是|回调类型，值固定为：`devInfoQuery`  
rptProperty|Boolean|Body|是|是否立即上报属性状态。如果值为true，则第三方云服务在应答本接口后，应立即调用“设备属性状态上报”接口上报设备当前全部属性状态。如果值为false，则忽略。
rptAlarm|Boolean|Body|是|是否立即上报报警状态。如果值为true，则第三方云服务在应答本接口后，应立即调用“设备报警状态上报”接口上报设备当前全部报警状态。如果值为false，则忽略。
 

**输出参数:**  


参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W20010  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。
uPlusID|String|Body|是|U+产品编号，接入U+产品的唯一标识，由U+统一定义。字符串长度范围：64。
online|Boolean|Body|是|设备当前在线状态。true为在线；false为离线；



##### 2、请求样例

**请求样例**

```
请求地址：https://***    
Body：
{
    "callbackType" : "devInfoQuery",
    "deviceId" : "***",
    "rptProperty" : true | false,
    "rptAlarm" : true | false
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***",
    "uPlusID" : "***",
    "online" : true | false
}

```




### 设备上报


#### 设备属性状态上报
> 调用Iot平台。  
> 设备注册后，即可在适合的时机上报设备的属性状态，属性上报会实时通过uSDK发送到App上用来展示。
为了保证设备属性变化展示的实时性，设备属性状态一旦发生变化，应该立即上报。
设备每次都需要上报当前全部属性状态，以便使用方可以实时得知设备当前最新的全部属性状态。
 


##### 1、接口定义
?> **接入地址：** `/v1/dev/property` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  
propertys|Property数组|Body|是|设备属性列表；Property类型定义见下面`参数类型定义`。数组长度范围：0~256。
  

**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

?> **参数类型定义:** `Property类型定义`    

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
name|String|Body|是|属性名。字符串长度范围：1~64。
value|String|Body|是|属性值。字符串长度范围：0~64。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/property   
Body：
{
    "deviceId" : "***",
    "propertys" : [
        { "name" : "***", "value" : "***" },
        { "name" : "***", "value" : "***" },
        ……
    ]
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```


#### 设备报警状态上报
> 调用Iot平台。  
> 设备注册后，即可在适合的时机上报设备的报警状态，设备报警上报会实时通过uSDK发送到App上用来展示。
为了保证设备报警变化展示的实时性，设备报警状态一旦发生变化，应该立即上报。
设备每次都需要上报当前正在发生的全部报警，以便使用方可以实时得知设备当前最新的全部报警状态。



##### 1、接口定义
?> **接入地址：** `/v1/dev/alarm` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。  
alarms|Alarm数组|Body|是|设备报警列表；Alarm类型定义见下面`参数类型定义`。数组长度范围：0~256。


**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

?> **参数类型定义:** `Alarm类型定义`    

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
name|String|Body|是|报警名。字符串长度范围：1~64。
value|String|Body|是|报警值。某些报警仅有报警名时，报警值填空字符串。字符串长度范围：0~64。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/alarm   
Body：
{
    "deviceId" : "***",
    "alarms" : [
        { "name" : "***", "value" : "***" },
        { "name" : "***", "value" : "***" },
        ……
    ]
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```



### 设备控制


#### 设备控制请求--属性读
> Iot平台回调。  
> 设备注册到Iot平台并且当前在线，App或AppServer控制设备时，即会使用本回调接口通过第三方云服务发送给设备执行。
对设备控制是一个异步请求，第三方云服务收到本请求时需要立即返回，即接口返回时仅代表第三方云服务收到本控制请求，而实际向设备下发及设备执行结果，第三方云服务需要通过主动调用“设备控制应答”接口发送至Iot平台。


##### 1、接口定义
?> **回调类型：** `设备属性读` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
callbackType|String|Body|是|回调类型，值固定为：`propertyRead`  
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
machineId|String|Body|是|设备注册时使用的设备物理唯一标识。字符串长度范围：1~32。
token|String|Body|否|设备绑定的第三方用户token，字符串长度范围：1~32。如当前设备没有授权的第三方用户，则无此字段。
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
propertyName|String|Body|是|要读取的设备属性名。字符串长度范围：无限制。


**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W20010  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://***   
Body：
{
    "callbackType" : "propertyRead",
    "deviceId" : "***",
    "sn" : "***",
    "propertyName" : "***"
 }

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```


#### 设备控制应答--属性读  
> 调用Iot平台。  
> 设备向第三方云服务发送，或者第三方云服务从设备取到设备控制应答后，需要调用本接口，将控制结果发送到Iot平台。
如果控制序列号与请求不同，本接口也会返回成功，但会直接丢弃本应答消息。


##### 1、接口定义
?> **接入地址：** `/v1/dev/opack/propertyRead` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
errNo|Integer|Body|是|设备控制应答错误码，表示本次设备控制结果，该错误码会直接返回至App处理。可返回错误码见[第三方云设备控制异步应答错误码](#jump1)定义。
propertyName|String|Body|是|要读取的设备属性名。字符串长度范围：1~64。
propertyValue|String|Body|是|读取到的设备属性值。字符串长度范围：0~64。


**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001，W10007 
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/opack/propertyRead   
Body：
{
    "deviceId" : "***",
    "sn" : "***",
    "errNo" : ***,
    "propertyName" : "***"
    "propertyValue" : "***"
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```



#### 设备控制请求--属性写  
> Iot平台回调。
> 设备注册到Iot平台并且当前在线，App或AppServer控制设备时，即会使用本回调接口通过第三方云服务发送给设备执行。
对设备控制是一个异步请求，第三方云服务收到本请求时需要立即返回，即接口返回时仅代表第三方云服务收到本控制请求，而实际向设备下发及设备执行结果，第三方云服务需要通过主动调用“设备控制应答”接口发送至Iot平台。


##### 1、接口定义
?> **回调类型：** `设备属性写` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
callbackType|String|Body|是|回调类型，值固定为：`propertyWrite`  
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
machineId|String|Body|是|设备注册时使用的设备物理唯一标识。字符串长度范围：1~32。
token|String|Body|否|设备绑定的第三方用户token，字符串长度范围：1~32。如当前设备没有授权的第三方用户，则无此字段。     
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
propertyName|String|Body|是|要写入的设备属性名。字符串长度范围：无限制。
propertyValue|String|Body|是|要写入的设备属性值。字符串长度范围：无限制。



**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W20010  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://***   
Body：
{
    "callbackType" : "propertyWrite",
    "deviceId" : "***",
    "sn" : "***",
    "propertyName" : "***",
    "propertyValue" : "***"
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```



#### 设备控制应答--属性写
> 调用Iot平台。  
> 设备向第三方云服务发送，或者第三方云服务从设备取到设备控制应答后，需要调用本接口，将控制结果发送到Iot平台。
如果控制序列号与请求不同，本接口也会返回成功，但会直接丢弃本应答消息。


##### 1、接口定义
?> **接入地址：** `/v1/dev/opack/propertyWrite` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
errNo|Integer|Body|是|设备控制应答错误码，表示本次设备控制结果，该错误码会直接返回至App处理。可返回错误码见[第三方云设备控制异步应答错误码](#jump1)定义。


**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001，W10007 
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/opack/propertyWrite  
Body：
{
    "deviceId" : "***",
    "sn" : "***",
    "errNo" : ***
 }

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```




#### 设备控制请求--操作  
> Iot平台回调。    
> 设备注册到Iot平台并且当前在线，App或AppServer控制设备时，即会使用本回调接口通过第三方云服务发送给设备执行。
对设备控制是一个异步请求，第三方云服务收到本请求时需要立即返回，即接口返回时仅代表第三方云服务收到本控制请求，而实际向设备下发及设备执行结果，第三方云服务需要通过主动调用“设备控制应答”接口发送至Iot平台。
<font color="#FF0000">特别注意：设备标准模型文档中定义的组操作（getAllProperty，getAllAlarm）必须实现。</font>



##### 1、接口定义
?> **回调类型：** `设备操作` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
callbackType|String|Body|是|回调类型，值固定为：`operation`  
machineId|String|Body|是|设备注册时使用的设备物理唯一标识。字符串长度范围：1~32。
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
token|String|Body|否|设备绑定的第三方用户token，字符串长度范围：1~32。如当前设备没有授权的第三方用户，则无此字段。
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
propertyName|String|Body|是|设备操作名。字符串长度范围：无限制。
parameters|Parameter数组|Body|是|设备操作请求参数列表。如当前操作请求无参数，则值为空数组[]。数组长度范围：无限制。



**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W20010  
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。


?> **参数类型定义：** <span id="jump">`Parameter类型定义`<span>  
  
参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
name|String|Body|是|参数名。字符串长度范围：无限制。
value|String|Body|是|参数值。字符串长度范围：无限制。 

 
##### 2、请求样例

**请求样例**

```
请求地址：https://***   
Body：
{
    "callbackType" : "operation",
    "deviceId" : "***",
    "sn" : "***",
    "operationName" : "***",
    "parameters" : [
        { "name" : "***", "value" : "***" },
        { "name" : "***", "value" : "***" },
        ……
    ]
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```


#### 设备控制应答--操作  
> 调用Iot平台。  
> 设备向第三方云服务发送，或者第三方云服务从设备取到设备控制应答后，需要调用本接口，将控制结果发送到Iot平台。
如果控制序列号与请求不同，本接口也会返回成功，但会直接丢弃本应答消息。


##### 1、接口定义
?> **接入地址：** `/v1/dev/opack/operation` </br>
**HTTP Method：** POST

**输入参数**

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
deviceId|String|Body|是|设备ID。字符串长度范围：1~32。
sn|String|Body|是|控制序列号，对于一次控制，控制应答的序列号需要同控制请求的序列号相同。字符串长度范围：1~64。
errNo|Integer|Body|是|设备控制应答错误码，表示本次设备控制结果，该错误码会直接返回至App处理。可返回错误码见[第三方云设备控制异步应答错误码](#jump1)定义。
parameters|[Parameter数组](#jump)|Body|是|设备操作应答参数列表。
如当前操作应答无参数，则值为空数组[]。数组长度范围：0~256。


**输出参数:**  

参数名|类型|位置|必填|说明
:-|:-:|:-:|:-:|:-
retCode|String|Body|是|返回码，00000代表请求成功，其它值代表错误 见“错误码定义”章节。可返回错误：公共错误码、W10001，W10007 
retInfo|String|Body|是|错误描述信息，本描述信息仅是用于调试的返回信息，不支持国际化，不能直接显示在UI上。字符串长度范围：0~256。详细错误情况以retCode在本接口文档中对应描述为准。

 
##### 2、请求样例

**请求样例**

```
请求地址：https://uws.haier.net/cloudgw/v1/dev/opack/operation
Body：
{
    "deviceId" : "***",
    "sn" : "***",
    "errNo" : ***,
    "parameters" : [
        { "name" : "***", "value" : "***" },
        { "name" : "***", "value" : "***" },
        ……
    ]
}

```
**请求应答**
```
{
    "retCode" : "00000",
    "retInfo" : "***"
}

```









[^-^]:常用图片注释
[framework]:_media/_Cloudgw/framework.png 
[sign]:https://haier-iot.github.io/guide/#/zh-cn/Standard/Basic?id=%e7%ad%be%e5%90%8d%e7%ae%97%e6%b3%95
[flow]:_media/_Cloudgw/flow.png 
[configuration_sdk1]:_media/_Cloudgw/configuration_sdk1.png 
[configuration_sdk2]:_media/_Cloudgw/configuration_sdk2.png 










