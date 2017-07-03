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
>A `DB` instance is a **description** of a database interaction.

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




