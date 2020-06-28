# 十三、 RPC 调用
## 13.1  接口说明
&#160;&#160;&#160;&#160;&#160;详见： https://github.com/WisedomChainGroup/rpc-doc

## 13.2 调用方法
&#160;&#160;&#160;&#160;&#160;&#160;根据接口说明文档类型，分别使用对应的GET/POST的HTTP请求来完成RPC的调用。

&#160;&#160;&#160;&#160;&#160;&#160;官方语言默认是Java,以Java为例，使用Java实现GET和POST请求的方法常用的有两种：HTTPClient和HttpURLConnection。前者是第三方开源框架实现，对HTTP请求的封装很好，使用HTTPClient基本可以满足工作需要，其中HTTPClient3.1是org.apache.commmons.httpclient下操作远程url的工具包，HTTPClient4.5.5是org.apache.htttp.client下操作远程url的工具包。而HttpURLConnection是Java的标准请求方式。

## 13.3 更新热
&#160;&#160;&#160;&#160;&#160;&#160;在特定的情况下，为让节点在不重启节点的情况下，对节点数据进行查询，或者对其数据更新修改等操作，所以节点提供了RPC接口热更新。

&#160;&#160;&#160;&#160;&#160;&#160;但出于对节点安全的防护，本接口默认是关闭状态，如果节点想要开启此接口，需要在wdc.yml添加环境变量WISDOM_DLL_RPC:'true',开启此接口，再把相应jar包上传到指定libs文件下。

### 13.3.1 接口详情
```
Function: dll
POST/HTTP/1.1/Content-Type: applicatioc/jsoc;charset=UTF-8
RequestURL: http://00.000.0.000:19585/dll
Parameter： jsonParam(json格式)
ResponseBody: { "message":"", "data":[], "statusCode":int }
```

### 13.3.2 返回格式
```json
{
  "message":"",
  "data":[],
  "code":int
}

  *message:描述
  *data：数据
  *code：
  {
    2000正确
    5000错误
  }
```
&#160;&#160;&#160;&#160;&#160;&#160;如果code是2000，message返回SUCCESS，data返回数据

&#160;&#160;&#160;&#160;&#160;&#160;如果code是5000，message返回错误信息，date返回为空

### 13.3.3 传参
&#160;&#160;&#160;&#160;&#160;&#160;接口传入参数名称是jsonParam，json格式，不能为空，示例如下：
```json
{
      "jarname":"dll-1.0.jar",
      "classpackage":"com.dll.DemoTest",
      "methodname":"invoke",
      "classtype":[
                0,
                1,
                2,
                3,
                4,
                5
        ],
        "objectList":[
                null,
                "zhangtong",
                2,
                true,
                3.23,
                10000
        ]
}
```
jarname：需要执行的jar包全称，注意后缀也需要；

classpackage:需要执行方法的路径+类名称，路径查看类方法中的package路径；

methodname：类中的方法名称；

classtype:：调用方法传参类型，参数类型包含0是DataSource、1是string、2是int、3是boolean、4是double、5是long，五种数据类型；

objectList：具体传参数据。
### 13.3.4 注意点
&#160;&#160;&#160;&#160;&#160;&#160;classtype和objectList数组大小必须一致，且数据类型和传参参数必须匹配上，顺序保持一致，如果classtype中第0个索引是0，那objectList第0个索引放null，DataSource
会在节点中获取并传入到方法中，DataSource用于对数据库的操作。

&#160;&#160;&#160;&#160;&#160;&#160;如果是无参方法，classtype和objectList为空数组，但jarname、classpackage和
methodname参数不能为空。