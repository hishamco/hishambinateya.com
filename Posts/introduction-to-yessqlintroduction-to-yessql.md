---
Id: 29
Posted Date: 13/3/2017
Tags: YesSQL,Document Database,.NET Core
Slug: introduction-to-yessqlintroduction-to-yessql
---
# Introduction to YesSQL

YesSQL is document-based database that built using .NET Core, basically it's an interface over relation databases. YesSQL allows you as developer to define and create your documents and indexes using plain object CLR objects (POCO).

The good part of YesSQL is that uses any RDBMS to store the documents, which gives you all the power of SQL databases such as transactions, replication, reporting and much more. Nothing fancy .. it's pure SQL!!

Till now it supports variety of databases such as: SQL Server, MySql, Postgre, Sqlite, LightningDB and In-Memory.

### Yet Another Document Database .. Why Not!!

AFAIK there are many great document databases available in the market such as DocumentDB, MangoDB, RavenDB, CouchDB .. etc. One of the main issues for most developers - even I - that sometimes we need a free database especially while working on development environment, so this will reduce the options that you have in the market, because not all the databases are free. The second issue is not all the available databases are transactional. The last point is if you thinking about .NET Core document database that abstracted almost the things in your behalf, and do the right job for you.

IMHO If you don't care about all the points that I mentioned before, don't bother yourself, because YesSQL will be useless for you, unless you need to try it and dig into the source code :)

### YesSQL vs NoSQL

The main difference between YesSQL & NoSQL is YesSQL uses relational database "_because in SQL we (still) trust!"_ as Sebastien said, in other hand NoSQL is non relational, basically it uses other data structures such as key-value, wide column, graph, or document.

### How Can I Use It

YesSQL is simple to use in you application, so for those who work on NHibernate or EntityFramework you notice a lot of similarities in the concepts.

Let's see how is easy is that to work with YesSQL, first of all assume we have the following model:
```csharp
public class Person
{
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public char Gender { get; set; }
    public DateTime BirthDate { get; set; }
}
```
then we need to configure the data storage, so I will use SQL Server for this demo, feel free to choose the one that you like
```csharp
var config = new Configuration
{
    ConnectionFactory = new DbConnectionFactory<SqlServerConnection>(@"Server=.;Database=MyDB;Integrated Security=True"),
    DocumentStorageFactory = new new SqlDocumentStorageFactory()
};

var store = new Store(config);

store.InitializeAsync().Wait();
```
Now let's persist some data
```csharp
var person = new Person
{
    FirstName = "Hisham",
    LastName = "Bin Ateya",
    Gener = 'M',
    BirthDate = new DateTime(1984, 2, 4);
};

using(var session = store.CreateSession())
{
    session.Save(person);
}
```
if you notice we use establish a session using the method `CreateSession()`, which is similar to `DbContext` in EF, then we saved the newly create object using `Save()` method.

Similarly you can query or update any document
```csharp
using(var session = store.CreateSession())
{
    var person = session.Query<Person>().FirstOrDefault();
   Console.WriteLine($"{person.FirstName} {person.LastName}");
}

using(var session = store.CreateSession())
{
    var person = session.Query<Person>().FirstOrDefault();
   
    person.BirthDate = new DateTime(1984, 4, 2);
}
```
We are done!! we saw how easy it was to play with some of the YesSQL fundamentals, also you can dig into some advance features such as indexes.

Last but not least I wanna point that [Sébastien Ros](https://twitter.com/sebastienros) gave a great demo of YesSQL in Orchard Hardvest 2017 Event, I encourage you to checkout the video in the link below.

[![](https://aspblogs.blob.core.windows.net/media/antoinegriffard/Open-Live-Writer/Orchard-Harvest-Conference-2017-Day-2_E42C/YesSql_2.jpg)](https://www.youtube.com/watch?v=D42eK6CJjF4)

Finally I'd like to mention that YesSQL powers Orchard Core's data access, this will be a great promise for such database in Orchard vNext.

You can checkout the source code for this project from [YesSQL](https://github.com/sebastienros/yessql) repository on GitHub.

Happy Coding !!