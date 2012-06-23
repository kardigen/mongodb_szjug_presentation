# Mongo DB
Szczecin Java User Group Presentation

## Agenda
* [Introduction](#introduction)
* [How to start](#how-to-start)
* [How to use](#how-to-use)
* [How to manage relations](#how-to-manage-relations)
* How to optimize
* How to scale
* How to monitor

All issues explained with examples and in 'hands on' way.

## Introduction
* Document oriented database. (json),
* No transactinos.
* Sesigned for large scale, high availability and robust systems.
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
        db.users.save({ login: 'zenon', name : 'Zenon Burak' } )
        db.users.save({ login: 'zdzich', forename: 'Zdzisław', surname:'Kanapka'})
        db.users.save({ login: 'joanna', forename: 'Joanna', surname:'Dark',
          contacts:[{email:'aska@hack.it'},{home:{street:'Bura',streetNo:666}}]})
        db.users.save({ login: 'kiler', forename: 'Jerzy', surname:'Kiler',
          contacts:[{mobile:'555222555'},{home:{street:'Kolumbijska',streetNo:12}}]})

### Advanced queries
        db.users.find({name:/^Z.*/})
        db.users.find({$or:[{name:/^Z.*/},{forname:/^Z.*/}]})
        db.users.find({'contacts.email':{$exists:true}})
        db.users.find({'contacts.home.streetNo':{$exists:true},$where:'return this.contacts[1].home.streetNo % 4 == 0'})

  For more info see [here](http://www.mongodb.org/display/DOCS/Advanced+Queries)

### Command findAndModify queries
  Useful for creating queues in database - see more [here](http://www.mongodb.org/display/DOCS/findAndModify+Command)

### Stored procedures
  There is way to execute some code on server side - very usefull when existaing database evoluting.
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
  
  1. Each post have list of comments. Comment has an field *creator* that refers to owner user. In this case we are interested to see who was a creator for each time when we are reading comments.
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
  