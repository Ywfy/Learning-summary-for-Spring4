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
