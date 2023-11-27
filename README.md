# Introduction to Table Relations

## Learning Goals

- Examine the atomicity principle of relational database design.
- Model one-to-many relationships between tables.
- Model many-to-many relationships between tables.
- Model one-to-one relationships between tables.
- Use a foreign key as a means of implementing a table relationship

***

## Key Vocab

- **Primary Key**: a value that uniquely identifies one row in a table.
- **Foreign Key**: a column or group of columns in one table that contain the value
  of another table's primary key.  A foreign key connects two tables together.
- **One-to-One**: a type of relationship between tables where a row in
  one table is connected to one rows in another table. **e.g.** A person
  has one social security profile.
- **One-to-Many**: a type of relationship between tables where a row in
  one table is connected to multiple rows in another table. **e.g.** A person
  may own many pets.
- **Many-to-Many**: a type of relationship between tables where multiple
  rows in table A are connected to multiple rows in table B. **e.g.**
  Students have many classes and classes have many students.

***

## Introduction

The majority of databases we work with as developers have more than
one table, and the tables are connected together in various ways to form
table relationships. In this lesson, we'll discuss how to model
relationships between different tables,
and explore the different types of table relationships that can exist.

## Properties of Relational Database Tables

Let's review a few design principles concerning tables
in a relational database:

- A table holds information about objects of a similar type.
- Each column within a table has a unique name.  A column may also be referred to as a field or attribute.
- All values within a column have the same type and are **atomic**.
- A row within a table represents a unique object or entity.  A row may also be referred to as a record.
- Each table must define a column that contains a unique identifier, called a **primary key**.
  - The primary key may consist of one or more columns.
    However, we will design our tables to use a single column named `id` that contains a unique integer for the primary key.
- Rows among multiple tables can be related using **foreign keys**, in which a column in
  one table store the value of the primary key of another table.

The third principle states that values within a column have the same type and are **atomic**.
Having the same type means the values are either all integers, or all strings, etc.
But what does it mean for the values to be atomic?

### Atomicity

A value is **atomic** if it can't be divided or split into smaller parts.  This is a
very important principle of relational database design.  Let's look at examples
of tables that contain non-atomic values to understand why this can present problems. 

Consider the following table that stores information about companies:

| id  | name           | address                                      |
|-----|----------------|----------------------------------------------|
| 1   | SuperMart      | 123 Fiction St, Kansas City, Missouri, 64101 |
| 2   | BigCo          | 45 Rue Dr, Kansas City, Kansas, 66012        |
| 3   | MegaMart       | 5 Street Blvd, Topeka, Kansas, 66601         |
| 4   | GiantCo        | 1 Kansas Lane, Boston, Massachusetts, 02111  |
| 5   | 100DollarStore | 45 Mystery Dr, Kansas, Vermont, 05252        |

The `address` column is **not atomic** since it consists of 4 parts:
street address, city, state, and zip. 

How would we write a query to get names of companies from the state of Kansas?
While we could use the `LIKE` keyword for pattern matching on the substring
"Kansas", it would be difficult to avoid getting companies with "Kansas"
in the street address or city name.
For example, this query would produce an incorrect result as it matches every row in the table:

```sql
SELECT name
FROM company
WHERE address LIKE '%Kansas%'
```

 We need to evolve the
table to divide the address into multiple columns, each containing a single
atomic value (street could possibly be further divided, but most applications
won't require that level of atomicity):

| id  | name           | street         | city        | state         | zip   |
|-----|----------------|----------------|-------------|---------------|-------|
| 1   | SuperMart      | 123 Fiction St | Kansas City | Missouri      | 64101 |
| 2   | BigCo          | 45 Rue Dr      | Kansas City | Kansas        | 66012 |
| 3   | MegaMart       | 5 Street Blvd  | Topeka      | Kansas        | 66601 |
| 4   | GiantCo        | 1 Kansas Lane  | Boston      | Massachusetts | 02111 |
| 5   | 100DollarStore | 45 Mystery Dr  | Seattle     | Washington    | 98101 |

Now we can easily query to get the names of companies in the state of Kansas:

```sql
SELECT name
FROM company
WHERE state = 'Kansas'
```

The query results in 2 companies:

```text
BigCo
MegaMart
```

Atomicity also means that a column should not contain a list of values.
Let's look at another example database that contains two tables: `employee`
and `department`:


#### Employee Table

| id  | first_name | last_name  | salary |
|-----|------------|------------|--------|
| 1   | Kiran      | Willow     | 76000  |
| 2   | Dani       | Elm        | 99000  |
| 3   | Hao        | Pine       | 45000  |
| 4   | Tal        | Oak        | 88000  |
| 5   | Yuri       | Birch      | 150000 |

#### Department Table

| id  | name      | location   | employees |
|-----|-----------|------------|-----------|
| 1   | Payroll   | Building A | 4, 5      |
| 2   | Marketing | Building B | 1, 2, 3   |

- The `employee` table stores one row per employee, and each column stores an atomic value.
- The `department` table stores the name and location, along with a comma-separated
  list of ids of employees that work for that department. 
  The string represents a list of values and thus is not atomic.

What if we want to count the number of employees per department?
It is difficult to write such a query since the `employees` column is not atomic.

## One-To-Many Relationships

The relationship between `department` and `employee` is **one-to-many**:
- An employee works for one department, while each department may have any number of employees.

We can model the employee and department as separate entities
as shown in the **Entity Relationship Diagram** (ERD) below.
The connecting line represents the **one-to-many** relationship.
The three forked lines with the `*` represents the many side of the
relationship (a department has many employees),
while the single line with `1` represents the one side (an employee works in one department).

![employee department ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/dept_emp_erd.png
)

A relational database relies on **primary keys** and **foreign
keys** to establish relationships between tables.

- **Primary Key**: a column in a table with an identifier (ID) that uniquely
  identifies one specific record, or row, in a table.
- **Foreign Key**: a column in one table that refers to a primary key in
  another table.

Since a department has many employees, we can't use a single column in the `department` table
to store the relationship.  Instead, we must update the `employee` table
to store the employee's department id.

The redesigned `department` table no longer contains the column for employees:

| id  | name      | location   |
|-----|-----------|------------|
| 1   | Payroll   | Building A |
| 2   | Marketing | Building B |

The redesigned `employee` table contains a new **foreign key** column `department_id` to store the **one-to-many** relationship:

| id  | first_name | last_name  | salary | department_id |
|-----|------------|------------|--------|---------------|
| 1   | Kiran      | Willow     | 76000  | 2             |
| 2   | Dani       | Elm        | 99000  | 2             |
| 3   | Hao        | Pine       | 45000  | 2             |
| 4   | Tal        | Oak        | 88000  | 1             |
| 5   | Yuri       | Birch      | 150000 | 1             |


- We store a single integer in the `department_id` column, thus each employee works in one department.
- Multiple rows in the `employee` table may have the same value in the `department_id` column,
  thus a department may have multiple employees.
- All columns contain atomic values.

Now we can easily write the query to count the number of employees per department:

```sql
SELECT department_id, count(*)
FROM employee
GROUP BY department_id
```

The query result contains 2 rows showing the number of employees in each department.

```text
department_id   count 
1               2
2               3
```


What if we want to print the name of each department, along with the employee count?
We need information from two tables: 
(1) the count per `department_id` from the `employee` table, and
(2) the name from the row in the `department` table that matches the `department_id` value
from the `employee` table.
That exact process of using a key in one table to identify a corresponding row
in another table is what an SQL `JOIN` statement will do for us!  We will explore the
`JOIN` statement in the next lesson.

Other examples of **one-to-many** relationships:



- A passport is issued for a single person. A person may have many passports (i.e. dual citizenship, current vs expired).  
The `passport` table foreign key `person_id` references the `person` table primary key `id`.    

![passport person erd](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/person_passport.png)


- A customer can create multiple purchase orders.  Each purchase order is for one customer.  
  The `purchase_order` table foreign key `customer_id` references the `customer` table primary key  `id`.
- A purchase order may consist of several items.  Each purchase order item is for a single purchase order.  
  The `purchase_order_item` table foreign key `purchase_order_id` references the `purchase_order` table primary key  `id`.

![purchase order erd](https://curriculum-content.s3.amazonaws.com/6036/java-mod-5-intro-table-relations/purchase_order_erd.png)

## Many-To-Many Relationships

Tables may also have **many-to-many** relationships. 

- A film may have many actors, and an actor may be in many films:  
  ![film actor ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/film_actor_erd.png)
- An author may write many books, and a book may have multiple authors:  
  ![author book ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/author_book_erd.png)

For example, assume we have the following `book` and `author` tables:

#### Book table

| id  | title                            | year | ISBN           |
|-----|----------------------------------|------|----------------|
| 1   | The C Programming Language       | 1978 | 978-0131101630 |
| 2   | The Unix Programming Environment | 1983 | 978-0139376818 |
| 3   | Unix, A History and Memoir       | 2019 | 978-1695978553 |

#### Author table

| id  | author              |
|-----|---------------------|
| 1   | Brian W. Kernighan  |
| 2   | Dennis M. Ritchie   |
| 3   | Rob Pike            | 



We need to be able to represent the following relationships between authors and books:

- Kernighan and Ritchie co-authored the C book.
- Kernighan and Pike co-authored the Unix book.
- Kernighan authored the memoir.

The relationship between authors and books is **many-to-many**, which means it
can't be stored in the `book` or `author` tables themselves.
We need to create a new table `book_author` (or `author_book`) to store
the relationship. The table would have a column `book_id` that is
a foreign key to the `book` table, along with a column
`author_id` that is a foreign key to the `author` table.

- Kernighan (author id 1) has written 3 books (book id 1, 2, 3), so there will be 3 rows with value 1 in the `author_id` column, one per book.
- The Unix book (book id 2) has two authors (author id 1, 2), so there will be 2 rows with value 2 in the `book_id` column, one per author.

We store the relationships between books and authors as shown:

| book_id | author_id |
|---------|-----------|
| 1       | 1         |
| 1       | 2         |
| 2       | 1         |
| 2       | 3         |
| 3       | 1         |


In the next lesson, we will explore how to implement a primary key for many-to-many relationships.

## One-To-One Relationships

**One-to-one** relationships are possible among tables, although not frequent.
Typically this type of relationship is used to store optional attributes for an entity
to avoid columns with lots of null values.  For example, assume we have a `pet` table
that stores information about various types of pets (dog, cat, fish, etc).
We would not want columns in the pet table to store data specific to fish such as
its water temperature range and whether its habitat is fresh water or salt water.
Such attributes are not relevant for most pets.  

Instead, we can create a separate table `fish_physiology` for that information, and create
a **one-to-one** relationship with the `pet` table. The new table would only
contain rows for fish, not dogs or cats. 

![fish ERD](https://curriculum-content.s3.amazonaws.com/6036/introduction-to-table-relations/fish_erd.png)

Note  **one-to-one** represents the maximum cardinality of the relationship.
A row in `pet` is related to at most one row in the `fish_physiology` table,
and each row in `fish_physiology` is related to at most one row in `pet`.
The minimum cardinality is 0 for `pet`, since dogs and cats won't participate in the
relationship. 

***

## Conclusion

Working with table relations takes a bit more abstract thought than working
with a single table. Instead of visualizing the data all in one place, we
have to consider how data in multiple tables can be related. Thankfully, in SQL,
the way to relate multiple tables is by using **foreign keys**, so once you are comfortable with this concept, you'll be
well on your way to mastering relational databases.


- If the relationship is one-to-many, add a foreign key attribute to the table on the "many" side of the relationship.
  - A department has many employees.  An employee works for one department.  Add the foreign key `department_id` in the `employee` table.
- If the relationship is many-to-many, create a new table with the primary keys of each entity's table.
  - A book has many authors.  An author writes many books.  Add a new table `book_author` with a composite primary key (book_id, author_id),
    with each id being a foreign key to the corresponding table.
- If the relationship is one-to-one, add a unique foreign key attribute in the table that represents the optional attributes.
  - Some pets are fish.  A fish has physiology attributes.  Add unique foreign key `fish_id` in the `fish_physiology` table.


***


## Resources

- [What is a Relational Database? - Google Cloud](https://cloud.google.com/learn/what-is-a-relational-database)    
- [Difference between Primary Key and Foreign Key - GeeksforGeeks](https://www.geeksforgeeks.org/difference-between-primary-key-and-foreign-key/)
- [dbdiagram.io: ERD Drawing Tool](https://dbdiagram.io)  
- [PostgreSQL Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)   
- [PostgreSQL FOREIGN KEY Tutorial](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-foreign-key/)  
- [PostgreSQL UNIQUE Tutorial](https://www.postgresqltutorial.com/postgresql-tutorial/postgresql-unique-constraint/)  

