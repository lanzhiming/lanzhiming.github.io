---
layout: post
title:  MyBatis insert返回主键不成功
date:   2017-04-05 11:08:00 +0800
categories: document
tag: 教程
---

* content
{:toc}

> 说明：Mybaits的insert/update一般默认返回记录的更新条数，业务需要在保存完实体（insert）之后需要返回主键值。
## 官网说明
[Mybaits官方文档](http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html#insert_update_and_delete)
* 这里我以Mysql为例。
```xml
<insert id="insertAuthor">
  insert into Author (id,username,password,email,bio)
  values (#{id},#{username},#{password},#{email},#{bio})
</insert>
```
* 首先，如果你的数据库支持自动生成主键的字段（比如 MySQL 和 SQL Server），那么你可以设置 useGeneratedKeys=”true”，然后再把 keyProperty 设置到目标属性上就OK了。例如，如果上面的 Author 表已经对 id 使用了自动生成的列类型，那么语句可以修改为:
```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username,password,email,bio)
  values (#{username},#{password},#{email},#{bio})
</insert>
```
* 根据官方文档，我的业务代码按理说可以这么写，然而并没有返回主键值。返回的还是行数。

```xml
<insert id="insertKPRecord" parameterType="org.krt.order.entity.Kprecord" useGeneratedKeys="true"
    keyProperty="id">
	insert into t_kprecord(kpcode,kpdate,jxsid,kpname,spid,spamount,spunit,kptype,purpose)
	values(#{kpcode},NOW(),#{jxsid},#{kpname},#{spid},#{spamount},#{spunit},#{kptype},#{purpose});
</insert>
```
既然官网的文档都不行，那看看别人实践的。
## 博客园写法
* 这个讲的比较详细，大家可以看看。
[Insert操作详解（返回主键、批量插入）](http://www.cnblogs.com/fsjohnhuang/p/4078659.html)

* 具体的写法
```xml
<insert id="add" parameterType="EStudent">
  // 下面是SQLServer获取最近一次插入记录的主键值的方式
  <selectKey resultType="_long" keyProperty="id" order="AFTER">
    select @@IDENTITY as id
  </selectKey>
  insert into TStudent(name, age) values(#{name}, #{age})
</insert>
```
* 他这里是针对的sqlserver,不过给了我灵感的一个标签是<selectKey>
## 最终正确结果
* 我结合了官方文档，然后做一个结合使用。
```xml
<insert id="insertKPRecord" parameterType="org.krt.order.entity.Kprecord" >
	<selectKey resultType="java.lang.Integer" order="AFTER" keyProperty="id">    
      SELECT LAST_INSERT_ID() AS ID      
   	</selectKey>
	insert into t_kprecord(kpcode,kpdate,jxsid,kpname,spid,spamount,spunit,kptype,purpose)
	values(#{kpcode},NOW(),#{jxsid},#{kpname},#{spid},#{spamount},#{spunit},#{kptype},#{purpose});
</insert>
```
