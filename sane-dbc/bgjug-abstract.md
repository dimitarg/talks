# About the talk

In this talk we are going to present a principled, robust and performant approach to interacting with RDBMS in the Java
programming language, implemented via the [sane-dbc](https://github.com/novarto-oss/sane-dbc) micro-library. We approach
the problem via plain old SQL, while utilizing the [DB Monad](https://github.com/functionaljava/functionaljava/blob/master/core/src/main/java/fj/control/db/DB.java)
to keep our programs sane (i.e. referentially transparent and boilerplate-free) and easy to reason about. 

Besides basic usage patterns, we are going to touch on topics such as
- Composition
- Threading
- Production-grade connection management
- Multithreaded correctness through immutability

The talk presumes no prerequisites besides intermediate command of Java, and basic knowledge of SQL.


# About the author

![image](../assets/DSC03207.JPG)

Dimitar Georgiev is a software engineer with a decade of experience programming on the JVM platform. He is specialized in Java, Scala, Functional Programming, HTTP, system integration, performance measurements and optimizations, and quality assurance through automating and testing all the things.

Dimitar is an author of the [sane-dbc](https://github.com/novarto-oss/sane-dbc) library and the soon-to-be opensourced
[cletty](https://github.com/novarto-oss/cletty) HTTP/2 server. 

Since 2013 he is employed at Novarto Ltd., where he works as a software architect, focusing on E-Commerce systems and custom applications.
