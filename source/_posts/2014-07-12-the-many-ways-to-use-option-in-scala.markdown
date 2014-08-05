---
layout: post
title: "Using Option in Scala"
date: 2014-07-12 19:54:36 -0400
comments: true
categories: 
---

There are many way to use Option in Scala. Per scala docs, the idiomatic way to use Option is to treat it as a collection that can have either a length of one of zero.

Let's says we have a basic class, Author
### Simple Option checking

```scala
case class Author(name:Option[String])
val author:Author = ...
```

The traditional way is to check if the name is defined. This way can be prone to error is you forget to add the "get" method

```scala
val authorName:String = if (author.name.isDefined) author.name.get else "name is empty"
```

A best way to check for a simple Option is GetOrElse

```scala
val authorName:String = author.name.getOrElse("name is empty")
```
<!-- more -->
In Scala 2.10, the fold method was introduced for Option. You could also use the following syntax:

```scala
val authorName:String = author.name.fold("name is empty")
```

It saves a little bit of typing, but not much difference, it might confuse those that are not familiar with it.

You could also use pattern matching on Option, which is generally discouraged, since it is rather verbose.

```scala
val authorName:String =
  author.name match {
    case Some(name) => name
    case _ => "name is empty"
  }
```

### Nested Options
Let's say our author variable is now an option:

```scala
val author:Option[Author] = ...
```

Using the *isDefined* method is getting rather verbose and error prone here.

```scala
val authorName:String = if (author.isDefined && author.get.name.isDefined) author.get.name.get else "empty"
```

You could use getOrElse here, but you would still need to check is the the author is defined. Still rather clunky.

```scala
val authorName:String = if (author.isDefined) author.get.name.getOrElse("empty")
```

The most idiomatic way to use nested options is map. The following will return the name if both author and author.name are present

```scala
val authorName:Option[String] = author.map(_.name)
```

Or:

```scala
val authorName:String = author.map(_.name).getOrElse("name is empty")
```

### Multiple Nested Options
#### Flatmap
Let's up the ante here.

```scala
case class Author(name:Option[Name])
case class Name(firstName:Option[String], lastName:Option[String])
val author:Option[Author] = ...
```

In this scenario, author, author.name, and author.firstName could be None. FlatMap to the rescue.

```scala
val authorName = author.flatMap(_.name).flatMap(_.firstName)
```

or

```scala
val authorName = author.flatMap(_.name).flatMap(_.firstName).getOrElse("first name is empty")
```

flatMap reduces the nested options, allowing us to retrieve what we need. No need to use any if statements, it is type safe.

#### For-Comprehension
Another way is to use a for-comprehension. It is a little more verbose, but allows us to make the code a little cleaner and easy to maintain. It is best used when dealing with multiple levels of Options.

```scala
val authorName:Option[String] = for {
  author <- author
  name <- author.name
  firstName <- name.firstName
} yield firstName
```

or

```scala
val authorName:String = (for {
  author <- author
  name <- author.name
  firstName <- name.firstName
} yield firstName).getOrElse("First name is empty")
```