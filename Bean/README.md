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
  ##### \<list>、\<set>、\<map>   注：数组也使用\<list>标签
  ```
  <!-- 测试如何配置集合属性 
		三个集合属性:<list>、<set>、<map>
		list、set标签内部可以通过：（注：数组也使用list）
			<value>指定简单的常量值
			<ref>指定对其他bean的引用
			<bean>指定内置Bean定义
			<null/>指定空元素
			内嵌其他集合
	-->
	<bean id="person3" class="com.atguigu.spring.bean.collections.Person">
		<property name="name" value="Mike"></property>
		<property name="age" value="27"></property>
		<property name="cars">
		<!-- 使用list节点为list类型的属性赋值 -->
			<list>
				<ref bean="car"/>
				<ref bean="car2"/>
				<bean class="com.atguigu.spring.bean.Car">
					<constructor-arg value="Ford"></constructor-arg>
					<constructor-arg value="ChangAn"></constructor-arg>
					<constructor-arg value="200000" type="double"></constructor-arg>
				</bean>
			</list>
		</property>
	</bean>
	
	<!-- 配置map属性值 -->
	<bean id="newPerson" class="com.atguigu.spring.bean.collections.NewPerson">
		<property name="name" value="Rose"></property>
		<property name="age" value="28"></property>
		<property name="cars">
			<!-- 使用map节点及map的entry子节点配置Map类型的成员变量 -->
			<map>
				<entry key="AA" value-ref="car"></entry>
				<entry key="BB" value-ref="car2"></entry>
			</map>
		</property>
	</bean>
	
	<!-- 配置Properties属性值 -->
	<bean id="dataSource" class="com.atguigu.spring.bean.collections.DataSource">
		<!-- 使用props和prop子节点来为properties属性赋值 -->
		<property name="properties">
			<props>
				<prop key="user">root</prop>
				<prop key="password">1234</prop>
				<prop key="jdbcUrl">niubi</prop>
				<prop key="driverClass">wiu2i</prop>
			</props>
		</property>
	</bean>
  ```

##### 定义独立可共享的集合bean
```
<!-- 配置单例的集合bean，以供多个bean进行引用  需要导入util命名空间-->
	<util:list id="cars">
		<ref bean="car" />
		<ref bean="car2"/>
	</util:list>
	
	<bean id="person4" class="com.atguigu.spring.bean.collections.Person">
		<property name="name" value="Jack"></property>
		<property name="age" value="29"></property>
		<property name="cars" ref="cars"></property>
	</bean>
```
<br>

## p命名空间
```
<!-- 通过P命名空间为 bean 的属性赋值，需要先导入P命名空间 
			相对于传统的配置方式更加的简洁	
	-->
	<bean id="person5" class="com.atguigu.spring.bean.collections.Person"
		p:age="30" p:name="Queen" p:cars-ref="cars"></bean>
```
<br>

## XML 配置里的 Bean 自动装配
通过<bean>的autowire属性指定自动装配的模式:<br>
	* byType(根据类型自动装配):目标类型必须是单例，即在IOC容器中目标类型只能有一个Bean而不能有多个，否则报错<br>
	* byName(根据名称自动装配):必须将目标 Bean 的名称和属性名设置的完全相同<br>
	* constructor(通过构造器自动装配):不推荐使用<br>
```
<bean id="address" class="com.atguigu.spring.bean.autowire.Address"
	p:city="BeiJing" p:street="HuiLongGuan">
</bean>
<!-- 
<bean id="address2" class="com.atguigu.spring.bean.autowire.Address"
	p:city="DaLian" p:street="ZhongShan" >
</bean> 
-->

<bean id="car" class="com.atguigu.spring.bean.autowire.Car"
	p:brand="AUDI" p:price="300000">
</bean>
		
<bean id="person" class="com.atguigu.spring.bean.autowire.Person"
	p:name="Tom" autowire="byType">
</bean>
```
