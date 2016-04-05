---
title: 一个把hibernate/jpa/jdbc封装成类似ActiveRecord风格API的数据库操作库jpa-utils
date: 2015-04-06 00:27:13
tags: Java
---
本博客系统使用Java Web实现,其中数据库层操作虽然直接写SQL或者使用MyBatis比较可控,
但是我对于快糙猛的项目一向懒得慢慢写,所以挺久之前封装过一个类似ActiveRecord的数据库操作库,
是对Hibernate/JPA/jdbc的封装,从而在上层使用时比较方便,又可以随时fallback到使用下层的Hibernate/JPA/jdbc. 项目名叫做jpa-utils


    项目地址: [https://github.com/zoowii/jpa-utils](https://github.com/zoowii/jpa-utils)


## 目前的特性包括

* 底层基于JPA或者Hibernate的Session/SessionFactory，基于HQL/SQL，比自己再轮一个类HQL稳定
* 也提供直接基于jdbc的支持,从而可以不依赖Hibernate等ORM,也方便直接和jdbc Connection, MyBatis, DbUtils, 数据库连接池等库直接集成使用
* 直接基于jdbc Connection的话,ORM映射部分目前只支持MySQL和H2数据库,其他数据库待支持
* 可以自动从JPA配置创建管理session,也可以手动指定EntityManagerFactory/EntityManager/SessionFactory(hibernate)/Session(hibernate)来构造jpa-utils中的Session来使用,还可以直接从jdbc Connection构造Session
* 提供类似ActiveRecord的使用方便友好的API，特别是查询API
* 查询的核心Finder类可以单独使用，直接使用到现有的使用JPA或Hibernate的代码中，只需要根据现有EntityManager/Session(hibernate)构造一个jpa-utils的session，然后使用Finder类来查询就好了
* 支持类似MyBatis的执行编程式XML中的SQL(TODO)

因为没有发布到公网maven repo,所以如果要用,要么maven install,要么发布到自己meven repo私服,要么mvn package成jar后自己使用

## 使用DEMO

```
// maven
!!! First deploy it to you local maven nexus, then.
<dependency>
     <groupId>com.zoowii</groupId>
     <artifactId>jpa-utils</artifactId>
     <version>x.y.z</version>
</dependency>

// create
Session session = EntitySession.currentSession(); // or Session.getSession(persistentUnitName);
session.begin();
try {
    for (int i = 0; i < 10; ++i) {
        Employee employee = new Employee();
        employee.setName("employee_" + StringUtil.randomString(10));
        employee.setAge(new Random().nextInt(100));
        employee.save(); // or employee.save(session);
        logger.info("new employee " + employee.getId());
    }
    session.commit();
} catch (Exception e) {
    e.printStackTrace();
    session.rollback();
}

// query
Session session = Session.currentSession();
session.begin();
try {
    Query<Employee> query = Employee.find.where().gt("age", 50);
    query = query.limit(8);
    List<Employee> employees = query.all(); // or query.all(session);
    for (int i = 0; i < employees.size(); ++i) {
        Employee employee = employees.get(i);
        logger.info((i + 1) + ". employee " + employee.getId());
    }
    logger.info("total: " + query.count()); // or query.count(session);
    session.commit();
} catch (Exception e) {
    e.printStackTrace();
    session.rollback();
}
```
