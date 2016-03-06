title: cast numeric string to Integer in json
tags:
  - java
  - type cast
  - reflection
date: 2016-02-28 22:21:00
---

最近反序列化json的时候，踩到了一个java强制类型转换的坑。踩坑时反序列化的数据格式如下:
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
![cast_numeric_string_to_Integer01.png](./images/cast_numeric_string_to_Integer01.png)
可见造成以上现象的原因是，字符串类型的数字在强转成Integer类型时是以ASCII码的形式转换的。
修改代码如下，我们得到了想要的结果。
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
