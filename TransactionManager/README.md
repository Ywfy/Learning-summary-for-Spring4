# Spring的事务管理
## 事务简介
* 事务管理是企业级应用程序开发中必不可少的技术,  用来确保数据的完整性和一致性. 
* 事务就是一系列的动作, 它们被当做一个单独的工作单元. 这些动作要么全部完成, 要么全部不起作用
* 事务的四个关键属性(ACID):
    * 原子性(atomicity): 事务是一个原子操作, 由一系列动作组成.事务的原子性确保动作要么全部完成要么完全不起作用.
    * 一致性(consistency): 一旦所有事务动作完成, 事务就被提交. 数据和资源就处于一种满足业务规则的一致性状态中.
    * 隔离性(isolation): 可能有许多事务会同时处理相同的数据, 因此每个事物都应该与其他事务隔离开来, 防止数据损坏.
    * 持久性(durability): 一旦事务完成, 无论发生什么系统错误, 它的结果都不应该受到影响. 通常情况下, 事务的结果被写到持久化存储器中.

<br>
Spring的声明式事务管理有两种方式：
* 基于注解
* 基于tx和aop命名空间的xml配置文件

一个简单的示例————图书购买系统：
### 1、配置事务管理器
```
   <!-- 配置事务管理器 -->
	<bean id="transactionManager" 
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
```

### 基于注解的方式

### 2、启动事务注解（注意引入tx命名空间）
```
   <!-- 启动事务注解 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
```
以下是完整的applicationContext.xml:
```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:tx="http://www.springframework.org/schema/tx"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context-4.0.xsd
		http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx-4.0.xsd">

	<context:component-scan base-package="com.atguigu.spring"></context:component-scan>

	<!-- 导入资源文件 -->
	<context:property-placeholder location="classpath:db.properties"></context:property-placeholder>

	<!-- 配置C3P0数据源 -->
	<bean id="dataSource"
		class="com.mchange.v2.c3p0.ComboPooledDataSource">
		<property name="user" value="${jdbc.user}"></property>
		<property name="password" value="${jdbc.password}"></property>
		<property name="jdbcUrl" value="${jdbc.jdbcUrl}"></property>
		<property name="driverClass" value="${jdbc.driverClass}"></property>
		
		<property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
		<property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
	</bean>
	
	<!-- 配置Spring的JdbcTemplate -->
	<bean id="jdbcTemplate"
		class="org.springframework.jdbc.core.JdbcTemplate">
		<property name="dataSource" ref="dataSource"></property>	
	</bean>
	
	<!-- 配置NamedParameterJdbcTemplate,该对象可以使用具名参数，其没有无参构造器，所以必须为其构造器指定参数 -->
	<bean id="namedParameterJdbcTemplate"
		class="org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate">
		<constructor-arg ref="dataSource"></constructor-arg>
	</bean>
	
	<!-- 
		Spring声明式事务配置：
		1、配置事务管理器
		2、启动事务注解（注意引入tx命名空间）
		3、在需要的方法上加@Transactional注解
	 -->
	<!-- 配置事务管理器 -->
	<bean id="transactionManager" 
		class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
		<property name="dataSource" ref="dataSource"></property>
	</bean>
	
	<!-- 启动事务注解 -->
	<tx:annotation-driven transaction-manager="transactionManager"/>
</beans>
```

### 3、创建类，并在需要的方法上加@Transactional注解
#### 1)、创建Dao
#### BookShopDao：
```
package com.atguigu.spring.tx;

public interface BookShopDao {
	
	//根据书号获取书的单价
	public int findBookPriceByIsdn(String isbn);
	
	//更新书的库存,使书号对应的库存-1
	public void updateBookStock(String isbn);
	
	//更新用户的账户余额：使username的balance-price
	public void updateUserAccount(String username, int price);
}

```

#### BookShopDaoImpl:
```
package com.atguigu.spring.tx;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.stereotype.Repository;

@Repository("BookShopDao")
public class BookShopDaoImpl implements BookShopDao {

	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	@Override
	public int findBookPriceByIsdn(String isbn) {
		// TODO Auto-generated method stub
		String sql = "SELECT price FROM book WHERE isbn = ?";
		return jdbcTemplate.queryForObject(sql, Integer.class, isbn);
	}

	@Override
	public void updateBookStock(String isbn) {
		// 检查书的库存是否足够，若不够，则抛出异常
		String sql2 = "SELECT stock FROM book_stock WHERE isbn = ?";
		int stock = jdbcTemplate.queryForObject(sql2, Integer.class, isbn);
		if(stock == 0) {
			throw new BookStockException("库存不足！");
		}
		String sql = "UPDATE book_stock SET stock = stock - 1 WHERE isbn = ?";
		jdbcTemplate.update(sql, isbn);
	}

	@Override
	public void updateUserAccount(String username, int price) {
		// 验证余额是否不足，若不足，则抛出异常
		String sql2 = "SELECT balance FROM account WHERE username = ?";
		int balance = jdbcTemplate.queryForObject(sql2, Integer.class, username);
		if(balance < price) {
			throw new UserAccountException("余额不足！");
		}
		
		String sql = "UPDATE account SET balance = balance - ? WHERE username = ?";
		jdbcTemplate.update(sql, price, username);
	}

}
```
这里自定义了两个运行时异常，分别叫 BookStockException 和 UserAccountException，只是简单地继承自RuntimeException，没有做任何修改
<br>

#### 2)、创建Service
#### BookShopService:
```
package com.atguigu.spring.tx;

public interface BookShopService {
	
	public void purchase(String username, String isbn);
}
```
#### BookShopServiceImpl:
```
package com.atguigu.spring.tx;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Isolation;
import org.springframework.transaction.annotation.Propagation;
import org.springframework.transaction.annotation.Transactional;

@Service("bookShopService")
public class BookShopServiceImpl implements BookShopService {

	@Autowired
	private BookShopDao bookShopDao;
	
	/**
	 * 添加事务注解@Transactional
	 * 
	 * 使用propagation 指定事务的传播行为，即当前的事务方法被另外一个事务方法调用时
	 * 如何使用事务，默认取值为REQUIRED，即使用调用方法的事务。
	 * 一个事务内可以包含有很多个小事务，
	 * REQUIRED，则必须所有小事务都能够成功完成，即大事务能够没有一丝错误，最后才确定数据库修改的保存
	 * REQUIRES_NEW，则每完成一个小事务就保存一次，后面的事务发生错误不会影响前面保存的结果
	 *
	 *使用isolation 指定事务的隔离级别，最常用的取值为READ_COMMITTED
	 *
	 *默认情况下Spring的声明式事务对所有的运行时异常进行回滚，也可以通过对应的
	 *属性进行设置，通常情况下取默认值即可。
	 *
	 *使用readOnly 指定事务是否为只读，表示这个事务只读取数据但不更新数据，这样
	 *帮助数据库引擎优化事务，若真的是一个只读取数据库值得方法，应设置readOnly=true
	 *
	 *使用timeout 指定强制回滚之前事务可以占用的时间
	 *@Transactional(propagation=Propagation.REQUIRES_NEW,
			isolation=Isolation.READ_COMMITTED,
			noRollbackFor= {UserAccountException.class})
	 */
	@Transactional(propagation=Propagation.REQUIRES_NEW,
			isolation=Isolation.READ_COMMITTED,
			readOnly=false,
			timeout=3)
	@Override
	public void purchase(String username, String isbn) {
		
		try {
			Thread.sleep(3000);
		} catch (InterruptedException e) {
			
		}
		
		//1、获取书的单价
		int price = bookShopDao.findBookPriceByIsdn(isbn);
		//2、更新书的库存
		bookShopDao.updateBookStock(isbn);
		//3、更新用户余额
		bookShopDao.updateUserAccount(username, price);
	}

}
```
<br>

#### 3)、创建Cashier
#### Cashier:
```
package com.atguigu.spring.tx;

import java.util.List;

public interface Cashier {
	
	public void checkout(String username, List<String> isbns);
}
```

#### CashierImpl:
```
package com.atguigu.spring.tx;

import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service("cashier")
public class CashierImpl implements Cashier {

	@Autowired
	private BookShopService bookShopService;
	
	@Transactional
	@Override
	public void checkout(String username, List<String> isbns) {
		for(String isbn:isbns) {
			bookShopService.purchase(username, isbn);
		}
	}

}

```
<br>

### 测试代码
```
package com.atguigu.spring.tx;

import static org.junit.jupiter.api.Assertions.*;

import java.util.Arrays;

import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

class SpringTransactionTest {

	private ApplicationContext ctx = null;
	private BookShopDao bookShopDao = null;
	private BookShopService bookShopService = null;
	private Cashier cashier = null;
	
	{
		ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		bookShopDao = ctx.getBean(BookShopDao.class);
		bookShopService = (BookShopService)ctx.getBean("bookShopService");
		cashier = ctx.getBean(Cashier.class);
	}
	
	@Test
	public void TestTransactionalPropagation() {
		cashier.checkout("AA", Arrays.asList("1001","1002"));
	}
	
	@Test
	public void testBookShopService() {
		bookShopService.purchase("AA", "1001");
	}
	
	@Test
	public void testBookShopDaoUpdateUserAccount() {
		bookShopDao.updateUserAccount("AA", 100);
	}
	
	@Test
	public void testBookShopDaoUpdateBookStock() {
		bookShopDao.updateBookStock("1001");
	}
	
	@Test
	void testBookShopDaoFindPriceByIsbn() {
		System.out.println(bookShopDao.findBookPriceByIsdn("1001"));
	}

}
```
<br>

### 
