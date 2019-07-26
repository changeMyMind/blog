#### 1.PostConstruct注解

表示程序初始化的时候直接执行被注解修饰的方法 记住方法不能有参数

```java

/**
* The method MUST NOT have any parameters except in the case of
* interceptors in which case it takes an InvocationContext object as
* defined by the Interceptors specification.
*/
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}
```



使用场景：

1. bloom过滤器的内容赋值
2. 某些固定系统的权限列表加载