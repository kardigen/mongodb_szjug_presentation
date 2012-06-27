# Mongo DB
Szczecin Java User Group Presentation

## Agenda
* [Introduction](#introduction)
* [How to start](#how-to-start)
* [How to use](#how-to-use)
* [How to manage relations](#how-to-manage-relations)
* [How to optimize](#how-to-optimize)
* [How to scale](#how-to-scale)

All issues explained with examples and in 'hands on' way.

## Introduction
* Document oriented database. (json),
* No transactinos. Atomic operations on document level.
* Designed for large scale, high availability and robust systems.
* Structured as heterogenic collections of documents (schema-less databases).

See more [here](http://www.mongodb.org/display/DOCS/Introduction)

## How to start
* Mongo db have great documentation - [start here](http://www.mongodb.org/display/DOCS/Quickstart)
* Install mongodb from your favorite package manager - instrucions [here](http://www.mongodb.org/display/DOCS/Quickstart)
* Start mongo

        mongod -dbpath <db_dir>

* Start mongo CLI

        mongo

* Basic setup is done.

## How to use
### Create database - *test*
        mongo test

### Create your first collection (*users*) and put document into it
        db.users.save( { login: 'stefan', name : 'Stefan Kowadło' } )

### First query
        db.users.find()

### Add some more users
        db.users.save({ login: 'zenon', name: 'Zenon Burak' } )
        db.users.save({ login: 'zdzich', forename: 'Zdzisław', surname:'Kanapka'})
        db.users.save({ login: 'joanna', forename: 'Joanna', surname:'Dark',
          contacts:[{email:'aska@hack.it'},{home:{street:'Bura',streetNo:666}}]})
        db.users.save({ login: 'kiler', forename: 'Jerzy', surname:'Kiler',
          contacts:[{mobile:'555222555'},{home:{street:'Kolumbijska',streetNo:12}}]})

### Advanced queries
        db.users.find({name:/^Z.*/})
        db.users.find({$or:[{name:/^Z.*/},{forename:/^Z.*/}]})
        db.users.find({'contacts.email':{$exists:true}})
        db.users.find({'contacts.home.streetNo':{$exists:true},$where:'return this.contacts[1].home.streetNo % 4 == 0'})

  For more info see [here](http://www.mongodb.org/display/DOCS/Advanced+Queries)

### Command findAndModify queries
  Useful for creating queues in database - see more [here](http://www.mongodb.org/display/DOCS/findAndModify+Command)

### Stored procedures
  There is way to execute some code on server side - very useful with existing database evolution.
  Consider that we want to update all users that have old *name* property to new approach with *forename/surname* fields.
  
        function fixName() {
          db.users.find({ name:{$exists:true}}).forEach( function(obj) {
                      var nameParts = obj.name.split(' ');
                      obj.forename = nameParts[0];
                      obj.surname = nameParts[1];
                      delete obj.name;
                      db.users.save(obj);
                     } ); }
        db.eval(fixName)
        
  For more info see [here](http://www.mongodb.org/display/DOCS/Server-side+Code+Execution).
  
## How to manage relations
### 1 - 1 or 1 - n strong relations
  This kind of relations should be modeled as embdded documents as we can see in case of *contacts* in users collection.
  
### n - n relations
  In this case we should consider business model needs.
  Good example of such relation is posts in Newspaper and user's comments. We have to consider two directions of relation:
  
  1. Each post have list of comments. Comment has an field *creator* that refers to owner user. In this case we are interested
    to see who was a creator for each time when we are reading comments.
  2. We are looking for comments that was created by specific user.

In this case we should check or assume which situation occurs more often. Lets assume that we are more interested that
the first case is more important. So we can model it like this: See example:
  
        db.posts.save({title:'Some very nice article', text: 'Very long long text',
          comments:[
            {creatorId:'stefan', comment:'eeee takie tam...'},
            {creatorId:'zenon',  comment:'Bomba czad bombelki!!!'}]})

In ther other case it would be better to model it like this:

        db.users.update({login:'zdzich'},{ $push: {
          comments: { postId:'some id'}, comment:'Hahaha - ale ubaw!' }})
  
## How to optimize
  We can do optimisation on two levels:
  * model level,
  * database level.
  
### Model level optimisation
  This kind of optimisation is dependent on the way we will use database. Sometimes optimisation
  in scalable database such mongo DB mean to denormalize datamodel and duplicate data. For instance
  in example above if we would like to have comments with full name of creator, we can duplicate
  user name in comments instead of finding it from relation to users collection. See how the posts
  can look like in this situation:
  
        db.posts.save({title:'Some very nice article', text: 'Very long long text',
          comments:[
            {creatorId:'stefan', name : 'Stefan Kowadło', comment:'eeee takie tam...'},
            {creatorId:'zenon', name : 'Zenon Burak', comment:'Bomba czad bombelki!!!'}]})

### Database level - indexes
  Indexes works in similar way to RDMS systems. But the best way to find out what indexes we need
  you should use *explain* method on real queries which you are using in your system.
  
  Let's assume that we want to implement login view and the query for *stefan* can look like:
  
        db.users.find({login:'stefan'})
  
  Try to use *explain* method to test query execution.
  
        db.users.find({login:'stefan'}).explain()

  You can see that mongo have to scan all db enteries to find appropriate one (nscannedObjects parameter).
  Let's set index on login and check again.
  
        db.users.ensureIndex({login:1})

  As you can see, the scaned enteries droped to 1.

  Now let's assume that we want to have suggestion box for finding users by name or it's part.
  So the query can looks like:
  
        db.users.find({$or:[{surname:/.*an.*/i},{forename:/.*an.*/i}]},{surname:1,forename:1}).sort({surname:1})
  
  _NOTE:_ case insensitive sort problem - vote [here](https://jira.mongodb.org/browse/SERVER-90)
  
  To find out what indexes we need, use *explain* method:
  
        db.users.find({$or:[{surname:/.*an.*/i},{forename:/.*an.*/i}]},{surname:1,forename:1})
          .sort({surname:1}).explain()
  
  Then try to set index on *forename* field and check what happend in explain query again.
  
        db.users.ensureIndex({forename:1})
  
  You can see that there is no difference. It is because the $or query operator.
  But what if we add index on *surname* field?
  
        db.users.ensureIndex({surname:1})
  
  Maybe you will be disaapointed but it looks like worse then without index at all. Simetimes indexes are very tricki.
  The simple and powerful solution in this particular case is to have another field *fullname* and appropriate index for it!
  
## How to scale
  *Sharding* is the keyword - see more [here](http://www.mongodb.org/display/DOCS/Sharding)

        mkdir -p db/a db/b db/conf logs
        mongod --shardsvr --dbpath db/a --port 10000 > logs/shard-a.log &
        mongod --shardsvr --dbpath db/b --port 10001 > logs/shard-b.log &
        mongod --configsvr --dbpath db/conf --port 20000 > logs/conf.log &
        mongos --configdb localhost:20000 --chunkSize 1 > logs/mongos.log &

  Do some db configuration

        mongo

        use admin
        db.runCommand( { addshard : "localhost:10000" } )
        db.runCommand( { addshard : "localhost:10001" } )
        db.runCommand( { enablesharding : "test" } )
        db.runCommand( { shardcollection : "test.users", key : {login : 1} } )

  Import some bid dataset

        mongoimport --host localhost --db test --collection users --type csv --file test.csv --headerline --upsert

  Login to db admin and check shared status
  
        mongo admin
        db.printShardingStatus()


