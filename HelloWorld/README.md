# HelloWorld
### 本教程使用STS 3，大家也可以使用Eclipse安装STS插件，但失败率很高，推荐直接使用STS。STS 3的官方[下载地址](http://spring.io/tools3/sts/all)
#### 1、引入jar包
```
c3p0-0.9.1.2.jar
commons-logging-1.1.3.jar
mysql-connector-java-5.1.37-bin.jar
spring-aop-4.0.0.RELEASE.jar
spring-beans-4.0.0.RELEASE.jar
spring-context-4.0.0.RELEASE.jar
spring-core-4.0.0.RELEASE.jar
spring-expression-4.0.0.RELEASE.jar
```
<br>

#### 2、创建Bean配置文件applicationContext.xml
在src目录下新建一个Spring Bean Configuration File，命名为applicationContext.xml<br>
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

	
</beans>
```
<br>

#### 3、创建HelloWorld类
```
package com.atguigu.spring.bean;

public class HelloWorld {

	public String name;
	
	public HelloWorld() {
		System.out.println("HelloWorld's Constructor...");
	}
	
	public void setName(String name) {
		System.out.println("setName: " + name);
		this.name = name;
	}
	
	public void hello() {
		System.out.println("hello " + name);
	}

}
```
<br>

#### 4、在applicationContext.xml中配置bean
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
  
  <!-- 配置bean -->
	<!-- 
		class：bean的全类名，通过反射的方式在IOC容器中创建bean，所以要求bean中一定要有无参构造器
		id：bean的标识，唯一
	 -->
  <bean id="helloWorld" class="com.atguigu.spring.bean.HelloWorld">
		<property name="name" value="Spring"></property>
	</bean>
	
</beans>
```
<br>

#### 5、编写main方法
```
package com.atguigu.spring.bean;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Main {

	public static void main(String[] args) {
    
    ApplicationContext ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
    HelloWorld helloWorld = ctx.getBean(HelloWorld.class);
    helloWorld.hello();
  }
}
```
<br>

#### 6、输出结果
```
一月 19, 2019 12:09:16 上午 org.springframework.context.support.ClassPathXmlApplicationContext prepareRefresh
信息: Refreshing org.springframework.context.support.ClassPathXmlApplicationContext@6193b845: startup date [Sat Jan 19 00:09:16 CST 2019]; root of context hierarchy
一月 19, 2019 12:09:16 上午 org.springframework.beans.factory.xml.XmlBeanDefinitionReader loadBeanDefinitions
信息: Loading XML bean definitions from class path resource [applicationContext.xml]
HelloWorld's Constructor...
setName: Spring
hello Spring
```
从这个结果我们可以看出一些东西：<br>
在Spring项目运行的时候，先加载applicationContext.xml，这时会自动完成配置文件bean的实例化，并且调动setter方法对属性赋值，<br>
最后才轮到我们获取对象，使用对象。<br>
