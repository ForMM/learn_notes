# Java知识总结

### 注解

注解：可以理解为标签，可以在类、方法、属性都可以加上标签。

元注解：

~~~java
@Documented
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
@Inherited
public @interface LogAnnotation {
	String value() default "default";
}

@Documented //此注解包含在javadoc中
@Target  //用在什么地方，ElementType.METHOD
/**
CONSTRUCTOR:构造器声明
FIELD:域声明
LOCAL_VARIABLE:局部变量声明
METHOD:方法声明
PACKAGE:包声明
PARAMETER:参数说明
TYPE:类、接口声明
**/
@Retention //需要在什么级别保存该注解信息
/**
SOURCE:注解将被编译器丢弃
CLASS:注解在class文件中可用，但会被JVM丢弃
RUNTIME:运行期也会保存该注解，通过反射机制可以读取注解信息
**/
@Inherited  //允许子类继承父类的注解
~~~



注解处理器：



利用注解方式可以实现的场景：

1. 校验类
2. 数据库字段处理
3. 日志追踪



### java动态代理

InvocationHandler接口和Proxy类实现动态代理。







### java工具类库

#### fastjson 

1.反序列化成泛型

~~~java
MsgData<TestBody> data = JSON.parseObject(json,new TypeReference<MsgData<TestBody>>(){});
~~~

TestBody类必须序列化



#### Arrays类

~~~
Arrays.sort(T aa);
Arrays.asList();
Arrays.binarySearch(bb, "yy");
Arrays.copyOf(aa, 9);
~~~

#### java源码学习记录

~~~java
java.lang
Object
#native方法
	registerNatives()
  getClass()
  hashCode()
  clone()
equal(Object o)
toString()
notify()
notifyAll()
wait(long timeout)
wait(long timeout,int nanos)
wait()
finalize()

String





~~~





