# AOP
## 在讲解AOP之前，先用一个例子来回顾一下动态代理的知识
就用一个简单的加减乘除计算器吧<br>
先确定计算器的接口:
```
package com.atguigu.spring.aop.helloWorld;

public interface ArithmeticCalculator {
	
	int add(int i, int j);
	int sub(int i, int j);
	
	int mul(int i, int j);
	int div(int i, int j);
}
```

接下来完成这个计算器的实现:
```
package com.atguigu.spring.aop.helloWorld;

public class ArithmeticCalculatorImpl implements ArithmeticCalculator {

	@Override
	public int add(int i, int j) {
		int result = i + j;
		return result;
	}

	@Override
	public int sub(int i, int j) {
		int result = i - j;
		return result;
	}

	@Override
	public int mul(int i, int j) {
		int result = i * j;
		return result;
	}

	@Override
	public int div(int i, int j) {	
		int result = i / j;	
		return result;
	}

}
```

我们希望每个计算执行前后都有日志记录：
```
package com.atguigu.spring.aop.helloWorld;

public class ArithmeticCalculatorLoggingImpl implements ArithmeticCalculator {

	@Override
	public int add(int i, int j) {
		System.out.println("The method add begins with["+ i + "," + j + "]");
		int result = i + j;
		System.out.println("The method add ends with " + result);
		return result;
	}

	@Override
	public int sub(int i, int j) {
		System.out.println("The method sub begins with["+ i + "," + j + "]");
		int result = i - j;
		System.out.println("The method sub ends with " + result);
		return result;
	}

	@Override
	public int mul(int i, int j) {
		System.out.println("The method mul begins with["+ i + "," + j + "]");
		int result = i * j;
		System.out.println("The method mul ends with " + result);
		return result;
	}

	@Override
	public int div(int i, int j) {
		System.out.println("The method div begins with["+ i + "," + j + "]");
		int result = i / j;
		System.out.println("The method div ends with " + result);
		return result;
	}

}
```

这时我们发现这么写实在是太累了，我们应该有更巧妙的方式————动态代理<br>
```
package com.atguigu.spring.aop.helloWorld;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.util.Arrays;

public class ArithmeticCalculatorLoggingProxy {
	
	//要代理的对象
	private ArithmeticCalculator target;
	
	public ArithmeticCalculatorLoggingProxy(ArithmeticCalculator target) {
		this.target = target;
	}
	
	public ArithmeticCalculator getLoggingProxy() {
		ArithmeticCalculator proxy = null;
		
		//代理对象由哪一个类加载器负责加载
		ClassLoader loader = target.getClass().getClassLoader();
		//代理对象的类型，即其中有哪些方法
		Class [] interfaces = new Class[]{ArithmeticCalculator.class};
		//当调用代理对象其中的方法时，该执行的代码
		InvocationHandler h = new InvocationHandler() {
			/**
			 * proxy:正在返回的那个代理对象，一般情况下，在invoke方法中都不使用该对象，
			 * method：正在被调用的方法
			 * args：调用方法时传入的参数
			 */
			@Override
			public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
				String methodName = method.getName();
				//日志 
				System.out.println("[before]The method " + methodName + " begin with " + Arrays.asList(args));
				
        //执行方法
				Object result = null;
				try {
					result = method.invoke(target, args);
				}catch(NullPointerException e){
				
				}
			
				//日志
				System.out.println("[after]The method " + methodName + " ends with " + result);
				return result;
			}
		};
		proxy = (ArithmeticCalculator)Proxy.newProxyInstance(loader, interfaces, h);	
		return proxy;
	}
}
```

测试代码:
```
package com.atguigu.spring.aop.helloWorld;

public class Main {

	public static void main(String[] args) {
		// TODO Auto-generated method stub
	//	ArithmeticCalculator atithmeticCalculator = null;
	//	atithmeticCalculator = new ArithmeticCalculatorLoggingImpl();
		
		ArithmeticCalculator target = new ArithmeticCalculatorImpl();
		ArithmeticCalculator proxy = new ArithmeticCalculatorLoggingProxy(target).getLoggingProxy();
		
		
		int result1 = proxy.add(1, 2);
		System.out.println(result1);
		int result2 = proxy.div(4, 2);
		System.out.println(result2);
	}

}
```

输出结果：
```
[before]The method add begin with [1, 2]
[after]The method add ends with 3
3
[before]The method div begin with [4, 2]
[after]The method div ends with 2
2
```
<br>

## AOP
![图片无法加载](https://github.com/Ywfy/Learning-summary-for-Spring4/blob/master/AOP/aop.PNG)<br>
AOP即面向切面编程，将重复的非主要业务逻辑的提取出来，当需要时，插入到需要的位置。
详细的定义有：
* 切面(Aspect):  横切关注点(跨越应用程序多个模块的功能)被模块化的特殊对象
* 通知(Advice):  切面必须要完成的工作
* 目标(Target): 被通知的对象
* 代理(Proxy): 向目标对象应用通知之后创建的对象
* 连接点（Joinpoint）：程序执行的某个特定位置：如类某个方法调用前、调用后、方法抛出异常后等。连接点由两个信息确定：方法表示的程序执行点；相对点表示的方位。
* 切点（pointcut）：每个类都拥有多个连接点：例如 ArithmethicCalculator 的所有方法实际上都是连接点，即连接点是程序类中客观存在的事务。AOP 通过切点定位到特定的连接点。类比：连接点相当于数据库中的记录，切点相当于查询条件
* AspectJ：Java 社区里最完整最流行的 AOP 框架

### SpringAOP配置流程
#### 1、引入jar包
```
spring-beans-4.0.0.RELEASE.jar
spring-context-4.0.0.RELEASE.jar
spring-core-4.0.0.RELEASE.jar
spring-expression-4.0.0.RELEASE.jar
commons-logging-1.1.3.jar

com.springsource.org.aopalliance-1.0.0.jar
com.springsource.org.aspectj.weaver-1.6.8.RELEASE.jar
spring-aop-4.0.0.RELEASE.jar
spring-aspects-4.0.0.RELEASE.jar
```

#### 2、在配置文件中加入aop的命名空间
<br>
#### 3、基于注解的方式
##### 1)、在配置文件中加入如下配置：
```
<aop:aspectj-autoproxy></aop:aspectj-autoproxy>
	使AspjectJ 注解起作用:自动为匹配的类生成代理对象
```
##### 2)、创建类，并且把这个类声明为一个切面:需要把该类放入到IOC容器中,再声明为一个切面
```
@Aspect
@Component
public class LoggingAspect {xxx
```

##### 3)、在类中声明各种通知
五种通知:<br>(1)@Before：前置通知，在方法执行之前执行<br>
        (2)@After：后置通知，在方法执行之后执行 <br>
        (3)@AfterRunning：返回通知，在方法返回结果之后执行<br>
        (4)@AfterThrowing：异常通知，在方法抛出异常之后<br>
        (5)@Around：环绕通知，围绕着方法执行<br>
i、声明一个方法<br>
ii、在方法前加入@Before或者其他注解<br>
```
        //声明该方法是一个前置通知:在目标方法开始之前执行
	@Before("execution(public int com.atguigu.spring.aop.impl.ArithmeticCalculator.*(int, int))")
	public void beforeMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinPoint.getArgs());
		System.out.println("The method " + methodName + " begins with " + args);
	}
	
	
	/**
	 * 定义一个方法，用于声明切入点表达式，一般地，该方法中再不需要添入其他的代码
	 * 使用@Pointcut 来声明切入点表达式
	 * 后面的其他通知直接使用方法名来引用当前的切入点表达式
	 * 	包名.类名.方法名	(不同类加类名，不同包加包名)
	 */
	@Pointcut("execution(public int com.atguigu.spring.aop.ArithmeticCalculator.*(..))")
	public void declareJointPointExpression() {}
	
	
	/**
	 * 在方法正常结束受执行的代码
	 * 返回通知是可以访问到方法的返回值的
	 * @param joinPoint
	 */
	@AfterReturning(value = "declareJointPointExpression()"
			,returning="result")
	public void afterReturning(JoinPoint joinPoint, Object result) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " ends with " + result);
	}
	
	
	//后置通知：在目标方法执行后(无论是否发生异常)，执行的通知。
	//在后置通知中还不能访问目标方法执行的结果
	@After("execution(* com.atguigu.spring.aop.impl.*.*(int, int))")
	public void afterMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " ends");
	}
	
	
	/**
	 * 在目标方法出现异常时会执行的代码，可以访问到异常对象；且可以指定在出现特定异常时再执行通知代码
	 * afterThrowing(JoinPoint joinPoint, NullPointerException ex)
	 * 例如将参数Exception ex 改为NullPointerException ex的话就只有在发生NullPointerException时才执行代码
	 */
	@AfterThrowing(value = "declareJointPointExpression()"
			,throwing = "ex")
	public void afterThrowing(JoinPoint joinPoint, Exception ex) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " occurs exception: " + ex);
	}
	
	
	/**
	 * 环绕通知需要携带 ProceedingJoinPoint 类型的参数，
	 * 环绕通知类似于动态代理的全过程： ProceedingJoinPoint 类型的参数可以决定是否执行目标方法，
	 * 且环绕通知必须有返回值，且返回值即为目标方法的返回值
	 */
	@Around("execution(public int com.atguigu.spring.aop.ArithmeticCalculator.*(..))")
	public Object aroundMethod(ProceedingJoinPoint pjd) {
		
		Object result = null;
		String methodName = pjd.getSignature().getName();
		Object[] args = pjd.getArgs();
		
		//执行目标方法
		try {
			//前置通知
			System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
			//执行目标方法
			result = pjd.proceed();
			//返回通知
			System.out.println("The method " + methodName + " ends with " + result);
		} catch (Throwable e) {
			//异常通知
			System.out.println("The method " + methodName + " occurs exeception: " + e);
			throw new RuntimeException(e);
		}
		//后置通知
		System.out.println("The method " + methodName + " ends");
		
		return result;
	}
 
 /*
   *@Before()内的字符串是切点表达式，表示执行 ArithmeticCalculator 
   *接口的 add() 方法. * 代表匹配任意修饰符及任意返回值, 参数列表中的 .. 
   *匹配任意数量的参数
   *切入点表达式可以通过操作符 &&, ||, ! 结合起来. 
   *
   */

```
实际上跟动态代理相对应有如下:
```
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
	// TODO Auto-generated method stub
	String methodName = method.getName();
	//日志 
	System.out.println("[before]The method " + methodName + " begin with " + Arrays.asList(args));
	//执行方法
	Object result = null;
	try {
		//前置通知
		result = method.invoke(target, args);
		//返回通知，可以访问到方法的返回值
	}catch(NullPointerException e){
		//异常通知,可以访问到方法出现的异常
	}
	//后置通知，因为方法可能会出异常，所以访问不到方法的返回值
				
	//日志
	System.out.println("[after]The method " + methodName + " ends with " + result);
	return result;
}
```

##### 4)、可以在通知方法中声明一个类型为JoinPoint的参数，然后就能访问链接细节，如方法名称和参数值。
```
@Aspect
@Component
public class LoggingAspect {
	
	@Before("execution(public int com.atguigu.spring.aop.impl.ArithmeticCalculator.*(int, int))")
	public void beforeMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		List<Object> args = Arrays.asList(joinPoint.getArgs());
		System.out.println("The method " + methodName + " begins with " + args);
	}
```

##### 5)、在同一个连接点上应用不止一个切面时,则存在优先级问题
此时可以使用@Order()来标明优先级,括号内数字越小优先级越高<br>
```
@Order(2)
@Aspect
@Component
public class LoggingAspect {...

@Order(1)
@Aspect
@Component
public class ValidationAspect {
	
	@Before("com.atguigu.spring.aop.LoggingAspect.declareJointPointExpression()")
	public void validateArgs(JoinPoint joinPoint){
		System.out.println("validate " + Arrays.asList(joinPoint.getArgs()));
	}
}

```
<br>

#### 4、基于XML的配置声明切面
注：基于注解的声明要优先于基于 XML 的声明，通过 AspectJ 注解, 切面可以与 AspectJ 兼容, 而基于 XML 的配置则是 Spring 专有的<br>
##### 1)、创建想要成为切面的类
LoggingAspect:
```
package com.atguigu.spring.aop.xml;

import java.util.Arrays;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.ProceedingJoinPoint;


public class LoggingAspect {
	
	public void beforeMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		Object[] args = joinPoint.getArgs();
		System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
	}
	
	public void afterReturning(JoinPoint joinPoint, Object result) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " ends with " + result);
	}
	
	public void afterMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " ends");
	}
		
	public void afterThrowing(JoinPoint joinPoint, Exception ex) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " occurs exception: " + ex);
	}
		
		
	public Object aroundMethod(ProceedingJoinPoint pjd) {
		
		Object result = null;
		String methodName = pjd.getSignature().getName();
		Object[] args = pjd.getArgs();
		
		//执行目标方法
		try {
			//前置通知
			System.out.println("The method " + methodName + " begins with " + Arrays.asList(args));
			//执行目标方法
			result = pjd.proceed();
			//返回通知
			System.out.println("The method " + methodName + " ends with " + result);
		} catch (Throwable e) {
			//异常通知
			System.out.println("The method " + methodName + " occurs exeception: " + e);
			throw new RuntimeException(e);
		}
		//后置通知
		System.out.println("The method " + methodName + " ends");
		
		return result;
		
	}
}
```
ValidationAspect:
```
ackage com.atguigu.spring.aop.xml;

import java.util.Arrays;

import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.core.annotation.Order;
import org.springframework.stereotype.Component;

public class ValidationAspect {
	
	public void validateArgs(JoinPoint joinPoint){
		System.out.println("validate " + Arrays.asList(joinPoint.getArgs()));
	}
}
```

##### 2)、全局配置
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.0.xsd">

	<!-- 配置bean -->
	<bean id="arithmeticCalculator"
		class="com.atguigu.spring.aop.xml.ArithmeticCalculatorImpl">
	</bean>
		
	<!-- 配置切面的bean -->
	<bean id="loggingAspect"
		class="com.atguigu.spring.aop.xml.LoggingAspect">
	</bean>
	
	<bean id="validationAspect"
		class="com.atguigu.spring.aop.xml.ValidationAspect">
	</bean>
	
	<!-- 配置AOP -->
	<aop:config>
		<!-- 配置切点表达式 -->
		<aop:pointcut expression="execution(* com.atguigu.spring.aop.xml.ArithmeticCalculator.*(int, int))" 
			id="pointcut" />
		<!-- 配置切面及通知 -->
		<aop:aspect ref="loggingAspect" order="2">
			<aop:before method="beforeMethod" pointcut-ref="pointcut"/>
		        <aop:after method="afterMethod" pointcut-ref="pointcut"/>
			<aop:after-returning method="afterReturning" pointcut-ref="pointcut" returning="result"/>
			<aop:after-throwing method="afterThrowing" pointcut-ref="pointcut" throwing="ex"/> 
			<!-- 
			<aop:around method="aroundMethod" pointcut-ref="pointcut"/>
			 -->
		</aop:aspect>
		
		<aop:aspect ref="validationAspect" order="1">
			<aop:before method="validateArgs" pointcut-ref="pointcut"/>
		</aop:aspect>
	</aop:config>
</beans>
```



