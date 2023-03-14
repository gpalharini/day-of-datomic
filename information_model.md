# Datomic's Information Model
In this lesson, we will discuss the Information Model. It will cover Datomic's preferred data notation, EDN, along with the fundamental unit of data in Datomic, the datom. We will also talk about Datomic’s ideas of databases, entities, and schemas.

image

We are going to look into five different things:
- **Notation**: When talking about notation, we talk about EDN, which is the textural serialization of data. It's the thing that you are looking at when you're typing Clojure programs.
Datoms: Datoms are a five-tuple of entity, attribute, value, transaction, and operation. 
- **Databases**: Mathematically, it is a set of datoms. A database value is a set of datoms that only ever grows, with the rare exception of the need for excision when required by a business rule.
- **Entity**: An entity is fiction. Everything that was just mentioned is facts, however, an entity is a little bit of a fiction. And the fiction is: if you have a bunch of datoms that are all about the same identity, then you could envision them as a map, an object. The thing that has a bunch of values in pairs. In this rationale, the identity that is you is your first name and also your last name - two attributes. Those are all facts about the same entity. You could envision them as a map. And this is interesting because the translation from datoms to entities is purely mechanical, and this matters a lot, because in other domains (like in a SQL database) if you wanna translate something to a map-like view, you would have to do what's called object-relational mapping (ORM). The reason why it is called mapping is that it is not a mechanical translation. It requires you to make decisions and the decisions that you make might be good in some contexts and bad in others. The decisions represent the work that you have to do because you don't have a universal schema.
- **Schema**: You do have schema, but it is not the same kind of schema that you may be used to in other database settings.  

### Notation: edn
image

Datomic’s preferred notation is EDN. If you squint a little bit in an EDN, what is it? It is an object, but what is it? You can imagine that when creating Datomic we could use something simpler like JSON, or TXML. We tried to make JSON work for this job because if JSON worked for this job, do you know how much easier our life would be? EDN wouldn't have to exist. All the stuff we did about encoding data in the Clojure ecosystem or Datomic ecosystem wouldn't have to exist. The fact that JSON is not appropriate for this job because some things are missing, look what EDN has in it: 

##### EDN Characteristics
image

EDN has strings, characters, numbers, booleans, nil, symbols, and keywords. Which of those things is JSON missing? JSON is not very good in integers and floating and can't tell numbers apart. It is not possible to construct it on top of something that can't tell apart the basic category division and numbers. 

Symbols and keywords are types that represent the name of things. That matters a lot. Go and look at any DSL in Clojure and try to do the DSL forbidding yourself from one of the things above, like doing the DSL and not being allowed to use symbols and keywords. It’s close to impossible. 

##### EDN Collection Types
image

Edn has four collection types:
- **List**: The list represents things that you transverse from the front.
- **Vector**: Vectors represent things that you can transverse in any random way, but are sequential.
- **Maps**: Things that represent lookups or maps are associative.
- **Set**: Things that represent membership sets are also with brackets but proceeded by the pound. 

There is a broader set of collection types. Having said that, EDN does not aspire or pretend to cover every data you might wanna have. The escape for that in Clojure is EDN extensibility.

##### EDN Extensibility
image

You can have #name followed by something that Clojure can understand. This is an important rule. That says: _"I know that in my domain this has some special meaning"._ And then your program can add extra meaning based on that name.

There are a couple of important things to understand here: your program **can**, but other programs **don't have to**. If someone makes awesome data literal and this person passes it to another person, and this other person doesn't know what that name means, it is possible to serialize it, deserialize it, get to the content of it, pass it on to another people - even if without the special meaning your program would give to it.

##### Built-in Tags
image

To prove that this works we’ve built a few tags into EDN. This could have been built-in value types but we figured why not have a few tags that we use when implementing a system. So instance in time is #inst followed by a string compliant with [RFC 3339](https://www.ietf.org/rfc/rfc3339.txt0). An UUID is #uuid followed by a string compliant with [RFC 4122](https://www.ietf.org/rfc/rfc4122.txt).

Why does this matter to you as a Datomic programmer? If you are using Datomic from Clojure, then all the print forms of data are going to look like this. If you are using Datomic from anywhere else, then it is going to map to what your language is capable of, which is going to vary in degrees of quality depending on what the language can do.

### Datoms
image

Datoms are atomic facts. They are immutable, they never change. They are **5-tuple**: 
- Entity
- Attribute
- Value
- Transaction
- Operation 

Going straight to examples:

##### Example Datoms 
image

Here we have some example datoms: 
- e
- a
- v
- tx
- op

As of the time of transaction 1008, Jane likes broccoli. At that same time, Jane also likes pizza. But then at the time of transaction 1148, Jane does not like pizza, and now that's false. This is conceptually the model but is not exactly what happens.

##### Entity IDs  
image

This is what happens. What is different here? 
- **Entity**: Everything in the e column is a number, entities are stored by number.
- **Attribute**: Everything in the a column is a number, so every attribute in the system is stored as a number.
- **Values**: In the v column, if there are references from one entity to another, we store numbers. 

If you go back and look at this, this is the entry to the question "What happens if you build a system and you don't care about the performance of the system and what happens if you build a system you do care about performance?” We want this to be stored efficiently. If you look at Datomic's database, this has a lot of compression on top. The objective is to make you not terrified of storing data.

### Database
image

A **database** is just a set of datoms, and its universal relation. It stores efficiently. How do we store it efficiently? There is representation - compacting things into numeric identifiers and there are layers of compression. That matters because the database is stored in 4 or 5 sort orders. It is redundantly stored. All of which only accumulate. Datomic gives you the benefits of accumulating only without some of the performance challenges of physically putting data in the next sector, and then the next sector, and so on.

##### One database, many indexes
image

The benefits of storing these different indexes are multiple. One of the benefits is that you don't have to think about defining indexes, you're not going to have that phase on your project. The things are pretty much indexed for a lot of common use cases automatically. 

The second thing it does is that it doesn't make you choose a database based on what questions you wanna have. So none of the whole NoSQL flows store the data in a way that optimizes a particular question only. 

If you look at these common scenarios, Datomic already has indexes that cover each of them.

Datomic indexes are named by the dominant sort order:

- **AEVT**: The AEVT index sorts the entire database by A. All the first names for all entities are together. 
- **EAVT**: The EAVT sorts it by the entity. All information about a given entity is together.

If you want a universal schema, you want A leading. If you want something that is like a traditional SQL database, that is row-oriented, that is E leading. 

- **AVET**: Similarly to AEVT, the AVET indexes store data based on attribute and value. It’s useful for analytics such as looking at all the prices in the system - not caring about what item costs that much.
- **VAET**: In a graph database, you want to be able to navigate from one entity to another. You want to be able to say "given a musician, what albums does that musician make?" or "Given that album, what tracks does it contain?" or "Given that track, is that track also on another album?" and so forth. The VAET index only applies to references to other entities.

People want graph databases for a couple of different reasons. If the dominant reason is to be able to do this kind of navigation, from one entity to another, then Datomic competes effectively in a worldwide consideration.

What Datomic does not do that most graph databases do is it does not label edges. Thinking about the way a graph database works: some nodes have edges connecting to another node. Well, that edge connection in a graph database is another thing you can hang properties on. If you want to do that, Datomic storage is not efficient. You have to reify that edge as an entity. Graph databases are designed for that.

It is important to know if you want the latter or both of those things, you want a graph store and you have to trade up. Datomic is probably not a good fit. 

But if all you care about in a graph store is to be able to navigate through relationships between entities with their attributes, Datomic is a great fit.

### Entities 
image

Entities are associative. Entities presume that they have some identity and they associate a bunch of key-value pairs with that identity. This is exactly like objects in object-oriented programming. In Datomic you can infer an entity by just grabbing all datoms that share **e** and say "that is an entity".

Entities have only two places to hang things: keys and values. They're map-like. So we have a 5-tuple thing we are smushing down to 2 tuples. How do we do that? How do we preserve all this information and then smush it down?

There are two things: 
- **1:** We get **e** “for free” because it is implicit in the entity. You just need to know that about your entity. 
- **2:** Then, transaction operation vanishes because entities are point-in-time. If I change my address in Datomic from "North Carolina" to "Texas", then I can look at the entity as in the time I am in Texas and I say _"I am in Texas"_. I can look at the entity at a point in time before that and say _"I am in North Carolina"_. Can I build an entity on top of the history database? No, it wouldn't make sense. It would say something like: _"I live in Texas and in North Carolina"_. You can't ask the history database for an entity. 

Entities have a superpower which is bidirectional navigation. In an entity, you can say: _"Give me an attribute"_. You can look back up and ask: _"Who is related to me through this thing?"_ Which is something that objects cannot do. Think about how object-oriented programming languages work. If you make an object, you're modeling an address book. The address book says _"this person has this address"_. Maybe the way you model it knows that the person's spouse and children all have the same address. But you can't from the object say: _"Give me all the things that point to me"_. Or _"Give me all the people who have me in the address"_. You can't navigate in that direction because those pointers don't exist. That is not how object-oriented programming languages work.

But it's how Datomic works, we have an EAVT index and we have a VAET index, so you can always navigate relationships in your system in any direction that you want. 
 
 ##### Tx Entity Map
image

Here is an example: entity 1001 is "Jane". Attribute 64 is "likes" value 1002 has the name "broccoli" and value 1003 has the name "pizza". You can’t take those datoms and mechanically convert them to that map. You need the database value in hand to do that because you need to figure out what 64 is and what 1002 is and so forth. But with the database and those datoms you can mechanically convert them to that map with no impedance. 

This is huge because this means that if you write an object layer on top of Datomic, it is not an object-relational mapping. It is a direct reflection of the data in the objects. 

You have to make choices like what is the cardinality of this relationship and if I am modeling something that goes to a joint table, how do I know where that table is? All those things are things that you have to say. Those are structural impediments. There is no impedance like that with entities.

### Schema
image

Schemas add power to your system. And I will conceptually use the word schema before we talk specifically about the mechanics of reifying it and putting it into the system. An information system has a schema whether or not the tools reify it. If you build an information system entirely on top of Java properties files and some navigation on the file system, that represents an implicit schema that you could describe in a programming language. The fact that you choose not to is entirely up to you, but the schema is there.

A schema is a model. When do you add and have a reified schema? When your database allows or requires a schema it’s giving you a model to which you are going to apply your data. The purpose of that model is to give you power.

When people say _"schemaless database"_ what they are saying is how much work you have to do to make the schema. Which might be 0 because might be implicit. That is possible. You can have a database where the schema is implicit and there will be no specific work you will have to do. 

I don't want schemas to force me to say things that I don't feel good about. For example, on tables in SQL databases, I commit to saying these 12 things always come together, and eventually, I discover a corner case where 2 of those things don't come together. I don't want that but I do want some power that comes out of declaring a schema. 

Power in Datomic schemas comes at a fairly small price, and the price is that Datomic schema is not to find relations. There is only one relation, the universal relation. We are not going to make a table of users or a table of items.

But schema does define the semantics of individual attributes without saying anything about who may or may not possess values for these attributes. This is a more granular statement and still gives you some power. So what kind of power can we get? 

##### Common Schema Attributes
image

Schema lets you give things an **ident**, which is a name or a programmatic name. That is valuable. We don't want to refer to attributes things like "number 64". We want to refer to them as "first name". 

Schema allows you to specify a **value type**: names have to be strings, prices have to be represented by decimals, and those kinds of things. Those will be important to the database.  

Schema allows you to specify the **cardinality** of the relationship between things. You can say that a user has only one email address. But we shouldn't, because we know that is not true, however, we can say that the user has only one primary email address. You can make statements about cardinality _one_ or _many_.

When you do all that, again, you should ask what the database gives you for making you do that.

Declaring `db/ident` the database says you get to talk about things with the programmatic names instead of numbers.

With `db/valueType` it says that you're going to say these attributes have a certain granular atomic type.

`db/cardinality` says that you can tell how many of something another thing or someone can have. What is the database going to do for me with that? Whenever something is cardinality one, if I assert a value for it the old value will be automatically retracted. 

`db/unique` says this attribute’s value is unique to one entity. Like only one person can have this primary email. What could the database possibly do for me with that information? There is a couple of interest things that it could do and Datomic provides 2 of them:
- **1:** You can refer to an entity by any attribute of it that is declared as “unique”, meaning it serves as an entity ID. 
- **2:** It is also possible to say that is not alowed that unique value to be stored a second time. This prohibits you from merging things, but you do want to enforce uniqueness.

`db/isComponent` shows ownership of things and prevents you from orphaning stuff. For example, if you have a child, your child is not a component of you. Your family is not a component of you. But your arm is a component of you. If I go, my arm goes with me. It is designed to facilitate managing entities in the system without making orphan entities. And it efficiently defines something that you care about. So when you pull an entity, its components are automatically going to be pulled.

Every single one of these things, except for `db/valueType`, you are allowed to change at any time. If you think about it, it sort of makes sense because all these other things are potentially compatible with the past things you said. You can take something that is not marked as unique and assert that it is unique and Datomic can go and check, then it will say: _"Yeah, everything you already said about that is unique so we’ll start treating it as unique right now"_. So you can be very agile here.

Names in Datomic are namespaced. When you make a mistake you can choose a new name and make a new name attribute, you can even rename existing attributes. So you can rename away the ugly mess you made.

##### Stories
image
The idea is that there are things called stories, which are stuff on the internet that we want to be able to gab about. A story has a **title**, which is a string. It has an **URL**, which is the actual place on the internet that it is. It has a **slug**, which is a unique identifier for you, so we can make URLs in our system. And it can also have **comments**. 

Notice that the attributes are namespace-qualified. The slash `/` is `namespace/name`.  In Datomic you can imply that the attributes go together by the namespace qualifying it. 

Stories can also have comments, and we will put that in a different namespace. 

##### Schema is plain old data 
image

This is what one of those looks like in an actual schema. So the `story/url` is a string and it has a cardinality.

##### Users
image

Comments have an author, which is a reference to a user entity, and comments can have other comments on them. Because of Datomic's time model, you can get the whole tree-like ordered view without doing anything. It just falls out because you can look at any comment and figure out when it was made. You can sort them without having a model in that.

##### "Types" do not dictate attrs
image

Notice that comments can be on stories or they can be in other comments, so that is why we choose a different namespace there. 

##### Schema in console
image

You can browse the schema directly in the console, so over on the left side of the console, when you connect to a database, you can just open up the schema and see that it is done as a tree by the namespace prefix. Again: the namespace prefixes are not semantic, they are just your own guidance to yourself.


