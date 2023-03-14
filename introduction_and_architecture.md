# Datomic's Introduction and Architecture

### Agenda
![Alt](https://ibb.co/4tZWX0M)
This is the agenda for this session:
- **What is Datomic?** This session starts with an overview of Datomic.
- **Information model**: We are also going to talk about the information model of Datomic. This is really the most important thing about a database. If the information model doesn't let you find out answers to questions you would like to ask about your database, then its operational characteristics are not gonna help you very much, and the mechanics of using it are not gonna help. 
- **Transaction model**: We will talk about the model of how the data gets into a Datomic database. These are the sourest transactions you will ever see. So this is a solid old-school transactional database.
- **Query model**: Then we will look at the query model. We are going to use the word query here in the broadest sense. We are going to look at Datalog, which is Datomic's query engine.
- **Pull**: We will also look at pull as Datomic has a model for the hierarchical selection of data. It's possible to do logic-based queries to find things, but it's also possible to say, "I already know what I want. I am interested in this particular thing and I just want to be able to pull a hierarchical selection of information about this entity, starting from a particular point." So pull is more of a navigation model and Datalog queries are more of a logic-based model.
- **Raw indexes**: Datomic also provides direct access to its indexes. It is possible to build your own tool on top of it and integrate it with any analysis system. 
- **Time model**: We will talk about the time model of Datomic, and this is probably the most distinctive feature of Datomic. Datomic is a pure accumulation system. It doesn't forget things it knew in the past, which seems to be a desirable property for a database. Once you get used to a database that works this way, you wonder why databases haven't always worked this way.
- **Operational model**: We are going to end the session by talking about the operational model of Datomic.

### What is Datomic? 
![Alt](https://ibb.co/FmvnJP4)
##### A complete rethink of databases
Datomic is a distributed database that is built from the ground up based on functional programming concepts, such as immutability. 

##### Agile
Datomic is **agile** in terms of its ability to move quickly in the direction that you want to go. It has the capability to tell you the day in which your past choices become the elephant in the room and dominate what you're able to do going forward. Datomic gives you a chance to build a system that you're going to be able to modify easily as you move forward. 

##### Robust
Datomic helps you build systems that are **robust**. Systems that do not fail in mysterious ways and that are easy to understand. When you have a database that only accumulates, you can say: "I would like to travel back in time to 4 days ago when we were making that horrible mistake, and I would like to look at the database right before we made the horrible mistake and I would like to look at the database right after we made the horrible mistake then try to reason about it".

How do you do that in a system that doesn’t just accumulate? It's a big mess. Basically saying: "Okay, let's figure out where the logs are and let's try to reconstruct from the operational logs what happened in the system while things were going wrong". But the actual thing that was there when it was falling apart and excrement was hitting the oscillator, it's gone. 

##### Powerful
Datomic is **powerful**. Well this should be totally straight forward and then, you know, it takes six days and this maven dependency over here causes that thing to break. And you know, this thing fails without giving an error that tells why it failed. But the library that was consuming, it just eats that and then, you know, passes up the chain, and the symptom, you know, happens over here. Those things make me feel not at all powerful. And so I would like the difficulty of my tasks to be related to the tasks themselves and not the tools that I use to try to solve problems.

##### Time aware
Datomic is **time-aware**. I want a database that can remember the past again. This seems blindingly obvious in retrospect, but there are a series of choices that were made in the 60s and 70s about how to model information and records that we’re just now getting over. 

### Agile: Universal Schema
![Alt](https://ibb.co/gSfZrKT)
One of the things that really helps Datomic to be **agile** is the idea of a **universal schema**. In a certain sense, Datomic is relational. But an actual relational database, a traditional SQL database, have you define 5, 10, 100, or 1000 different relations that manifest as tables. With Datomic you never define a relation. There is a single relation. It's a universal relation and the universal relation is of five tuples.

So if you actually just wanted to implement a really impoverished Datomic-like database in SQL, then what you would do is: you'd have a table that had five columns in it, entity, attribute, value, transaction, and operation.

The **entity** is the identity of the thing you're talking about, the attributes are a name identifying some property of that thing. The value is that property. For example:

"Stu likes pizza". _Stu_ is an **entity**, _likes_ is an **attribute**, and _pizza_ is the **value**. 

With those three, if you just stop there, you'd have what people call a triple store. And the big reason that triple stores came into existence was to get exactly the same kind of agility that we're talking about here. And triple stores start to look really nice if you're modeling something like an inventory database.

Let’s say you have an inventory database where you have thousands of parts that have tens of thousands of distinct attributes. But an individual part doesn't have 10,000 different attributes. Each individual part has some small set of attributes and somebody else has a different set. 

So what happens when you try to actually make relations in a traditional relational database to do this? It gets ugly pretty quickly. You end up with a table of 10,000 columns, all of which are knowable because any particular thing may or may not have those things.

The **universal schema** helps you model data when you don't get to dictate that everything has to show up in a box of a certain shape. A lot of our systems have that characteristic. Not all domains have things showing up in a box of a certain shape.

If you're processing log files, for example, everything shows up in a box of a certain shape: date, timestamp, process ID, etc -  and absolutely everything has that shape. But if you have a system where things occasionally have a different shape then this agility is super helpful. 

### Update in place
![Alt](https://ibb.co/9qt7PvQ)
We're coming from a world of **update-in-place** systems and you see this with most databases, but you also see this with programming languages. For decades we've done primarily imperative programming where data structures are updated in place. So if you have an array you can just reach into it and replace the fifth element with something else. And this has dominated the way computer science is taught, it's dominated the way you look at the top 20 programming languages by any measure. 

You can look at a lagging indicator like the [TIOBE](https://www.tiobe.com/tiobe-index) index, or a leading indicator like usage on open source. You will see that the dominant languages work this way. We know what they are: C, C++, Java, C#, but also Python, Ruby, etc. These languages are all updated in place, and updating in place makes programs difficult to reason about.

If I'm sharing something with you, and either one of us might update it, we have a coordination problem. I have to worry about what's going to happen. Am I gonna update it? Are you going to wait? What happens if we're both updating at the same time? If I want to distribute something, I'll need to hand it out with the promise that it might change later. Again, you have a coordination problem.

Concurrently accessing those things is hard when you access something that's updated in place, and this is important for databases. If something is updated in place, when you access it, you’d better get every piece of information you want eagerly because if you don't, what's going to happen when you come back five minutes later? It may have changed. You won’t have a sound basis for reasoning about it.

If you've ever heard the phrase **cache expiry**, you're dealing with update-in-place tech. Because what would the expiry policy be for data that didn't update in place? Infinite. You can go forever. You don't have to have a cache expiry on something that's not updating the place. You can say: "Look, this is what it is. It's never going to change so we can cache it as long as we want to". 

Interestingly database programming, it's gotten a lot more interesting in the last 10 years with the NoSQL technology. But most of the NoSQL solutions don't really touch this: they are modern in other ways, but they continue in many cases to be updated in place, and they also continue to be very data-oriented. So if you take one of the more popular NoSQL databases and you want to run it in the cloud, you have this additional problem that may be actually the most difficult thing about it. 

### Persistent data structures
![Alt](https://ibb.co/W0sqh4S)
Contrast this with building what is called **persistent data structures**, structures that preserve their past values. This is the way Clojure works. This is the way F# works in the Microsoft ecosystem, and this is the way Haskell works. People that are using these languages are having a much easier time building robust systems because they don't have to worry about coordination.

### Powerful
![Alt](https://ibb.co/0YmDn6d)
Then we go back to the notion of being **powerful**. If you have to coordinate over something unnecessarily or in a way that's unrelated to solving your problem, that's horrifically complicated. I'm trying to do something, but I'm worried about you reaching in and touching something which will break me - that has nothing to do with me solving my problem. I don't want to be thinking about that at all.

### Time aware
![Alt](https://ibb.co/Qf4m0Qn)
To get an idea of what the **time-aware** notion of Datomic looks like. I'll show you a couple of different ways you can look at data when you're working with Datomic. When you're querying or doing a pull or accessing the index, you’re always working with a database value. So you have a thing that is not the connection, you have a value, and that value is the content and state of the database at a particular point in time.

You have that in memory and it's never going to change. But given that value, you can say: "Give me a new value that is of a point in the past”. Now you have a value that has all the recent stuff filtered out. Or you can say: "Give me a value since some point in the past". Now you're saying: "I only want to see a sliver of what happened recently. So give me everything that happened in the last 10 hours but nothing else".

You can also get reports of transactions so you can watch a running system and tap into it. Because everything is accumulated only, that transaction report could have some pretty interesting things in it. It has in it the new **datoms**, the stuff that got added in the transaction. Even subtractions are accumulated only, so there are no datoms disappearing. There could be a transaction saying that datom is no longer true, but the value that used to be true is still there. 

You can also do speculative work. You can say: "I would like to know what the database would look like if this happened. But I'm not really ready to commit to this happening yet". And that's called **with**. _With_ is an operation that takes a database value, and returns a new in-memory update. This is not a real change to the database, but it has all the semantics of the database. You can say: "If I did this, what would happen?" And people have done some surprisingly interesting things with this, where they will build up a whole stack of transactions locally and do sophisticated analysis and validations to decide if they're happy. And then only submit the transactions after testing locally.

And then you can get what's called the **history view**, which is a timeless view. The analogy of looking at history would be moving from your source code to your Git repo, and doing a “git log” search where you can see both what's happening now and what was there _yesterday_, but not before yesterday, and then what was there before that and so on. That is the **basic model**.

### Operational model 
![Alt](https://ibb.co/Tmjrw1t)
Talking briefly about the **operational model**. This is where things start to differ from a traditional database. Thinking about a traditional data center SQL database, that database does everything about being a database: it implements storage, transactions, and queries. And that means it's challenging to make independent decisions about storage or transactionality, or reading, or whatever. It is already built to all happen in one place, and this is a thing that it will always come back to because it was designed to run in one place.

Try to build a system that runs all in one place, and then break it apart and distribute it without having thought about that in advance. And you'll discover the challenges, and you hear scary words like replication start to come into play now. What Datomic looks like is quite different. 

There is a service called the **transactor** that does transactions. It implements transactionality. It does not do querying and it does not do storage. This provides an independent place in your system where you can manage and scale and think about transactions. The transactor service currently also does **indexing**.

Then there is the **storage service**, where things are actually placed durably. That is a collection of concurrent processes and disks or RAM.

There are what’s called **peers**. Peers provide access to queries, and peers are the same distance from storage as transactors. Transactors do not live with storage. Peers do not live with storage, but they are just as close, which is why they are called peers. Everybody has equal access to the underlying storage in the system, and peers come in two flavors: 

- **Peer library:** There is the peer library, which is a JAR file that you put in a JVM-based application and it gives you in-memory access to query. So you can do query, pull, indexes, and all that at memory speeds.

- **Per server**On the other hand, there are also peers that are themselves servers. A peer server is a process (or a group of processes) running in this role that accesses servers and uses HTTP as its protocol.

As an application developer, you are going to do one of two things. You are going to write a JVM-based process, and you are going to embed the peer jar in that process. Or you are gonna write a process in an arbitrary language and use a client to connect to a peer server.

The latter is a more traditional architecture. It is easier for people to walk up to. Your role is to be a client. You call over to somewhere else and it does the work. Now, another characteristic of being split apart like this and of being accumulated-only is that memcached becomes a transparent addition. 

Think about what happens when you typically add caching to an application: You identify a hot query, such as looking up the user's ID and password because they're logging in all the time from their smartphone. So you write custom code to put that into the cache and then you have all those caching-related questions like how to stay in sync with the data. 

With Datomic, the thing that goes into memcached is just the raw indexes of the database, and this happens transparently. So all you do is walk up to a system and operationally say: "I want to put in a memcached layer of a given size with this many machines serving it". Nothing else changes.

How much code do you have to write? Zero. What do you have to do in order for that to get the hot stuff cached? Run your system. What do you have to change if your system's characteristics change and have a different load? Nothing. What happens if you wanted to serve a transactional load in and an analytics load from the same system and they had really disjointed hot sets? Run two Memcached clusters, and configure sets of peers to use one or the other Memcached cluster. What do you have to do to tune them to actually get the right data that's hot in them? Nothing.




