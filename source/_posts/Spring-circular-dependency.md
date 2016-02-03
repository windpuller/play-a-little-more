title: Spring circular dependency
date: 2015-10-25 14:11:22
tags: [circular dependency, spring, java, inherited]
---
最近踩了一个继承加循环依赖的坑，抽象出的代码如下：
ChildClass:
```
@Service
public class ChildClass extends ParentClass {

    @Resource
    private TestService testService;

    public String play() {
        return testService.getParent().play();
    }
}
```
ParentClass:
```
public class ParentClass {

    private String value = "I am parent class";

    public String play() {
        return value;
    }

    public void setValue(String str) {
        this.value = str;
    }
}
```
TestService:
```
@Service
public class TestService {
    private ParentClass parent;

    @PostConstruct
    public void init() {
        parent = create();
    }

    public ParentClass getParent() {
        return this.parent;
    }

    @SuppressWarnings({ "rawtypes", "unchecked" })
    public ParentClass create() {
        try {
            Class classDefinition = Class.forName("com.springapp.mvc.inherited.ChildClass");
            ParentClass parent = (ParentClass) classDefinition.getConstructor(new Class[0]).newInstance(new Object[0]);
            parent.setValue("I was processed by test service");
            return parent;
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
    }
}
```
ResultController:
```
@Controller
@RequestMapping("/inherited")
public class ResultController {

    @Resource
    private ChildClass childClass;

    @RequestMapping(method = RequestMethod.GET)
    public ModelAndView printWelcome() {
        ModelAndView modelAndView = new ModelAndView("hello");
        modelAndView.addObject("message", childClass.play());
        return modelAndView;
    }
}
```

我们注意到，ChildClass重写了ParentClass的play方法。由于在TestService中使用反射方式生成了ChildClass的实例，并赋值给了ParentClass的实例，此时ParentClass实例并不能访问到ChildClass实例的成员变量testService，且ParentClass实例的play方法已经被ChildClass实例的play方法隐藏，也就是说<code>testService.getParent().play()</code>实际调用的是ChildClass的play()方法，显而易见，此处出现了循环调用。但是由于ParentClass的实例中<code>testService == null</code>，因此程序在进入这一层时会抛出NPE。

之后我们使用注解在TestService中实现成员变量parent的初始化，代码如下：
ParentClass:
```
@Service
public class TestService {
    
    @Resource
    private ParentClass parent;

    @PostConstruct
    public void init() {
        parent.setValue("I was processed by test service");
    }

    public ParentClass getParent() {
        return this.parent;
    }
}
```
此时运行程序，NPE消失了，但是出现栈溢出问题。从<code>＠Resource</code>的注解来看，注解的继承关系是向上的，所有使用注解注入的实例共用资源，即以依赖注入形式实现的ChildClass和ParentClass的实例共享资源。因此，TestService中的成员变量parent可以访问到子类ChildClass实例的成员变量testService，进而形成循环依赖并最终导致栈溢出。
<code>＠Resource</code>部分注解：
```
 * Even though this annotation is not marked Inherited, deployment
 * tools are required to examine all superclasses of any component
 * class to discover all uses of this annotation in all superclasses.
 * All such annotation instances specify resources that are needed
 * by the application component.  Note that this annotation may
 * appear on private fields and methods of superclasses; the container
 * is required to perform injection in these cases as well.
```
