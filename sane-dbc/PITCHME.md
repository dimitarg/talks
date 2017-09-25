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

- Mostly generated SQL |
    - But handwritten SQL is the best SQL |
- No type safety for side effects |
    - Results in distributed system fallacies |
- Lack of referential transparency |
    - Cannot reason about programs |
    - Cannot compose programs |


+++
### No type safety for side effects

```java
public Thingie foo()
{
    return new Thingie(42, "A thingie");
}

public Thingie bar()
{
    return entityManager.find(Thingie.class, 42);
}
```

- Same freaking signature |
- Compiler wont't stop you |
- For loop with a DB call in the body |

+++
### Fallacies of distrubuted computing

This makes the programmer think they are calling a function in local address space, when in fact they are dealing with
a distributed system.

- This is EJB and JAX-WS all over again
- https://en.wikipedia.org/wiki/Fallacies_of_distributed_computing


+++
### Referential transparency

```java
int x = foo();
int y = x;
```

<===>

```java
int x = foo();
int y = foo();
```

- Powerful reasoning technique - [equational reasoning](https://www.google.bg/search?q=equational+reasoning) |


+++
### Referential transparency - broken

```java
public int foo()
{
    doSomethingWithDatabase();
    // all bets are off now
    // no referential transparency
    // no function composition
    // no automatic reasoning
    // no maths, mostly
    // this is no longer a function
    return 42;
}
```

---
### Towards a better world

- These problems were solved decades ago |
- The solution is simple |
    
+++
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


+++
### `bind()`

```java
DB<Boolean> login = login(email, pass);
DB<Either<String, List<Order>>> selectOrders =
    login.bind(success -> {
        if(!success)
        {
            Either<String, List<Order>> err =
                Either.left("auth failure");
                return DB.unit(err);
        }
        
        return selectOrdersByEmail(email)
        .map(orders ->
            Either.right(orders));
    }
);
            
```
@[1]
@[2]
@[3](take the result of one operation, and return another operation)
@[4]
@[8](DB.unit returns a successful DB with a result x)
@[11](`DB<List<Order>>`)
@[12]
@[13](Result is `DB<Either<String, List<Order>>>`)

+++
### `bind()` - sequencing side effects

```java
DB<Unit> wipeUser = deleteOrders(user).bind(ignore ->
    deleteAddresses(user).bind(ignore2 -> 
        deleteUser(user).map(ignore3 ->
            Unit.unit()
        )
    )
);
```

---
### Summary
- You write the SQL
- DB handles the composition
- We handle the JDBC boilerplate

+++
### Summary
- Flexibility
- Performance
    - Pure SQL
    - Abstraction practically zero-cost
- Robustness and sanity
- Reuse and conciseness through composition

+++
### Thank you!
<span>https://github.com/novarto-oss/sane-dbc</span>

