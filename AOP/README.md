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
五种通知:(1)@Before：前置通知，在方法执行之前执行<br>
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
	
	//后置通知：在目标方法执行后(无论是否发生异常)，执行的通知。
	//在后置通知中还不能访问目标方法执行的结果
	@After("execution(* com.atguigu.spring.aop.impl.*.*(int, int))")
	public void afterMethod(JoinPoint joinPoint) {
		String methodName = joinPoint.getSignature().getName();
		System.out.println("The method " + methodName + " ends");
	}
 
 /*
   *@Before()内的字符串是切点表达式，表示执行 ArithmeticCalculator 
   *接口的 add() 方法. * 代表匹配任意修饰符及任意返回值, 参数列表中的 .. 
   *匹配任意数量的参数
   *切入点表达式可以通过操作符 &&, ||, ! 结合起来. 
   *
   */

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


