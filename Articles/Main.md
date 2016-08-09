# OpenYeller: Let's build from scratch a geolocated messaging WebApp
## Introduction

The purpose of these articles is to create a messaging system to send geolocated micro messages. Kind of geolocated tweets.  
This is done in a goal of demonstrating a couple of technologies and how they can be used together in order to build a project that can be tested and deployed easily.  
The application will be built on a microservices architecture using [Swagger](http://swagger.io) to develop the microservices in Javascript with [Node.js](https://nodejs.org) and [Express](http://expressjs.com/).   We will use also [Docker](https://www.docker.com/) to package the microservices, [Mongodb](https://www.mongodb.com/) as a main database and [Redis](http://redis.io/) as a cache system but also for Oauth authentication.  

Note that we will not tackle production problematics in this article like scalability, replication etc... Everything will be hosted on a single VM in Digital Ocean and nothing will be "production grade" (Meaning, no monitoring, no automatic restart, no logs, no exception catching, no unknown exception reporting etc...).  
But, this can be an introduction to a proper production architecture if you want to bring it further.  


## Prerequisites
### Knowledge
All you really need to follow these articles is to have a basic knowledge of Node.js and Javascript.  
It would help also to have a vague knowledge of the following:  
   - Mongodb
   - Redis
   - Docker
   - Swagger

You don't need to know much about these as this article can work as a tutorial too.  

### Softwares
You can work on Linux / Windows or Mac Os X. Almost any OS can do but you will need to have installed:  
- Node.js
- Npm
- Docker
  - Docker toolbox in case of Windows/Mac Os X
  - Docker / Docker-compose / Docker-machine for Linux
- A good text editor. I highly recommend [Atom.io](http://atom.io)

## Table of contents

  * [Part 1 - First Swagger microservice, Docker container creation and autobuild](OpenYellerPart1.md)
  * Very very Soon - Part 2 - Swagger project edition, Mongodb / Redis integration and finalization of the UserManager microservice
  * Very Soon - Part 3 - Deployment and creation of the remaining microservices
  * SOON - Part 4 - API Gateway
  * SOON - Part 5 - Continuous integration and deployment
  * SOON - Part 6 - Creation of the web interface

## Contacts
You can contact me on Twitter ![Twitter](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/TwitterBird-40x40.png): [Matthieu Vincke](https://twitter.com/MatthieuVincke)
