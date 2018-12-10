---
title: Using Automapper to improve performance of Entity Framework
tags:
  - csharp
  - entity framework
  - automapper
url: 69.html
id: 69
categories:
  - .NET
  - Best Of
  - Other topics
  - Tutorials
date: 2016-04-02 11:30:03
---

Entity Framework is an ORM technology widely used in the .NET world. It's very convenient to use and lets you forget about SQL... well, at least until you hit performance issues. Looking at the web applications I worked on, database access usually turned out to be the first thing to improve when  optimizing application performance.

### Navigation properties

The main goal of Entity Framework is to map an object graph to a relational database. Tables are mapped to classes. Relationships between tables are represented with **navigation properties**. ![ef](/images/2016/04/ef-300x191.png) The above example will be mapped to the following classes:

```csharp
    public partial class Article
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Content { get; set; }
        public System.DateTime CreatedDate { get; set; }
        public int AuthorId { get; set; }
    
        public virtual Author Author { get; set; }
    }

    public partial class Author
    {
        public int Id { get; set; }
        public string FirstName { get; set; }
        public string LastName { get; set; }
    
        public virtual ICollection<Article> Articles { get; set; }
    }
```

The highlighted lines declare navigation properties. Thanks to navigation properties, it's very convenient to access details of Article's Author. However, it comes at a cost. Imagine the following code in the view:

```html
    <table>
        @foreach (var article in (List<Article>) ViewBag.Articles)
        {
            <tr>
                <td>@article.Author.LastName</td>
                <td>@article.Title</td>
            </tr>
        }
    </table>
```

Assuming that ViewBag.Articles  is loaded with the below method, this code might turn out to be very slow.

```csharp
        public List<Article> GetArticlesOlderThan(DateTime dateTime)
        {
            return context
                .Articles
                .Where(article => article.CreatedDate < dateTime)
                .OrderBy(article => article.CreatedDate)
                .ToList();
        }
```

Unfortuantely, it will fire a separate SQL query to the database server **for each** element in the Articles collection. This is highly suboptimal and might result in long loading times.

### Lazy and eager loading

The reason behind this behaviour is the default setting of Entity Framework which tells it to load navigation properties **on demand**. This is called **lazy loading**. One can easily overcome this problem by enabling **eager loading**:

```csharp
            return context
                .Articles
                .Include("Author")
                .Where(article => article.CreatedDate < dateTime)
                .OrderBy(article => article.CreatedDate)
                .ToList();
```

Eager loading will cause EF to pre-load all Authors for all selected Articles (effectively, performing a join). This might work for simple use cases. But imagine that Author has 50 columns and you are only interested in one of them. Or, Author is a superclass of a huge class hierarchy modelled as table-per-type. Then, the query built by EF would become unncessarly huge and it would result in transfering loads of unnecessary data.

### Introducing DTO

One way to handle this situation is to introduce a new type which has all Article's properties but additionally has some of the related Author's properites:

```csharp
    public class ArticleDto
    {
        public string Title { get; set; }
        public string Content { get; set; }
        public DateTime CreatedDate { get; set; }
        public string AuthorFirstName { get; set; }
        public string AuthorLastName { get; set; }
    }
```

Now we can perform projection in the query. We will get a much smaller query and much less data transfered over the wire:

```csharp
        public List<ArticleDto> FastGetArticlesOlderThan(DateTime dateTime)
        {
            return context
                .Articles
                .Select(article => new ArticleDto
                {
                    AuthorFirstName = article.Author.FirstName,
                    AuthorLastName = article.Author.LastName,
                    Content = article.Content,
                    CreatedDate = article.CreatedDate,
                    Title = article.Title
                })
                .Where(article => article.CreatedDate < dateTime)
                .OrderBy(article => article.CreatedDate)
                .ToList();
        }
```

### Automapper

We improved performance, but now the code looks much worse - it involves manual mapping of properties which is in fact trivial to figure out. What's more, we would need to change this code every time we add or remove a field in the Article class. A library called **Automapper** comes to rescue. Automapper is a **convention-based** class mapping tool. Convention-based means that it relies on naming conventions of parameters. For example, Author.FirstName  is automatically mapped to AuthorFirstName . Isn't that cool? You can find it on **NuGet**. Once you add it to your solution, you need to create Automapper configuration:

```csharp
        static MapperConfiguration config = new MapperConfiguration(cfg =>
        {
            cfg.CreateMap<Article, ArticleDto>();
        });
```

Here we declare that Article should be mapped to ArticleDto, meaning that every property of Article should be copied to the property of ArticleDto with the same name. Now, we need to replace the huge manual projection with Automapper's ProjectTo  call.

```csharp
        public List<ArticleDto> AutomappeGetArticlesOlderThan(DateTime dateTime)
        {
            return context
                .Articles
                .ProjectTo<ArticleDto>(config)
                .Where(article => article.CreatedDate < dateTime)
                .OrderBy(article => article.CreatedDate)
                .ToList();
        }
```

You need to add one more line:

```csharp
using AutoMapper.QueryableExtensions;
```

And that's it. You've just improved readability of your code and made it less fragile to changes.

### Summary

Automapper is a very flexible tool. You don't need to rely on naming convensions, you can easily declare your own mappings. Additionally, we have just used just a specific part of Automapper - **Queryble Extensions** which work with ORMs. You can also use Automapper on regular collections or just on plain objects. 

I believe the problem I highlighted here is just a symptom of a much broader issue of **incompatibility of relational and object oriented worlds**. Although Entity Framework tries to address the issue by allowing to choose between eager and lazy loading, I don't think it is a good solution. Classes managed by EF being elements of a public API are a big problem. As a user of such interface you never know if a navigation property is loaded and whether accessing it will result in a DB query. 

Therefore, I advocate the use of mapped DTOs. This approach reminds me slightly of an idea called **Functional Relational Mapping** adopted for example by the Slick framework for Scala. I believe it to be a great alternative to classic ORMs. Some references:

*   [Lazy and eager loading in EF](https://msdn.microsoft.com/en-us/data/jj574232.aspx)
*   [Automapper configuration](https://github.com/AutoMapper/AutoMapper/wiki/Configuration)
*   [Slick: Functional Relational Mapping library for Scala](http://slick.typesafe.com/doc/3.1.1/introduction.html)