# ES-Head 连接Elasticsearch7.1.1 错误 406
> https://www.lmaye.com/2019/07/06/20190706101752/

## 1. 页面数据查询报406错误

```

 {"error":"Content-Type header [application/x-www-form-urlencoded] is not supported","status":406}

```

## 2. 解决方法:  
进入head安装目录； 
```
找到vendor.js，编辑vendor.js 共有两处  
6886行: /contentType: “application/x-www-form-urlencoded改成  
contentType: "application/json;charset=UTF-8"

7573行: var inspectData = s.contentType === “application/x-www-form-urlencoded” && 改成
var inspectData = s.contentType === "application/json;charset=UTF-8" &&

``

## 3.关闭浏览器重新打开ES-head插件
