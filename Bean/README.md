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

至于获取bean的方式，我们前面就演示过了，通过调用applicationContext对象的getBean()方法<br>
<br>

## 依赖注入的方式：属性注入和构造器注入
### 1、属性注入(开发中最常用的注入方式）
* 属性注入即通过 setter 方法注入Bean 的属性值或依赖的对象
* 属性注入使用<property>标签

```
<bean id="helloWorld" class="com.atguigu.spring.bean.HelloWorld">
    <!-- name属性填Bean对应的属性名，value填注入值 -->
		<property name="name" value="Spring"></property>
	</bean>
```

### 2、构造器注入
* 通过构造方法完成bean属性的注入
* 构造器注入使用<constructor-arg>标签
  
```
<!-- 2、构造器注入 
			使用<constructor-arg>标签
				value：传给构造器的值,也可以作为子标签写在内部
				index：对应构造器的第几个参数（从0开始）
				type:指定参数的类型（用来区分重载构造器）
	-->
	<bean id="car" class="com.atguigu.spring.bean.Car">
		<constructor-arg value="AUDI" index="0"></constructor-arg>
		<constructor-arg value="ShangHai" index="1"></constructor-arg>
		<constructor-arg value="300000" index="2" type="double"></constructor-arg>
	</bean>
```

### 3、注入属性值细节
#### 1)、如果字面值包含特殊字符，可以使用 \<![CDATA[]]> 包裹起来
```
<bean id="car2" class="com.atguigu.spring.bean.Car">
		<constructor-arg value="BaoMa" type="java.lang.String"></constructor-arg>
		<!-- 如果字面值包含特殊字符，可以使用<![CDATA[]]>包裹起来 -->
		<!-- <constructor-arg value="<ShangHai>" type="java.lang.String"></constructor-arg> -->
		<constructor-arg>
			<value><![CDATA[<"shanghai">]]></value>
		</constructor-arg>
		<constructor-arg type="int">
			<value>250</value>
		</constructor-arg>
</bean>
```

#### 2)、引用其他Bean
通过使用ref或者内部bean
```
<bean id="person" class="com.atguigu.spring.bean.Person">
		<property name="name" value="Tom"></property>
		<property name="age" value="24"></property>
		<!-- 可以使用property的ref属性建立bean之间的引用关系 -->
		<!-- <property name="car" ref="car2"></property>  -->
		<!-- 或者ref子标签 -->
		<!-- 
		<property name="car">
			<ref bean="car2"/>
		</property> 
		-->
		<!-- 内部bean,不能被外部引用，只能在内部使用 -->
		<property name="car">
			<bean class="com.atguigu.spring.bean.Car">
				<constructor-arg value="Ford"></constructor-arg>
				<constructor-arg value="ChangAn"></constructor-arg>
				<constructor-arg value="200000" type="double"></constructor-arg>
			</bean>
		</property>
	</bean>
  ```
  
  #### 3)、NULL值和级联属性
  ```
  <bean id="person2" class="com.atguigu.spring.bean.Person">
		<constructor-arg value="Jerry"></constructor-arg>
		<constructor-arg value="25"></constructor-arg>
		<!-- 测试赋值NULL -->
		<!-- <constructor-arg><null/></constructor-arg> -->
		<constructor-arg ref="car"></constructor-arg>
		<!-- 级联属性赋值,注意：属性对象必须先被初始化后，级联属性才可以赋值，否则报错 -->
		<property name="car.maxSpeed" value="300"></property>
	</bean>
  ```
  
  #### 4)、集合属性
  ##### \<list>、\<set>   注：数组也使用\<list>标签

  
