# Spring中Bean的配置
## IOC & DI 概述
* IOC(Inversion of Control)：其思想是反转资源获取的方向，传统的方法是new一个对象，然后使用这个对象。而应用了IOC，则是容器主动地将对象创建出来，然后当我们需要的时候，直接伸手向容器要就行而不用去创建对象。<br>
* DI(Dependency Injection)：依赖注入，这是IOC思想的目前最常用的实现方式，即组件以一些预先定义好的方式(例如: setter 方法)接受来自如容器的资源注入。<br><br>

## 配置Bean的形式
配置Bean的形式有两种：基于XML文件的方式；基于注解的方式<br>
这里先说下基于XML文件的方式:<br>
<br>
在Spring的配置文件中通过bean节点来配置bean<br>
```
<!-- id：bean的标识，在IOC容器中必须唯一。 -->
<bean id="helloWorld" class="com.atguigu.spring.bean.HelloWorld">
</bean>
```

从前面的HelloWorld，我们已经看出在使用bean之前，先要加载applicationContext.xml，完成bean的实例化，之后才能使用Bean。<br>
接下来我们就详细说说这里面的门道：<br>
加载applicationContext.xml实际上就是为了IOC容器的创建，此外还有另一种方法：BeanFactory，不过我们使用applicationContext.xml就行了,而日常开发也是使用的applicationContxt.xml<br>
<br>
加载applicationContext.xml的方式有两种：<br>
* ClassPathXmlApplicationContext ：从类路径下加载配置文件<br>
* FileSystemXmlApplicationContext : 从文件系统中加载配置文件<br>

当我们开始了applicationContext的加载后，IOC容器开始创建，并且在IOC容器初始化时就实例化所有单例的bean<br>
<br>

至于获取bean的方式，我们前面就演示过了，通过调用applicationContext对象的getBean()方法
