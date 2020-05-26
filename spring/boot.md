

- spring中Constructor、@Autowired、@PostConstruct的顺序

```java
@Service
public class ArrangeServiceImpl implements ArrangeService {

    @Autowired(required = false)
    private IdGenerator idGenerator;
  
    public ArrangeServiceImpl() {
    }

    @PostConstruct
    private void initUuid() {
        if (idGenerator == null) {
            idGenerator = new StrongUuidGenerator();
        }
    }
}
```

> Constructor >> @Autowired >> @PostConstruct，上面可以实现，没有IdGenerator手动创建一个

- @PostConstruct和@PreConstruct

   被@PostConstruct修饰的方法会在服务器加载Servlet的时候运行，并且只会被服务器调用一次，类似于Serclet的inti()方法。被@PostConstruct修饰的方法会在构造函数之后，init()方法之前运行。

   被@PreDestroy修饰的方法会在服务器卸载Servlet的时候运行，并且只会被服务器调用一次，类似于Servlet的destroy()方法。被@PreDestroy修饰的方法会在destroy()方法之后运行，在Servlet被彻底卸载之前。



## 事务

```java
public class Foo {
    @Transactional
    public void bar() { /* … */ }
    public void baz() {
        this.bar();
    }
}
```

在调用 **baz()**方法时，不会开启事务，简单的解决方法是，**baz()**也添加事务注解。