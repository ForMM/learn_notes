# Java知识总结

### java自定义注解











### java工具类库

#### fastjson 

1.反序列化成泛型

~~~java
MsgData<TestBody> data = JSON.parseObject(json,new TypeReference<MsgData<TestBody>>(){});
~~~







