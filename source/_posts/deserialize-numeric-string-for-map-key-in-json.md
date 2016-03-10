title: deserialize numeric string for map key in json
tags:
  - java
  - jackson
  - deserialization
date: 2016-02-28 22:21:00
---

最近反序列化json的时候，踩到了一个jackson转换map key的坑。踩坑时反序列化的数据格式如下:
```
{
  "data": {
    "1": "观景",
    "2": "小资",
    "3": "院落"
  },
  "ret": true,
  "errcode": 0
}
```
然后使用如下代码对其进行反序列化：
```
String retStr = "{\"data\":{\"1\":\"观景\",\"2\":\"小资\",\"3\":\"院落\"},\"ret\":true,\"errcode\":0}";
ObjectMapper objectMapper = new ObjectMapper();
Map<String, Object> retMap = objectMapper.readValue(retStr, new TypeReference<Map<String, Object>>(){});
Map<Integer, String> dataMap = (Map<Integer, String>) retMap.get("data");
```
之后针对一些数字，需要判断dataMap中是否存在这个key，这里调用的是Map的containsKey()方法：
```
boolean existing = dataMap.containsKey(1);
```
明明dataMap中存在key=1的键值对，但是得到的结果却是false。
此时debug查看dataMap的内容(如下)，发现dataMap中是存在key=1的键值对的。但是查看key的类型，并不是int而是char，而上边调用constainsKey()方法传入的参数是int，因此返回false。
![cast_numeric_string_to_Integer01.png](/blog/2016/02/28/deserialize-numeric-string-for-map-key-in-json/image01.png)
深扒了一下jackson的底层代码，发现它在反序列化map的key的时候，如果没有明确指定key的类型，将默认按照String类型处理，最终调用的是<code>StringKD</code>的<code>_parse</code>方法，直接将json中的数字以字符串的形式返回了。
```
final static class StringKD extends StdKeyDeserializer
{
    @Override
    public String _parse(String key, DeserializationContext ctxt) throws JsonMappingException {
        return key;
    }
}    
```
修改代码如下，显式地将key转化为Integer类型，我们得到了想要的结果。
```
String retStr = "{\"data\":{\"1\":\"观景\",\"2\":\"小资\",\"3\":\"院落\"},\"ret\":true,\"errcode\":0}";
ObjectMapper objectMapper = new ObjectMapper();
Map<String, Object> retMap = objectMapper.readValue(retStr, new TypeReference<Map<String, Object>>(){});
Map<String, String> dataMap = (Map<String, String>) retMap.get("data");
Map<Integer, String> tempMap = Maps.newHashMapWithExpectedSize(dataMap.size());
for (Map.Entry<String, String> entry : dataMap.entrySet()) {
    tempMap.put(Integer.valueOf(entry.getKey()), entry.getValue());
}
boolean existing = tempMap.containsKey(1);
```
当然还有另外一种解决方案，就是用明确了数据类型的类(即下面的<code>AjaxResult</code>)去接收反序列化的结果，代码如下，这个时候jackson知道了map key的类型，直接调用了<code>IntKD</code>的<code>_parse</code>，将字符串转化为Integer之后返回。
```
public class AjaxResult {
    private boolean ret;
    private int errcode;
    private Map<Integer, String> data;

    public boolean isRet() {
        return ret;
    }

    public AjaxResult setRet(boolean ret) {
        this.ret = ret;
        return this;
    }

    public int getErrcode() {
        return errcode;
    }

    public AjaxResult setErrcode(int errcode) {
        this.errcode = errcode;
        return this;
    }

    public Map<Integer, String> getData() {
        return data;
    }

    public AjaxResult setData(Map<Integer, String> data) {
        this.data = data;
        return this;
    }
}
```

```
AjaxResult ajaxResult = objectMapper.readValue(retStr, new TypeReference<AjaxResult>() {});
Map<Integer, String> ajaxDataMap = ajaxResult.getData();
dataNodeMap.containsKey(1);
```
