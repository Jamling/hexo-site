---
title: User guide
date: 2016-01-30 17:43:26
layout: project
title2: project.userguide
---

## Add library dependencies
- Android studio
```gradle
dependencies {
    compile 'cn.ieclipse.aorm:aorm-core:1.1.2'
}
```
- Eclipse
Install [Android ADT-extension](https://marketplace.eclipse.org/content/android-adt-extensions) plugin

## Settings

Optional settings, setting the Aorm globally.
```java
Aorm.enableDebug(true);// Enable/Disable debug to print SQL
Aorm.allowExtend(false);// Disable model bean extend
Aorm.setExactInsertOrUpdate(true);//Set use actuarial insertOrUpdate. If true, will query the object from database, insert if not exists or update if exist, otherwise insert when PK is 0 or update when PK more than 0 (maybe update fail)
```

## Create model bean

### Add mapping table
Add @Table annotation in your java bean

```java
@Table(name = "student")
public class Student implements java.ioSerializable{
    //...
}
```

### Add column mapping
Add @Column annotation for java bean field

- pk (primary key field)
id is true
```java
    @Column(name = "_id", id = true)
    private long id;
```

- normal field

```java
    @Column(name = "_name")
    private String name;

    @Column()
    private int age;
```

- un-mapping field
No Aorm annotation assigned.
```java
    private String address;
```

The bean must generate getter/setter methods

## Create table
If Android ADT-extensions install in Eclipse. Select the java element includes Aorm beans and Open New ORM Provider to generate ContentPrivider in wizards.

Or create manually.
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

## Usage

### Query

#### Simple query
Query all students
```java
Session session = ExampleContentProvider.getSession();
// simplest query, query all student table.
Criteria criteria = Criteria.create(Student.class);
List<Student> list = session.list(Student.class);
```
Query to cursor, so used it in CursorAdapter
```java
Cursor c = session.query(criteria);
```

Query to single object
```java
// query student whose id is 4
s = session.get(Student.class, 4);
```
#### Restriction query
Equals
```java
// add restrication: id equals
criteria.add(Restrictions.eq("id", 1));
```
Like
```java
// add restriction: name like Jamling
criteria.add(Restrictions.like("name", "Jaming"));
```
Compare
```java
// add restriction: age > 30
criteria.add(Restrictions.gt("age", 30));
```
Order
```
// add order
criteria.addOrder(Order.asc("age"));
```
Distinct
```
// set district
criteria.setDistinct(true);
```
Limit
```
// set limit from row 10 to 20
criteria.setLimit(10, 10);
```

#### Join query
The Criteria support four join
1. Left Join
2. Left outer join
3. Inner join
4. Cross join

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

### Insert
```java
Session session = ExampleContentProvider.getSession();
// insert
Student s = new Student();
s.setName("Jamling");
long rowId = session.insert(s, null);
```

### Update
```java
// update student's name to Jame whose id is 1
s.setId(1);
s.setName("Jame");
int rows = session.update(s);
```

### Delete
```java
// delete student whose id is 2
session.deleteById(Student.class, 2);
```




