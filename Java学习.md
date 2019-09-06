# Java知识总结

### java自定义注解

注解：

元注解：

注解处理器：

利用注解方式可以实现的场景：











### java工具类库

#### fastjson 

1.反序列化成泛型

~~~java
MsgData<TestBody> data = JSON.parseObject(json,new TypeReference<MsgData<TestBody>>(){});
~~~

TestBody类必须序列化





