---
title: User guide
date: 2016-01-30 17:43:26
layout: project
title2: project.userguide
---

## 引入
- Android studio
在项目build.gradle中添加以下依赖
```gradle
dependencies {
    compile 'cn.ieclipse.aorm:aorm-core:1.1.2'
}
```
- Eclipse
安装 [Android ADT-extension](https://marketplace.eclipse.org/content/android-adt-extensions) 插件，然后Add ORM capapility

## 设置

设置Aorm的一些全局设置
```java
Aorm.enableDebug(true);// 启用/禁用debug，在debug模式下会打印SQL等日志
Aorm.allowExtend(false);// 允许/禁止模型继承，如果开启继承，ROM映射模型将会从父类中查找映射字段
Aorm.setExactInsertOrUpdate(true);//设置是否精确的插入或更新操作。如果设置为true, 在执行写操作之前先查询数据库，如果查询出来的对象不存在，则插入新数据；如果存在，则更新数据库。如果设置为false，那么则根据主键是否为0来决定插入或更新操作（如果主键大于0，则执行更新操作，有可能会导致更新失败）。
```

## 创建映射模型

### 设置映射表
只需简单地对class添加@Table注解，即可设置对数据库表的映射

```java
@Table(name = "student")
public class Student implements java.ioSerializable{
    //...
}
```

### 设置映射字段
对Java bean的属性添加@Column注解

- pk (标识为主键)
id is true
```java
    @Column(name = "_id", id = true)
    private long id;
```
- 普通字段
```java
    @Column(name = "_name")
    private String name;
    @Column(name = "_age")
    private int age;
```
- 非映射字段
如果不想映射数据库字段，则无需设置@Column注解
```
    private String address;
```

映射的属性必须设置标准的Getter/Setter属性。

## 创建表
如果使用的是Eclipse已经安装了Android ADT-extensions插件，则可以使用此插件来生成数据库：右键选择Aorm模型（可以多选）->右键菜单->ADT extensions->New ORM Provider打开组件生成向导来自动生成。

当然，您也可以手动创建，如下代码所示：
```java
        mOpenHelper = new SQLiteOpenHelper(this.getContext(), "example.db",
                null, 1) {
            public void onCreate(SQLiteDatabase db) {
                // method 3: use AORM to create table
                Aorm.createTable(db, Grade.class);
                Aorm.createTable(db, Student.class);
                Aorm.createTable(db, Course.class);
            }
        };
```

## 示例

### 查询

#### 简单查询
查询所有的学生记录。
```java
Session session = ExampleContentProvider.getSession();
// 最简单的查询，查询所有的记录
Criteria criteria = Criteria.create(Student.class);
List<Student> list = session.list(Student.class);
```
查询为cursor对象，cursor可以直接在CursorAdapter中使用
```java
Cursor c = session.query(criteria);
```

查询单个学生信息
```java
// 查询主键为4的学生
s = session.get(Student.class, 4);
```
#### 条件查询
Equals查询
```java
// 添加id equals条件
criteria.add(Restrictions.eq("id", 1));
```
Like查询
```java
// 添加姓名中包含Jamling的学生
criteria.add(Restrictions.like("name", "Jaming"));
```
比较查询
```java
// 添加年龄大于30岁条件
criteria.add(Restrictions.gt("age", 30));
```
排序
```
// 按年龄升序排序
criteria.addOrder(Order.asc("age"));
```
Distinct
```
// set district
criteria.setDistinct(true);
```
Limit
```
// 限定查询第10到20条记录
criteria.setLimit(10, 10);
```

#### 连接查询
Criteria支持以下四种连接
1. 左连接
2. 左外连接
3. 内连接
4. 交叉连接（笛卡尔积）

通过Criteria.create方法来创建连接。如下示例代码。

```java
Criteria criteria = Criteria.create(Student.class, "s")
                .addChild(Grade.class, "g")
                .add(Restrictions.eqProperty("s.id", "g.sid"));
// query to list.
List<Object[]> ret = session.listAll(criteria);
Object[] item = ret.get(0);
Student s = (Student) item[0];
Grade g = (Grade) item[1];
```

### 插入记录

```java
Session session = ExampleContentProvider.getSession();
// insert
Student s = new Student();
s.setName("Jamling");
long rowId = session.insert(s, null);
```

### 更新记录

```java
// update student's name to Jame whose id is 1
s.setId(1);
s.setName("Jame");
int rows = session.update(s);
```

### 删除记录

```java
// delete student whose id is 2
session.deleteById(Student.class, 2);
```
