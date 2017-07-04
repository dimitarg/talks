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
List<Foo> foos = dbi.submit(selectTheFoos());
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

DB<Integer> batchInsert = batchInsertFoos(asList("foo", "bar", "baz"));    
Integer updateCount = dbi.transact(batchInsert);
```
@[1](You can use a datasource as a supplier of connection, but not mandatory)
@[2](Needs a supplier for connections. Lazy, still no side effect here)
@[6](Will return the update count upon interpretation)
@[7](Turns a `DB<A>` into an `A`, rolls back and throws `RuntimeException` on failure)







