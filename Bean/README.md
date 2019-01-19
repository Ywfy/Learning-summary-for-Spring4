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
在实际项目开发中不建议使用自动装配，明确清晰的配置文档要更加便于维护<br>
<br>

## bean的继承与依赖
### 1、继承Bean配置:
基本上就是对象继承那一套：<br>
* 子Bean继承父Bean的配置，可以进行覆盖
* 若想父Bean只是作为模板而不会被实例化，可以设置<bean>的abstract属性为true
* 父Bean的属性并不是都会被继承，如autowire,abstract等
* 父Bean可以不设置class属性，但此时abstract必须设为true
```
<!-- 抽象bean：bean的abstract属性为true的bean，这样的bean不能被IOC实例化，只能被用来继承配置、
	若某一个bean的class属性没有指定，则该bean必须是一个抽象bean-->
	<bean id="address" p:city="BeiJing" p:street="WuDaoKou" abstract="true">
	</bean>
	
	<!-- bean配置的继承：使用bean的parent属性指定继承哪个bean的配置 -->
	<bean id="address2" class="com.atguigu.spring.bean.autowire.Address"
	 parent="address">
	</bean>
	
	<bean id="address3" class="com.atguigu.spring.bean.autowire.Address"
	parent="address2" p:street="DaZhongSi">
	</bean>
	
	<bean id="car" class="com.atguigu.spring.bean.autowire.Car"
		p:brand="AUDI" p:price="300000"></bean>
	
```

### 2、依赖Bean配置：
```
<!-- 要求在配置 person时，必须有一个关联的car，换句话说person这个bean依赖于Car这个bean
		 depends-on:前置依赖，可以通过空格或者逗号分隔多个依赖
		 	前置依赖的Bean会在本Bean实例化之前创建好 -->
	<bean id="person" class="com.atguigu.spring.bean.autowire.Person"
	p:name="Tom" p:address-ref="address2" depends-on="car">
	</bean>
```
<br>

## bean的作用域
```
	<!-- 
	使用bean的scope属性来配置bean的作用域
		singleton：默认，容器初始时创建bean实例，在整个容器的生命周期内只创建这一个bean，单例
		prototype:原型的，容器初始化时不创建bean的实例，而在每次请求时都创建一个新的Bean实例，并返回。
	 -->
	<bean id="car" class="com.atguigu.spring.bean.autowire.Car"
	scope="prototype">
		<property name="brand" value="AUDI"></property>
		<property name="price" value="300000"></property>
	</bean>
```
<br>

## 外部属性文件
注:要引入context命名空间
```
	<!-- 导入属性文件 -->
	<context:property-placeholder location="classpath:db.properties"/>
	<bean id="DataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<!-- 使用外部属性文件的属性 -->
		<property name="user" value="${user}"></property>
		<property name="password" value="${password}"></property>
		<property name="driverClass" value="${driverclass}"></property>
		<property name="jdbcUrl" value="${jdbcurl}"></property>
	</bean>
```
db.properties:<br>
```
user=root
password=aA4405821985%
driverclass=com.mysql.jdbc.Driver
jdbcurl=jdbc:mysql:///test
```
<br>

## SpEL
* 支持运行时查询和操作对象图的强大的表达式语言。
* 使用#{}作为定界符
```
	<bean id="address" class="com.atguigu.spring.bean.spel.Address">
		<!-- 使用spel为属性赋一个字面值 -->
		<property name="city" value="#{'BeiJing'}"></property>
		<property name="street" value="WuDaoKou"></property>
	</bean>
	
	<bean id="car" class="com.atguigu.spring.bean.spel.Car">
		<property name="brand" value="AUDI"></property>
		<property name="price" value="500000"></property>
		<!-- 使用SpEL引用类的静态属性 -->
		<property name="tyrePerimeter" value="#{T(java.lang.Math).PI*80}"></property>
	</bean>
	
	<bean id="person" class="com.atguigu.spring.bean.spel.Person">
		<property name="name" value="Tpm"></property>
		<!-- <property name="car" ref="car"></property> -->
		<!-- 使用SpEL 来引用其他的Bean -->
		<property name="car" value="#{car}"></property>
		<!-- 使用SpEL 来应用其他的Bean的属性 -->
		<property name="city" value="#{address.city}"></property>
		<!-- 在SpEL 中使用运算符 -->
		<property name="info" value="#{car.price>300000?'金领':'白领'}"></property>
	</bean>
```
<br>

## IOC容器中Bean的生命周期
Spring IOC容器可以管理Bean的生命周期，过程为：
* 通过构造器或工厂方法创建Bean实例
* 为Bean的属性设置值和对其他Bean的引用
* 调用Bean的初始化方法
* Bean可以被使用了
* 当容器关闭时，调用Bean的销毁方法
Bean的初始化方法通过init-method属性指定，销毁方法通过destroy-method指定<br>
```
<bean id="car" class="com.atguigu.spring.bean.cycle.Car"
	init-method="init" destroy-method="destroy">
		<property name="brand" value="AUDI"></property>
</bean>
```
以下是car的定义：<br>
```
public class Car {
	
	private String brand;
	
	public Car() {
		// TODO Auto-generated constructor stub
	}
	
	public void setBrand(String brand) {
		System.out.println("SetBrand...");
		this.brand = brand;
	}
	
	public void init() {
		System.out.println("init...");
	}
	
	public void destroy() {
		System.out.println("destroy...");
	}

	@Override
	public String toString() {
		return "Car [brand=" + brand + "]";
	}
	
}
```

实际上，我们可以创建"Bean后置处理器"在调用初始化方法前后对 Bean 进行额外的处理<br>
### Bean后置处理器创建流程:
#### 1、创建后置处理器，实现BeanPostProcessor接口
```
public class MyBeanPostProcessor implements BeanPostProcessor {

	/**
	 *  bean:bean实例本身
	 *  beanName：IOC容器配置的bean的名字 
	 *  返回值：是实际上返回给用户的那个Bean，注意：可以在以上两个方法中修改返回的Bean，甚至返回一个新的Bean
	 *  注意:后置处理器是处理所有Bean的
	 */
	
	//init-method之后被调用
	@Override
	public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessAfterInitialization: " + bean + ", " + beanName);
		return bean;
	}

	//init-method之前被调用
	@Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		// TODO Auto-generated method stub
		System.out.println("postProcessBeforeInitialization: " + bean + ", " + beanName);
		Car car = new Car();
		car.setBrand("Ford");
		return car;
	}

}
```

#### 2、在配置文件中配置
```
<!-- 不需要配置id，IOC容器自动识别是一个BeanPostProcessor -->
<bean class="com.atguigu.spring.bean.cycle.MyBeanPostProcessor"></bean>
```

添加Bean后置处理器后，Bean的生命周期变为如下：
* 通过构造器或工厂方法创建Bean实例
* 为Bean的属性设置值和对其他Bean的引用
* 后置处理器before处理
* 调用Bean的初始化方法(init-method)
* 后置处理器after处理
* Bean可以被使用了
* 当容器关闭时，调用Bean的销毁方法(destroy-method)
<br>

## 工厂方法创建Bean
### 1、静态工厂方法:直接调用某一个类的静态方法就可以返回Bean的实例
##### 1)、创建静态工厂
```
public class StaticCarFactory {
	
	private static Map<String,Car> cars = new HashMap<String,Car>();
	
	static {
		cars.put("AUDI", new Car("AUDI",300000));
		cars.put("Ford", new Car("Ford",400000));
	}
	
	//静态工厂方法
	public static Car getCar(String name) {
		return cars.get(name);
	}
}
```
##### 2)、在配置文件中配置静态工厂
```
	<!-- 通过静态工厂方法来配置bean，注意不是配置静态工厂方法实例，而是配置bean实例 
		  class对应factory的全类名 
		  factory-method对应工厂方法名
		      工厂方法的参数放在  <constructor-arg>子标签中
		  getBean使用工厂的ID
	-->
	<bean id="car1" 
		class="com.atguigu.spring.bean.factory.StaticCarFactory"
		factory-method="getCar">
		<constructor-arg value="AUDI"></constructor-arg>
	</bean>
```

### 2、实例工厂方法:先需要创建工厂实例，再调用工厂的实例方法来返回Bean的实例
##### 1)、创建实例工厂
```
public class InstanceCarFactory {
	
	private Map<String,Car> cars = null;
	
	public InstanceCarFactory() {
		cars = new HashMap<String,Car>();
		cars.put("AUDI", new Car("AUDI",300000));
		cars.put("Ford", new Car("Ford",400000));
	}
	public Car getCar(String brand) {
		return cars.get(brand);
	}
}
```
##### 2)、在配置文件中配置实例工厂
```
	<!--通过实例工厂方法来配置bean 
		factory-bean属性:指向实例工厂的bean
		factory-method对应工厂方法名
		工厂方法的参数放在  <constructor-arg>子标签中
	 -->
	<!-- 配置工厂的实例 -->
	<bean id="carFactory" class="com.atguigu.spring.bean.factory.InstanceCarFactory">
	</bean>
	<!-- 通过实例工厂方法来配置bean -->
	<bean id="car2" factory-bean="carFactory" factory-method="getCar">
		<constructor-arg value="Ford"></constructor-arg>
	</bean>
```
<br>

## FactoryBean方法创建Bean
* Spring 中有两种类型的 Bean, 一种是普通Bean, 另一种是工厂Bean, 即FactoryBean. 
* 工厂 Bean 跟普通Bean不同, 其返回的对象不是指定类的一个实例, 其返回的是该工厂 Bean 的 getObject 方法所返回的对象 
### FactoryBean方法流程:
##### 1、实现FactoryBean接口
```
package com.atguigu.spring.bean.factorybean;

import org.springframework.beans.factory.FactoryBean;

//自定义的FactoryBean需要实现FactoryBean接口
public class CarFactoryBean implements FactoryBean<Car> {
	
	private String brand;
	
	public void setBrand(String brand) {
		this.brand = brand;
	}
	//返回bean的对象
	@Override
	public Car getObject() throws Exception {
		// TODO Auto-generated method stub
		return new Car("BMW", 500000);
	}

	//返回的bean的类型
	@Override
	public Class<?> getObjectType() {
		// TODO Auto-generated method stub
		return Car.class;
	}

	//返回的实例是否为单例
	@Override
	public boolean isSingleton() {
		// TODO Auto-generated method stub
		return true;
	}
		
}

```
##### 2、全局配置文件配置
```
	<!-- 
		通过FactoryBean来配置Bean的实例
		class：指向FactoryBean的全类名
		property：配置FactoryBean的属性
		
		但实际返回的实例却是FactoryBean的getObject()方法返回的实例
	 -->
	<bean id="car" class="com.atguigu.spring.bean.factorybean.CarFactoryBean">
		<property name="brand" value="BMW"></property>
	</bean>
```
<br>

## 基于注解的方式配置Bean
* 组件扫描(component scanning):  Spring 能够从 classpath 下自动扫描, 侦测和实例化具有特定注解的组件。
* 特定组件包括：
	* @Component:基本注解，标识了一个受Spring管理的组件
	* @Respository:标识持久层组件
	* @Service:标识服务层(业务层)组件
	* @Controller:标识表现层组件
* 对于扫描到的组件, Spring 有默认的命名策略: 使用非限定类名, 第一个字母小写. 也可以在注解中通过 value 属性值标识组件的名称
* 当在组件类上使用了特定的注解之后, 还需要在 Spring 的配置文件中声明 <context:component-scan> ：
```
	<!-- 指定Spring IOC 容器扫描的包,将会扫描这个包及其子包 -->
	<!-- 可以通过resource-pattern指定扫描资源，只扫描特定的类 --> 
	<context:component-scan 
		base-package="com.atguigu.spring.bean.annotation"
		resource-pattern="repository/*.class">
	</context:component-scan>
	
```
* 包含扫描和排除扫描
```
<!-- <context:exclude-filter> 子节点指定排除哪些指定表达式的组件  -->
<!-- <context:include-filter> 子节点指定包含哪些表达式的组件，该子节点需要use-default-filters=false配合使用-->
<context:component-scan 
	base-package="com.atguigu.spring.bean.annotation"
	use-default-filters="true">
		<!-- annotation:过滤目标是所有标注了指定注解的类 -->
		<!-- 
		<context:exclude-filter type="annotation" 
			expression="org.springframework.stereotype.Repository"/> 
		-->
		<!--
		<context:include-filter type="annotation" 
			expression="org.springframework.stereotype.Repository"/>
		-->
		
		<!-- assignable:过滤目标是所有继承（实现）了指定类（接口）的类 -->
		<!-- 
		<context:exclude-filter type="assignable" 
			expression="com.atguigu.spring.bean.annotation.repository.UserRepository"/>
		-->
		<!-- 
		<context:include-filter type="assignable" 
			expression="com.atguigu.spring.bean.annotation.repository.UserRepository"/> 
		-->
</context:component-scan>
```
* 使用@Autowired自动装配属性bean
```
@Controller
public class UserController {
	
	@Autowired
	private UserService userService;
	
	public void execute() {
		System.out.println("UserController execute...");
		userService.add();
	}
}
```
默认情况下，当IOC容器中存在多个类型兼容的Bean时，即继承自同一类或引用了同一接口，@Autowired将无法工作<br>
解决方法：<br>
1、@Repository("userRepository"),给想要正确装配的类进行如左操作，括号内为要自动装配的属性的非限定名<br>
	 				spring里的非限定名为首字母小写，其他不变。<br>
2、使用@Qualifier("userRepositoryImpl")，直接指定<br>
```
@Service
public class UserService {
	
	private UserRepository userRepository;
	//此处将@Autowired写在方法上也是可以的，将自动装配传参
	@Autowired
	public void setUserRepository(@Qualifier("userRepositoryImpl")UserRepository userRepository) {
		this.userRepository = userRepository;
	}
	
	public void add() {
		System.out.println("UserService add...");
		userRepository.save();
	}
}
```

对于自动装配链，比如 UserControlle===>UserService===>UserRepository,若其中一环出现了无法装配的情况，比如存在多个类型兼容的Bean时，默认情况下是抛出异常。<br>
我们通过以下操作可以改变这个情况，使得允许某环可以不强制装配
```
	//@Autowired(required=false)允许该属性不被设置，就是可以不强制装配，若找不到就算了
	@Autowired(required=false)
	private TestObject testObject;
```
<br>

## 泛型依赖注入
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-Spring4/blob/master/Bean/%E5%9B%BE%E7%89%872.png)<br>
泛型依赖注入：子类之间的依赖关系由其父类泛型以及父类之间的依赖关系来确定，父类的泛型必须为同一类型。<br>
简单点说，就是两个子类之间的依赖关系不需要在子类中去声明，而是在父类中进行了声明，而依赖的纽带就是 泛型类型，必须是相同的父类泛型类型才具有依赖关系。<br>
简单的代码示例：<br>
BaseRepository<T>:
```
package com.atguigu.spring.bean.generic.di;

public class BaseRepository<T> {}
```
	
BaseService<T>:
```
public class BaseService<T> {
	
	//@Autowired加在属性上会被子类继承
	@Autowired
	protected BaseRepository<T> repository;
	
	public void add() {
		System.out.println("add...");
		System.out.println(repository);
	}
}

```

User:
```
public class User{}
```

UserRepository:
```
@Repository
public class UserRepository extends BaseRepository<User>{}
```
	
UserService:
```
@Service
public class UserService extends BaseService<User>{}
```

测试代码：
```
public class Main {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		ApplicationContext ctx = new ClassPathXmlApplicationContext("beans-generic-di.xml");
		
		UserService userService = (UserService)ctx.getBean("userService");
		userService.add();
	}

}
```

结果:
```
add...
com.atguigu.spring.bean.generic.di.UserRepository@3d921e20
```
在这个示例中，明明userService和UserRepository是没有什么关系的，但是因为他们的父类具有依赖关系，并且两个类的父类泛型都为User,所以他们也拥有了这种依赖关系<br>
