# 13. RPC Call
## 13.1  Interface Description
&#160;&#160;&#160;&#160;&#160;See： https://github.com/WisedomChainGroup/rpc-doc

## 13.2 Call Method
&#160;&#160;&#160;&#160;&#160;&#160;According to the document type of the interface description, the corresponding GET/POST HTTP request is used to complete the RPC call.

&#160;&#160;&#160;&#160;&#160;&#160;The official language is Java by default. Taking Java as an example, there are two commonly used methods to implement GET and POST requests in Java: HTTPClient and HttpURLConnection. The former is a third-party open-source framework, which encapsulates HTTP requests very well. Using HTTPClient can basically meet the needs of work.HTTPClient3.1 is the toolkit for operating remote URL under org.apache.commmons.httpclient, and HTTPClient4.5.5 is the toolkit for operating remote URL under org.apache.htttp.client. Httpurlconnection is the standard request mode of Java.

## 13.3 Hot Patching
&#160;&#160;&#160;&#160;&#160;&#160;In specific cases, in order to make the node query the node data or update and modify its data without restarting the node, the node provides RPC interface hot patching.

&#160;&#160;&#160;&#160;&#160;&#160;However, for the protection of node security, the interface is closed by default. If a node wants to open this interface, it needs to add an environment variable WISDOM_DLL_RPC:'true' in wdc.yml to open the interface, and then upload the corresponding jar package to the specified libs file.

### 13.3.1 Interface Details
```
Function: dll
POST/HTTP/1.1/Content-Type: applicatioc/jsoc;charset=UTF-8
RequestURL: http://00.000.0.000:19585/dll
Parameter： jsonParam(json format)
ResponseBody: { "message":"", "data":[], "statusCode":int }
```

### 13.3.2 Return Format
```json
{
  "message":"",
  "data":[],
  "code":int
}

  *message:describe
  *data：data
  *code：
  {
    2000correct
    5000error
  }
```
&#160;&#160;&#160;&#160;&#160;&#160;If the code is 2000, message returns success and data returns data

&#160;&#160;&#160;&#160;&#160;&#160;If the code is 5000, message returns an error message and date returns null

### 13.3.3 Transfer Parameters
&#160;&#160;&#160;&#160;&#160;&#160;The parameter name passed in by the interface is jsonParam in json format and cannot be empty. The example is as follows:
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
jarname: the full name of the jar package to be executed. Note that the suffix is also required;

classpackage: needs to execute the path of the method + class name, and the path looks at the package path in the class method;

methodname: the name of the method in the class;

classtype: the parameter type of the calling method. The parameter types include five data types: 0 is DataSource, 1 is string, 2 is int, 3 is boolean, 4 is double, 5 is long;

Objectlist: specific transfer parameter data.

### 13.3.4 Notes
&#160;&#160;&#160;&#160;&#160;&#160;The size of the array of classtype and objectlist must be consistent, and the data type and parameter passing parameters must match in the same order. If the 0 th index in the classtype is 0, then the 0 th index of the objectlist will be null.The DataSource will be obtained from the node and passed into the method. Datasource is used for database operation.

&#160;&#160;&#160;&#160;&#160;&#160;If it is a nonparametric method, the classtype and objectlist are empty arrays, but the jarname, classpackage, and methodname parameters cannot be empty.
