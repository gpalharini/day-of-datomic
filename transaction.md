# Datomic's Transaction

### Introduction
Welcome to the third part of the Datomic training series. With the basics out of the way, we are going to talk about Datomic’s transaction model. We'll see how Datomic is an old-school ACID database and how Datomic allows you to group data together into entities. 

We will also discuss how Datomic handles identity and the related question of uniqueness. Finally, we will cover database functions and reified transactions. 

### Transactions
image

### ACID
image

Datomic transactions are **ACID**. ACID is an acronym, something that most SQL databases do, and there are four properties:

- **Atomic**: A transaction is Atomic. That means whatever the transaction does to the system, it either does it all or doesn't do anything. In Datomic a transactor is a set of datoms being added to the system. It is always what it is. Some of the datoms are retractions, they have the effect of making old things no longer true. But they are always additive, it is always adding datoms to the system. The A part of ACID means that if you make a transaction with 40 datoms in it, either these 40 datoms are going to be in the system after that transaction, or none of them are. 

- **Consistent**: The C in ACID stands for consistency. All the processes see the same global ordering of transactions. Everybody sees the things that happen in the same order. And Datomic is consistent in the old-fashioned ACID sense. If you look at SQL databases, they actually talk about transactions in isolation levels, where you can turn down the ACID guarantees. And why would you do that? Performance. Volume. If you want to be able to handle a large number of writes or distribute a large number of writes, then you will want to turn this thing down. Guess what: Datomic does not let you turn this thing down. In SQL words, isolation levels go from 0 to 3. 0 would be a nice name for chaos: you put some data in a system, and maybe something happened. 
 _"Datomic is not a good system for things we don't care about. Datomic is for systems that actually care about the data. We care about each individual piece of it. It is not like dumping log files."_
If 0 is chaos, 3 is what we call "serializable". That means the things that happened in the system have been serialized. They don't have to be serialized, but the highest level means that in the face of problems in the system, you can never tell that things were actually just serialized. There are all kinds of clever things to preserve that property while trying to allow parallel execution. Datomic transactions are just serialized. It is a single-writer system. All things happen in a global order because things only happen in one place.

- **Isolated**: Isolated means that you never see other transactions and stuff in the process that hasn’t been committed yet. This falls directly into being a single-writer system, there is nobody else working at the same time. Again: these properties are super easy when you make this particular assumption. 

- **Durable**: Durable means that, with Datomic, the results of the transaction are flushed through durable storage before reporting the transaction is complete. When you get a transaction request, you turn around and tell storage: "Store this". That does not mean that it is indefinably durable but is in the scope of maintaining ACID properties. If a meteor hits the earth, that doesn't mean that the data is still going to be there. But Datomic has done its job, it has durable storage. 

These are the ACID properties. It makes the software easy to write and to reason about. One of the ideas behind Datomic is that the massive write scale is not the only thing you might want to change about SQL databases.

### Assertion and Retraction
image
Transactions are always at the bottom, just collections of assertions and retractions. You can write them this way, you can write in a list form or a tuple form. This is to add or retract. After that, we write the entity ID, the attribute, and the value. Note that of the five things that actually get stored in Datomic, only four of them are present here: add or retract, entity, attribute, and value. What is missing? Time, or transaction. You can't add that because the transactor is in charge of that. The transactor has to do that so you don't get to specify that, you get to do this, and you will get back a report that tells you when that happened and where the transaction order is.

### Entity maps
image

You can make multiple statements about the same entity with a map. Quite often, especially if you are transacting data by hand, you are going to use the map form because it is more convenient to say multiple things about the same entity. If you say 6 things about the same entity, how many things do you have to make if you make a datom at the time? 6 x 4 is 24. Using the map 6 x 2 is 12. You just put attributes and values on a map. 

##### Lists vs. Entity Map
image

These can also be nested. So if you look at the **top**, you have a list of 4-tuple assertions: add, entity ID, attribute, and value. Looking at the **bottom** you have the equivalent map form. What is the thing that actually gets stored in a database? It is always datoms, because Datomic databases are a set of datoms. But in many cases, both on the input side and the output side, having this entity map form is convenient. And it is also closely aligned with what you are used to if you are coming from object-oriented programming.

### Cross Reference 
If you need to make a cross-reference, you can use the same `:db/id` twice, or more than twice in a transaction. Here we are creating "Bob", whose name is Bob, and we are creating Bob's spouse. We are doing both of them in the same transaction. We are also specifying the spouse relationship between each other, which means that I need to link them up, but these IDs don't exist yet. If I made these IDs and came back later I can use the lookup, so we can use the numerical value of the ID. But at the time I am making Bob, the transaction that is gonna make Alice hasn't run yet because it is the same transaction, and vice-versa. 

Here we are using Bob as the person's spouse. Person spouse is a reference attribute. We are putting a string value in it, and Datomic will see that it is a temp ID. That temp ID had better match the temp ID that appears as an entity ID; in this case, `:db/id`. `:db/id` is special, `:db/id` is not really an attribute. It is a way of linking the entity to this map. This map is attribute, value, attribute, value, attribute, value.

### Nesting 
image

The super-nice thing about these entity maps is that they can nest other maps. So here we are creating an order. This is making an order which is an entity that has 2 new line items: chocolate and whisky. Chocolate and whisky, in this case, are variables that were set earlier in the program that represent the IDs of those things that we want to order. Because it is nested, this will automatically make the relationships for you.

### Identity
image

How to keep track of who is who? There are several ways to do so and it is super important to understand the difference between them.

image

Datomic has its own notion of identity that is relative only to the database and you should treat it as opaque. It happens to be a 64-bit number but you shouldn't treat it as a number that you should do math with. It is an opaque identifier assigned by Datomic.

Every time you make a transaction it is either referring to existent entities by those numbers or new entities by those numbers. They are unique in the database. The record in the database is unique.

When you add external identifiers, you have the uniqueness that is semantic to your domain. There are a couple of different scenarios there:

- **External ID**: Sometimes you have an external ID that you're asserting that it is unique. But is not a naturally unique identifier like the UUID. So you will mark that as a `:db.unique/identity`, which is a schema property. Typically, that is going to be a string, a UUID, or a URI that came from an external system.

- **Assigning a UUID**: You might also want to assign a UUID. Imagining that you don't have this globally unique value being given to you. You don't have data coming out of a system, they already have a “uniquefier” on that. But you want to have a “ that is stronger than just system relative. You might want to assign your own global UUID. Again you make a `:db.unique/identity` attribute and use d/squuid, where squuid is an API for making those. But in the absence of that, any UUID or UUID generator works.

- **Programmatic names**: And finally, there are programmatic names, which are things like variable names in your program. Those programmatic names, in Datomic, are typically used to name schema attributes. But you can also use them for other things, if you have a set of enumerated types, like the title of a person, and you are allowing a set of titles, or you are doing products and they come in sizes, you might make an ident set of these things. 

The thing to know about `:db/ident` is that looking up something by ident is faster than looking at any other attribute in the database. It has to be. Because every time you make a query, the first thing you do is use the items to figure out what the actual thing is querying on this. So every query must do dozens of these item resolutions in order to start doing the query. 

What does this tell about items, how do they work? The items are stored in the database but in addition to being stored in the database, they are stored in memory and in every process in the system. Every peer, every transactor, and every item is stored in memory. 

If you go back to the analogy with a program, if you compile your program, your program has a symbol table on it. And depending on how that gets manifested in the compiled code, that might still be there in your compiled program.  If you have a program that hasn't symbols in it, it might be not able to compile because you'll run out of memory. Does that mean that your program can't manipulate a big piece of data? No. Just means your program can't have a million programmatic names. Datomic is the same way. It would be a mistake to use `:db/ident` for every entity in your system.

Let's imagine generating a random string or a keyword name for every entity in the system. Now you don't have a database anymore. What do you have? You have a big in-memory HashMap. That is not what idents are for. It is important to use idents in the right way. In Datomic there could be more than ten thousand idents in a single system. We do not want to have a 1-to-1 relationship between idents and entities. 

### Built-in tx Functions 
image

When talking about transaction functions, you want to have richer semantics than just asserting or retracting. The canonical example of this is you want to modify a value in the database based on its past value. In Datomic you don't have to take someone's 100 dollars and say they now have 110. You can deposit ten dollars and have the balance updated depending on what the value was before. This is what transaction functions are for: they allow you to say: 

_"I want to have access to the prior value of something in order to decide what to do"._  

That means that the transaction functions have to run in the transactor because Datomic is a single-writer system, it is sensitive semantically. 

##### :db/retractEntity
`:db/retractEntity` followed by an entity ID produces however many retractions are necessary to make everything about that entity go away or be declared as false - since we only accumulate. That entity and its subcomponents and their subcomponents and so forth.

##### :db/cas 
`:db/cas` stands for compare and swap. It takes an attribute, an old value, and a new value, and it says: this transaction should succeed in replacing the old value with the new value if, by the time the transaction runs, the old value is in fact the old value. It is a check that says: if that value is not the old value, that means none another value I was using is accurate. In this case, I should fail the transaction. Conceptually, CAS is one of the coolest concepts in software. It is a very powerful tool that comes up again and again.

There are all kinds of other things you might wanna use transaction functions for. Things like validating the presence of multiple attributes to say that a particular entity is fully formed. Or validating relationships between attributes saying that if an entity has this, the other thing has to have a certain value.

### Reified Transactions
image

##### :db/txInstant
Every transaction in Datomic is an entity. Just like all the other entities in the system. You have the entity ID, attribute, value, and transaction in a datom. That transaction value is a reference to an entity that gets created whenever a transaction happens. Which means that if you put 6 datoms in a system on a transaction, how many datoms are going to be actually added? At least 7. The one automatically added holds a value to the `:db/txInstant` attribute, which is the wall clock time on the transactor when the data was added.

The transaction identity is non-decreasing. So you can walk up to a bunch of transactions and figure out which one came first just by its numerical value, and that is fully consistent. Or you can ask when it happened if you trust the clock in your transactor by looking at the value for `:db/txInstant`.   

Also, Datomic will not let you cheat time. You cannot walk up to the transactor and say "it is 2014" for a certain bunch of data, then turn the clock back to 2012. The minute you try to assert a piece of data that is older in the wall clock time, Datomic will not allow it. We can't enforce that the clock isn’t cracked, but we can enforce that the clock only moves forward. If you accidentally screw up your clock, you are going to be stopped.

### Transaction Attributes
image

Here are two examples of using the transaction entity: 

##### Example 1: 

If you refer to the `:db/id` `'datomic.tx`, that name is special. Strings that begin with Datomic should not be used by you as temp IDs unless you are saying something about a transaction. Datomic says: make these attributes part of the transaction. When saying `:publish/at`, you are saying that for some other users of the system, I want this to be visible only as some date, which is `:publish/at`.

There is a need to enforce that when you have a query that says: find me transactions and filter out the ones that say they are not published yet. It's possible to do that because this information has been hung onto a transaction.

##### Example 2: 
The second example is overriding time. So you are allowed to set up the wall clock time. We are setting up the wall clock here to the first second in February of 2013. Typically the only time you are going to override `txIstant` is when importing data. 




