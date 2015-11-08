title: '@transactional'
date: 2015-11-08 11:36:44
tags: [transactional, spring, java]
---
在<code>＠Transactional</code>注解的程序块中调用未使用<code>＠Transactional</code>注解的方法，则事务将在未被注解的方法中继续生效。在未被注解的方法中，继续使用事务中的数据库连接，该方法中的任何异常都会引起事务的回滚。
在<code>＠Transactional</code>注解的程序块中调用使用<code>＠Transactional</code>注解的方法，如果这两个方法属于同一个实例，那么事务在被调用的方法中不会生效；如果两个方法属于不同的实例，则事务在被调用的方法中继续生效。
同理，在一个实例内部，通过未被<code>＠Transactional</code>注解的方法调用被<code>＠Transactional</code>注解的方法，被调用方法的事务也不会生效。
之所以会出现以上现象，是受到Spring AOP机制的限制。<code>＠Transactional</code>注解使用的是Spring的AOP机制，AOP仅在外部实例调用被注解的方法时才会生效，在同一个实例中调用任何被注解的方法都只能调用到原生方法，而不是注解后生成的代理方法。
如果不得不在一个实例内部调用被<code>＠Transactional</code>注解的方法，需要在这个实例内部显示的注入自身类的实例，并通过这个被注入的实例调用被注解的方法，才能访问到代理方法使事务生效。示例如下：
```
@Service("transactionalTestClass")
public class TransactionalTestClass {
    
    @Resource("transactionalTestClass")
    private TransactionalTestClass transactionalTestClass;
    
    public void noTransactionalMethod(){
        transactionalTestClass.transactionalMethod();
    }

    @Transactional(rollbackFor = Exception.class)
    public void transactionalMethod() {
        //do somthing
    }
}
```