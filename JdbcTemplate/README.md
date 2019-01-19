# JdbcTemplate
#### 以一个简单的示例来讲解，以后使用直接套就行,持久层还是推荐用ORM框架
Employee:
```
package com.atguigu.spring.jdbc;

public class Employee {
	
	private Integer id;
	private String lastName;
	private String email;
	
	private Integer deptId;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getLastName() {
		return lastName;
	}
	public void setLastName(String lastName) {
		this.lastName = lastName;
	}
	public String getEmail() {
		return email;
	}
	public void setEmail(String email) {
		this.email = email;
	}
	
	public Integer getDeptId() {
		return deptId;
	}
	public void setDeptId(Integer deptId) {
		this.deptId = deptId;
	}
	@Override
	public String toString() {
		return "Employee [id=" + id + ", lastName=" + lastName + ", email=" + email + ", deptId=" + deptId + "]";
	}

}
```
<br>

Department:
```
package com.atguigu.spring.jdbc;

public class Department {
	
	private Integer id;
	private String name;
	public Integer getId() {
		return id;
	}
	public void setId(Integer id) {
		this.id = id;
	}
	public String getName() {
		return name;
	}
	public void setName(String name) {
		this.name = name;
	}
	@Override
	public String toString() {
		return "Department [id=" + id + ", name=" + name + "]";
	}
	
}
```
<br>

EmployeeDao:
```
ackage com.atguigu.spring.jdbc;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.stereotype.Repository;

@Repository
public class EmployeeDao {
	
	@Autowired
	private JdbcTemplate jdbcTemplate;
	
	public Employee get(Integer id) {
		String sql = "SELECT id, last_name lastName, email FROM employees WHERE id = ?";
		RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);
		Employee employee = jdbcTemplate.queryForObject(sql, rowMapper, 1);
		
		return employee;
	}
}

```
<br>

applicationContext.xml:
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
<br>

测试代码:
```
package com.atguigu.spring.jdbc;

import static org.junit.jupiter.api.Assertions.*;

import java.sql.SQLException;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.sql.DataSource;

import org.junit.jupiter.api.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.jdbc.core.BeanPropertyRowMapper;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.RowMapper;
import org.springframework.jdbc.core.namedparam.BeanPropertySqlParameterSource;
import org.springframework.jdbc.core.namedparam.NamedParameterJdbcTemplate;
import org.springframework.jdbc.core.namedparam.SqlParameterSource;

class JDBCTest {

	private ApplicationContext ctx = null;
	private JdbcTemplate jdbcTemplate;
	private NamedParameterJdbcTemplate namedParameterJdbcTemplate;
	
	{
		ctx = new ClassPathXmlApplicationContext("applicationContext.xml");
		jdbcTemplate = (JdbcTemplate)ctx.getBean("jdbcTemplate");
		namedParameterJdbcTemplate = (NamedParameterJdbcTemplate)ctx.getBean("namedParameterJdbcTemplate");
	}
	
	/**
	 * 使用具名参数时，可以使用update(String sql, SqlParameterSource paramSource)方法进行更新操作
	 * 1、SQL语句中的参数名与类的属性一致
	 * 2、使用SqlParameterSource的BeanPropertySqlParameterSource实现类作为参数
	 */
	@Test
	public void testNamedParameterJdbcTemplate2() {
		String sql = "INSERT INTO employees(last_name, email, dept_id) VALUES(:lastName,:email,:deptId)";
		Employee employee = new Employee();
		employee.setLastName("XYZ");
		employee.setEmail("xyz@sina.com");
		employee.setDeptId(3);
		
		SqlParameterSource paramSource = new BeanPropertySqlParameterSource(employee);
		namedParameterJdbcTemplate.update(sql, paramSource);
	}
	
	/**
	 * 可以为参数起名字，
	 * 1、好处：若有多个参数，则不用再去对应位置，直接对应参数名，便于维护
	 * 2、缺点：较为麻烦
	 */
	@Test
	public void testNamedParameterJdbcTemplate() {
		String sql = "INSERT INTO employees(last_name, email, dept_id) VALUES(:ln,:email,:deptid)";
		
		Map<String,Object> paramMap = new HashMap<>();
		paramMap.put("ln", "FF");
		paramMap.put("email", "ff@atguigu.com");
		paramMap.put("deptid", 2);
		namedParameterJdbcTemplate.update(sql, paramMap);
	}
	
	/**
	 * 从数据库中获取一条记录，实际得到对应的一个对象
	 * 注意不是调用queryForObject(String sql, Class<Employee> requiredType, Object... args)方法!
	 * 而需要调用queryForObject(String sql, RowMapper<Employee> rowMapper, Object...args)
	 * 1、其中的RowMapper指定如何去映射结果集的行，常用的实现类为BeanPropertyRowMapper
	 * 2、使用SQL中列的别名完成列名和类的属性名的映射。例如last_name lastName
	 * 3、不支持级联属性，JdbcTemplate到底是一个JDBC的小工具，而不是ORM框架
	 */
	
	/**
	 * 获取单个列的值，或做统计查询
	 * 使用queryForObject(String sql, Class<Long> requiredType)
	 */
	@Test
	public void testQueryForObject2() {
		String sql = "SELECT count(id) FROM employees";
		long count = jdbcTemplate.queryForObject(sql, Long.class);
		
		System.out.println(count);
	}
	
	/**
	 * 查到实体类的集合
	 * 注意调用的不是queryForList方法
	 */
	@Test
	public void testQueryList() {
		String sql = "SELECT id, last_name lastName, email FROM employees WHERE id > ?";
		RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);
		List<Employee> employees = jdbcTemplate.query(sql, rowMapper, 5);
		
		System.out.println(employees);
	}
	
	
	@Test
	public void testQueryForObject() {
		String sql = "SELECT id, last_name lastName, email, dept_id as \"department.id\" FROM employees WHERE id = ?";
		RowMapper<Employee> rowMapper = new BeanPropertyRowMapper<>(Employee.class);
		Employee employee = jdbcTemplate.queryForObject(sql, rowMapper, 1);
		
		System.out.println(employee);
	}
	
	/**
	 * 执行批量更新，批量的INSERT,UPDATE,DELETE
	 * 最后一个参数是Object[]的List类型：因为修改一条记录需要一个Object的数组。那么多条不就需要多个Object的数组吗
	 */
	@Test
	public void testBatchUpdate() {
		String sql = "INSERT INTO employees(last_name, email, dept_id) VALUES(?,?,?)";
		
		List<Object[]> batchArgs = new ArrayList<>();
		batchArgs.add(new Object[] {"AA","aa@atguigu.com",1});
		batchArgs.add(new Object[] {"BB","bb@atguigu.com",2});
		batchArgs.add(new Object[] {"CC","cc@atguigu.com",3});
		batchArgs.add(new Object[] {"DD","dd@atguigu.com",3});
		batchArgs.add(new Object[] {"EE","ee@atguigu.com",2});
		jdbcTemplate.batchUpdate(sql, batchArgs);
	}
	
	/**
	 * 执行insert,update,delete
	 */
	@Test
	public void testUpdate() {
		String sql = "UPDATE employees SET last_name = ? WHERE id = ?";
		jdbcTemplate.update(sql,"Jack",5);
	}
	
	@Test
	void testDataSource() throws SQLException {
		DataSource dataSource = ctx.getBean(DataSource.class);
		System.out.println(dataSource.getConnection());
	}

}
```
