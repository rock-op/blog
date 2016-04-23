---
layout: default
title: "java中的静态工厂方法"
categories: java
date: 2016-04-14
---

According to http://www.javapractices.com/topic/TopicAction.do?Id=21

Just like constructor, factory method are static methods that return an instance of the native class.

Advantages of factory method:

- have names, unlike constructors, which can clarify code.
- do not need to create a new object upon each invocation - objects can be cached and reused, if necessary.
- can return a subtype of their return type - in particular, can return an object whose implementation class is unknown to the caller. This is a very valuable and widely used feature in many frameworks which use interfaces as the return type of static factory methods.

According to http://stackoverflow.com/questions/929021/what-are-static-factory-methods-in-java

> We don't want to provide direct access to the connections, because they're resource intensive. So we use a static factory method getDbConnection which creates a connection if we're below the limit. Otherwise, it tries to provide a "spare" connection, and finally fails with an exception if there are none.


```
public class DbConnection
{
   private static final int MAX_CONNS = 100;
   private static int totalConnections = 0;
   private static Set<DbConnection> availableConnections = new HashSet<DbConnection>();
   // The constructors are marked private, so they cannot be called except from inside the class
   private DbConnection()
   {
     totalConnections++;
   }
   // the factory method is marked as static so that it can be called without first having an object.
   public static DbConnection getDbConnection()
   {
     if(totalConnections < MAX_CONNS)
     {
       return new DbConnection();
     }
     else if(availableConnections.size() > 0)
     {
         DbConnection dbc = availableConnections.iterator().next();
         availableConnections.remove(dbc);
         return dbc;
     }
     else {
       throw new NoDbConnections();
     }
   }
   public static void returnDbConnection(DbConnection dbc)
   {
     availableConnections.add(dbc);
   }
}
```
