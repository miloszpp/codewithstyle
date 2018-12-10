---
title: Slick vs Anorm - choosing a DB framework for your Scala application
tags:
  - databases
  - scala
url: 96.html
id: 96
categories:
  - Other topics
  - Scala
date: 2016-07-09 16:48:16
---

Scala doesn't offer many DB access libraries. [Slick](http://slick.lightbend.com) and [Anorm](https://www.playframework.com/documentation/2.5.x/ScalaAnorm) seem to be the most popular - both being available in the [Play framework](https://www.playframework.com). Despite both serving the same purpose, they present completely different approaches. In this post I'd like to present some arguments that might help when choosing between these two.

### What is Slick?

Slick is a **Functional Relational Mapper**. You might be familiar with Object Relational Mappers such as Hibernate. Slick embraces Scala's functional elements and offers an alternative. Slick authors claim that the gap between relational data and functional programming is much smaller than between object-oriented programming. Slick allows you to write type safe, SQL-like queries in Scala which are translated into SQL. You define mappings which translate query results into your domain classes (and the other way for INSERT  and UPDATE ). Writing plain SQL is also allowed.

```scala
// Example taken from docs
( for( c <- coffees; if c.price < limit ) yield c.name ).result
// Equivalent SQL: select COF_NAME from COFFEES where PRICE < 10.0
```

### What is Anorm?

Anorm is a thin layer providing database access. It is in a way similar to Spring's JDBC templates. In Anorm you write queries in plain SQL. You can define your own row parsers which translate query result into your domain classes. Anorm provides a set of handy macros for generating parsers. Additionally, it offers protection against SQL injection with prepared statements. Anorm authors claim that SQL is the best DSL for accessing relational database and introducing another one is a mistake.

```scala
// example taken from docs
SQL"""
  select * from Country c 
    join CountryLanguage l on l.CountryCode = c.Code 
    where l.Language = $lang and c.Population >= ${population - margin}
    order by c.Population desc limit 1"""
  .as(SqlParser.str("Country.code").single
```

### Blocking/non-blocking

As mentioned, Slick API is non-blocking. Slick queries return instances of DBIO  monad which can be later transformed into Future . There are [many benefits](http://codewithstyle.info/asynchronous-programming-scala-vs-c/) of a non-blocking API such as improved resilience under load. However, you will not notice these benefits unless your web applications is handling thousands of concurrent connections. Anorm, as a really thin layer, does not offer a non-blocking API.

### Expressibility

Slick's DSL is very expressive but it will always be less than plain SQL. Anorm's authors seem to have a point that re-inventing SQL is not easy. Some non-trivial queries are difficult to express and at times you will miss SQL. Obviously, you can always use the plain SQL API in Slick but what's the point of query type safety if not all of your queries are under control? Anorm is as expressive as plain SQL. However, passing more exotic query parameters (such as arrays or UUID s) might require spending some time on reading the docs.

### Query composability

One of huge strengths of Slick is query composability. Suppose you had two very similar queries:

```scala
SELECT name, age, occupation, c.country
FROM author AS a
LEFT JOIN cities c ON c.id = a.city_id
WHERE age > 50

SELECT name, age, occupation, c.country
FROM author AS a
LEFT JOIN cities c ON c.id = a.city_id
WHERE age < 50
```

In Slick, it's very easy to abstract the common part into a query. In Anorm, all you can do is textual composition which can get really messy.

### Inserts and updates

In Slick you can define two-way mappings between your types and SQL. Therefore, INSERT s are as simply as:

```
authors += author
```

In Anorm you need to write your INSERT s and UPDATE s by hand which is usually a tedious and error-prone task.

### Code changes and refactoring

Another important feature of Slick is query type safety. It's amazing when performing changes to your data model. Compiler will always make sure that you won't miss any query. In Anorm nothing will help you detect typos or missing fields in your SQL which will usually make you want to write unit tests for your data access layer.

### Conclusion

Slick seems to be a great library packed with very useful features. Additionally, it will most likely save your ass if you need to perform many changes to your data model. However, my point is that it comes at a cost - writing Slick queries is not trivial and the learning curve is quite steep. And you risk that the query you have in mind is not expressible in Slick. An interesting alternative is to use Slick's plain SQL API - it gives you some of the benefits (e.g. non-blocking API) but without sacrificing expressability. As always, it's a matter of choosing the right tool for purpose. I hope this article will help you to weigh in all arguments.