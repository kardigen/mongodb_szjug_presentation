# Mongo DB
Szczecin Java User Group Presentation

## Agenda
* [Introduction](#introduction)
* [How to start](#how-to-start)
* [How to use](#how-to-use)
* How to manage relations
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
}

  