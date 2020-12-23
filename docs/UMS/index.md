# User Management System (UMS)

[(Github Repo)](https://github.com/rishavbhowmik/videosystem_ums)

This microserivce is a rest API based system, designed to provide secured access to user data across the system.

## Rest API server

This server is build in NodeJS using Fastify framework. This server has CROS set to `*` for allowing easier development/testing of distributed microservices.

## Actions of the driver

### Establishing Connection to MongoDB database

We are connecting to MongoDB cluster using MongoDB official driver in NodeJS [`mongodb`](https://www.npmjs.com/package/mongodb). In this, we are using a custom-built class `DB` which can be used to ensure that all operations on the MongoDB clusters are only initiated once the connection is established to the Database, which also helps to prevent multiple connections on the database from a single server.

### User class

Class `User` provides a set of static functions to perform operations with user data, class `DB` is inherited class to `User` for establishing a connection and gain access to the MongoDB database.

Major member funtions:-

- *Register* new User
- *Login/Authorization* of user and return `user_token`
- *Verify* `user_token`
- *Retrive and return* user data
- *Update* User data

This class provides a set of function, with strict verification protocol to ensure an added layer of security, if the UMS server has some security flaw by any chance.

## Endpoints
