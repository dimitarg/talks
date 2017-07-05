---
# sane-dbc
> A sane approach to interacting with an RDBMS in Java

Dimitar Georgiev, Novarto Ltd.

<span>https://github.com/dimitarg</span>

---
### Mission

> `sane-dbc` addresses the aspect of RDBMS interaction in the Java programming language.
It optimizes for programmer efficiency, robustness and runtime performance.

---
### Agenda

- Current situation |
- Essence of DB Interaction |
- Basic usage |
- Composition |

---
### Java - Current state of affairs

- ORM |
- Code generation (jOOQ) |
- Utilities - Spring JdbcTemplate, Apache DbUtils, etc |

---
### A program in plain JDBC

```sql
CREATE TABLE FOO
(ID INTEGER IDENTITY PRIMARY KEY, DESCRIPTION NVARCHAR(100))
```

+++
### A program in plain JDBC
```java
void insertSomeFoos(Connection c) throws SQLException
{
    try(PreparedStatement insert = c.prepareStatement(
        "INSERT INTO FOO(DESCRIPTION) VALUES (?)"))
    {
            for(String descr: asList("one", "two", "three"))
            {
                insert.setString(1, descr);
                insert.executeUpdate();
            }

    }
}
```
+++
### A program in plain JDBC
```java
List<Foo> selectTheFoos(Connection c) throws SQLException
{
    try(PreparedStatement s = c.prepareStatement(
            "SELECT ID, DESCRIPTION FROM FOO"))
    {
        List<Foo> result = new ArrayList<>();
        ResultSet rs = s.executeQuery();
        while(rs.next())
        {
            result.add(new Foo(rs.getInt(1), rs.getString(2)));
        }        

        return result;

    }
}
```


---
### Essentially,

```java
void insertSomeFoos(Connection c) throws SQLException;
List<Foo> selectTheFoos(Connection c) throws SQLException;
```
@[1]
@[2]

+++
`=>`

+++
## This
```java
A run(Connection c) throws SQLException;
```
@[1](fj.control.db.DB)

---
### Descripton
>A `DB` instance is a **value** which **describes** a database interaction.

```java
public class SelectOp<A> extends DB<List<A>>
{
    @Override
    public List<A> run(Connection c) throws SQLException
    {
     // do the business
    }
}

```

+++
### Descripton
It describes
- Does not read |
- Does not update |
- Does not throw |

+++?image=assets/sane-dbc/nuke.jpg
### Descripton
- Does not launch the nukes |

+++
### Descripton

```java
public DB<List<Foo>> selectTheFoos()
{
    return new SelectOp("SELECT * FROM FOO", ...)
}

public void doesNotLaunchTheNukes()
{
    selectTheFoos();
    selectTheFoos();
    selectTheFoos();
}
```

---
### That's it. 
- Questions and beers?
- Really, that's all there is |

---
### Interpretation

>An interpreter is the piece of code which takes a `DB<A>` and calls its `run()` method.

- The interpreter launches the nukes |
- May wrap the result of run() in another type |

+++
### Interpretation
- Responsible for all aspects of actual execution |
    - Transactionality or lack thereof |
    - Connection management |
    - Error handling |
    - Forking, or lack thereof |


+++
### Simple interpretation
```java
SyncDbInterpreter dbi = new SyncDbInterpreter(
    () -> DriverManager.getConnection("jdbc:hsqldb:mem:test", "sa", "")
);

DB<List<Foo>> selectOp = selectTheFoos();    
List<Foo> foos = dbi.submit(selectOp);
```
@[1](Interpreters are stateless, so nothing special here)
@[2](Needs a supplier for connections. Lazy, still no side effect here)
@[5]
@[6](Turns a `DB<A>` into an `A`, throws `RuntimeException` on failure)

+++
### Simple transactional interpretation
```java
DataSourse ds = getYourPooledDatasource();
SyncDbInterpreter dbi = new SyncDbInterpreter(
    () -> ds.getConnection()
);

DB<Integer> batchInsert = batchInsertFoos(
        asList("foo", "bar", "baz"));    
Integer updateCount = dbi.transact(batchInsert);
```
@[1](You can use a datasource as a supplier of connections)
@[2](Same interpreter type)
@[6](Will return the update count upon interpretation)
@[8](Turns a `DB<A>` into an `A`, rolls back and throws `RuntimeException` on failure)

+++
### Production setup
```java
HikariDataSource ds = Hikari.createHikari(
    jdbcUrl, user, pass,
    DbSpecific.defaultMysqlConnectionProps(false));

ExecutorService executor = Executors.newCachedThreadPool();

AsyncDbInterpreter dbi = new AsyncDbInterpreter(ds, executor);
CompletableFuture<List<Foo>> foos = dbi.submit(selectTheFoos());


```
@[1](HikariCP support for free)
@[3](Sensible defaults for MySQL, can override)
@[5](Worker pool for blocking on JDBC and Hikari Pool)
@[7]
@[8](Turns a `DB<A>` into a `CompletableFuture<A>`. Failure of the operation will fail the future)

---
###Built-in ops

[Sample usage](https://github.com/novarto-oss/sane-dbc/blob/master/sane-dbc-examples/src/test/java/com/novarto/sanedbc/examples/BasicUsage.java)

---
### Composition
Principle of writing software
- compose functions into programs
- compose programs into larger programs

This never works with side effects
- But it does with descriptions of them

+++
### Composition
```java
public abstract class DB<A> 
{
    public final <B> DB<B> map(final F<A, B> f){/*...*/}
    
    public final <B> DB<B> bind(final F<A, DB<B>> f){/*...*/}
    
    //...
}


```
@[3]
@[5](also known as `flatMap`)

+++ 
### `map()`
```java
DB<List<Employee>> selectEmployees =
    EmployeeDb.selectByIds(asList(2, 1, 4, 3));

DB<Map<Integer, List<Employee>>> byId =
    selectEmployees.map(employees ->
        employees.stream().collect(groupingBy(
            employee -> employee.departmentId)));
```
@[1]
@[2]
@[5](Mapping a DB with a function gives another DB. No side effect)
@[7](Group employees by department id, using java.util.Stream)

+++
### `map()`
```java
DB<Option<User>> selectUser = selectUser(email);
DB<Boolean> loginOk = 
selectUser.map(userOption -> {
    if (userOption.isNone())
        {
            return false;
        }

        User user = userOption.some();
        return hash(password).equals(user.hash);
    });
```
@[1]
@[2]
@[3]
@[4]
@[6](invalid username)
@[9]
@[10](validate password)
