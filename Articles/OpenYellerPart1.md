OpenYeller Part1  
_More info on the project_: [OpenYeller](https://github.com/matthieu-vincke/OpenYeller.com)
---
# First Swagger microservice, Docker container creation and autobuild

The first part of the OpenYeller articles will define the architecture and start implementing it.  
The goal is to create the first microservice and to add it to a Docker container. We will also see how to trigger an automatic build of our microservice on Docker Hub.  

## Table of contents

 * [Changelog](#changelog)
 * [Project architecture](#project-architecture)
 * [Project skeleton and first Swagger microservice](#project-skeleton-and-first-swagger-microservice)
 * [Creation of the first Docker container](#creation-of-the-first-docker-container)
 * [Docker Hub and automatic container build](#docker-hub-and-automatic-container-build)
 * [Conclusion of the first part](#conclusion-of-the-first-part)
 * [Resources](#resources)
 * [Contacts](#contacts)

## Changelog


| Version    | Date        | Modifications by  | Details                      |
| ---------- |-------------| ------------------| -----------------------------|
|V1.0        | 2016-08-09  | Matthieu Vincke   | First version of the article |


---

## Project architecture
### Diagram

The global architecture of OpenYeller is simple.  
We just have one container by microservice, one Mongodb container and one Redis container. On top of that, one API gateway will manage the routing of the client requests to the corresponding microservice.  
Everything will be deployed on a single virtual machine.  
We will not discuss here about replica sets or even clusters for the databases. It would be very interesting but it is not the point here.  
You can find a lot of articles on the web about how to create Mongodb clusters using Docker, same thing for Redis.  
Also, we will not manage multiple containers: we will have just one instance of each container to avoid to have to manage load balancing between them. You can easily find articles about how to do that using Nginx if you wish.  
Let's have a look at our architecture:  

![ArchitectureDiagram](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/Diagram.png "Architecture Diagram")

Mongodb will be used to store the users details, their profile and the Yells (remember, the Yells are the geolocated micro messages)  
Redis will be used as a cache system for our API requests and for oauth authentication using Swagger plugins (refers to the part2 of the article: "Configuration of Swagger plugins").  

Let's see in deeper details the microservices.

### Microservices

Microservice                | Description
----------------------------|-----------------------------
UserManager         | In charge of the user sign-up / sign-in.
UserProfiles        | Manage the users profile: avatar / description etc...
Yells               | Will handle all the logic links to the Yells

Each microservice will expose a RESTful API and will communicate with each other using this API. Each of them will be embedded inside a Docker container and will be deployed on the same virtual machine.  
Docker will assign an IP for each container. Hence, each microservice will have its own internal IP and its own communication port.  
Because of this, it could be difficult for the client to communicate with the correct microservice: it would have to keep in memory the webservices available for each microservice and to communicate with the correct microservice when needed.  
To ease the work and have just one point of entry, we will introduce the API Gateway.  

### API Gateway

The API Gateway is in charge of registering the services available on each microservice and to proxy the requests to the microservice that can handle them.  
Using an API Gateway has a lot of advantage. The most important being that the client doesn't have to care about the complex architecture behind.  
The drawback is obviously that it becomes a point of failure: if the gateway crashes, nothing is available from the outside...  
As already explained, the goal of this article is not to do "production-proof" quality, we will not spend time to armor the code of the API Gateway (or the code of the microservices...) too much. But, if you plan to use an API gateway in production, keep in mind that this is a critical component.  

### The interface

The interface will be created during the part 4 using Angular2 and MaterializeCss. It will start from the starter: [Angular2-Materializecss-Starter](https://github.com/matthieu-vincke/Angular2-Materializecss-Starter) and will be deployed on a separate virtual machine.


## Project skeleton and first Swagger microservice
### Project skeleton

Now that we have a good idea of the architecture, let's start to create the following folders arborescence:  

```
|-- OpenYeller
  |-- Microservices

```

Microservices folder will contain all the microservices that we will create: one folder for each microservice.  
Because each microservice will be embedded in a Docker container, we will not share the node modules amongst them even though they will use the same kind of modules. Meaning we will have a node_modules folder in each microservice folder and not a global node_modules folder on the root of the project.
Indeed, during each container creation, Docker will be in charge of doing the "npm install" to install all the modules declared in the package.json file. Because of that, we will not "versionize" the modules so, we will have to be careful to indicate clearly their version in the package.json and to rely on NPM (which may sounds audacious, I know... [npm_left_pad_chaos](http://www.theregister.co.uk/2016/03/23/npm_left_pad_chaos/))  


### What is Swagger?

Better to quote the reference http://swagger.io/:  

```
Swagger is a simple yet powerful representation of your
RESTful API. With the largest ecosystem of API tooling on
the planet, thousands of developers are supporting Swagger
in almost every modern programming language and deployment
environment. With a Swagger-enabled API, you get interactive
documentation, client SDK generation and discoverability.
```

Swagger is a tool that will help us to define the APIs of our microservices.  

**Why Swagger?**  
The API documentation is what matter the most for a microservice.  
Ideally, the API documentation should be the only documentation of a microservice. One shouldn't need anything else to understand the microservice and what it does.  
That's what we will try to achieve for OpenYeller: each microservice API will be designed using Swagger so that we will have a proper documentation like this one by example: http://petstore.swagger.io/#/pet  
But, Swagger is not just about the documentation.  
It generates also automatically the code and unit tests skeletons so that we just have to write the remaining code.  
Put simply, Swagger ease our life by helping us to develop microservices properly.  


### Swagger installation

Installation of Swagger is really easy. Simply run in a console:  

```
npm install -g swagger
```

Using the -g flag, you will have access to the "swagger" command from anywhere on your computer.


### Create the Swagger project

Let's now use Swagger to create our first microservice: the UserManager.  
Inside the "OpenYeller/Microservices" folder, run:  

```
swagger project create UserManager
```
When asked, choose the option `express`  

![SwaggerCreateUserManager](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/swaggerCreateUserManager.png "Swagger Create UserManager")

Swagger should end with:

```
Success! You may start your new app by running: "swagger project start UserManager"
```

Simply type what Swagger suggest:  
```
swagger project start UserManager
```

![SwaggerStartUserManager](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/swaggerStartUserManager.png "Swagger Start UserManager")

To reach the service, go with your browser on http://127.0.0.1:10010/hello?name=Scott (or use the curl command as suggested)
You should see the message: "Hello, Scott!"   

Well done, your Swagger project is created!  

### Swagger project folders tree

Let's now check the folder arborescence created by Swagger.  

```
|-- OpenYeller
  |-- Microservices
    |-- UserManager
      |-- api    
        |-- controllers
        |-- helpers
        |-- mocks
        |-- swagger
      |-- config    
        |--
      |-- node_modules  
        |--
      |-- test    
        |--
      |-- app.js
      |-- package.json
      |-- README.md

```

As you can see, Swagger generated multiple folders and files in UserManager.  

- **api**: contains the definition of the API.
  - _controllers_: The controllers will be in charge of managing each request and to reply to it. Usually, we have one controller for each request group. Here, by default, there is the controller "hello_world" that is in charge of replying to the /hello get request we did in the browser earlier.
  - _helpers_: Helper functions are called before the controllers and are used by the Swagger plugins like OAuth/Quota/Cache etc.. We will use this in the part2 of the article. [MoreInfo](https://github.com/apigee-127/a127-documentation/wiki/Helper-functions)
  - _mocks_: when we define an API request using Swagger, we define the input and output formats. Given these, Swagger can generates automatically mocks to simulate the requests. If we want to override the default response given by Swagger, we can write a mock controller in this folder. We will not do that in our project. [MoreInfo](https://github.com/apigee-127/a127-documentation/wiki/Mock-mode)
  - _swagger_: contain the definition of the API in a single Yaml file. This is the file that we edit using the Swagger editor (see after)
- **config**: contains the default configuration
- **test**: the unit tests. One is created for demo purposes to test the "hello_world" controller. As you can guess, one file will be created for each controller.
- **node_modules**: the node modules needed by Swagger
- **app.js:** the entry point
- **package.json**: contains all the modules needed and their version + the description of the project that we will have to change
- **README.md**: The place to put some useful information.

### Edit the project

We can edit the API using:
```
swagger project edit UserManager
```
It should open a browser window and show the beautiful editor:

![SwaggerDefaultEditor](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/SwaggerDefaultEditor.png "Swagger Default Editor")

But, for now, we will keep it as it is. You can quit the Swagger editor.  
Let's start by integrating the SwaggerUi tools so that we can see our amazing API documentation.  
Indeed, by default, the documentation is not available, we need to integrate SwaggerUI for this.  
Open app.js in your text editor.  

Inside the callback of SwaggerExpress.create:  
Add the following:  

```javascript
app.use(swaggerExpress.runner.swaggerTools.swaggerMetadata());
app.use(swaggerExpress.runner.swaggerTools.swaggerUi());
```

To have:  

```javascript
SwaggerExpress.create(config, function(err, swaggerExpress) {
  if (err) { throw err; }

  app.use(swaggerExpress.runner.swaggerTools.swaggerMetadata());
  app.use(swaggerExpress.runner.swaggerTools.swaggerUi());

  // install middleware
  swaggerExpress.register(app);
 [...]
 });
```

Now, let's restart our project:

```
swagger project start UserManager
```

Open a browser on: http://127.0.0.1:10010/docs/  
And, be amazed by your API documentation.  
OK, it is the default hello service only... For now!  

We don't spend time yet on the controllers code (../Part1/OpenYeller/MicroServices/UserManager/api/controllers/hello_world.js) as we will explain in details the code and how to edit it in the part2 of the article.

### Launch the Unit Tests

As I said previously, Swagger not only generate automatically the connections for the API, it also creates the unit tests associated... The skeletons of the unit tests only, of course, we will still need to use our brains to write their content! But, with the demo app, we have already 2 tests working.  

To run the existing tests:

```
swagger project test UserManager
```

You should have 2 successes.  

![SwaggerFirstUnitTests](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/SwaggerFirstUnitTests.png "Swagger First Unit Tests")

To check the code of these tests, go in: (../Part1/OpenYeller/MicroServices/test/api/controllers/hello_world.js)
As you can see, it uses the module _should_ to define the test conditions and _supertest_ to do the API requests. We will spend more time on this during the part2 of the article.  
For now, let's see how to package this microservice into a Docker container.

## Creation of the first Docker container
### What is Docker?

I am sure you already heard about Docker. Let's summarize quickly by quoting http://www.docker.com

```
Docker is an open platform for developers and sysadmins to
build, ship, and run distributed applications, whether on
laptops, data center VMs, or the cloud.
```

In this project, we will use Docker to package each microservice into one container.  
Then, we will use a Docker compose file to combine our containers with a Mongodb and Redis containers. (Part2)

### Creation of the Dockerfile

As mentioned, the plan is to create one container by microservice. A Dockerfile contains the definition of a container.  
Hence, the Docker file for UserManager should live in its folder: OpenYeller/Microservices/UserManager

```
|-- OpenYeller
  |-- Microservices
    |-- UserManager
      |-- api    
        |--
      |-- config    
        |--
      |-- test    
        |--
      |-- node_modules  
        |--
      |-- app.js
      |-- package.json  
      |-- Dockerfile

```

In this file, we will tell Docker that we want to create a container based on Node.js 6.3.1 (or higher if there is a more recent version at the time).  
As of today, for production, it is better to use Node.js 4 because this is the LTS version (https://github.com/nodejs/LTS#lts_schedule). But, as this is not a project that we plan to push in production, let's use the latest version available to have the latest features.  
Each container can be seen as a small virtual machine. Hence, we will need to copy in it our project files, to expose the port 10100 and finally run our app.  

The Dockerfile below does this job:

```python
# Based on the latest version
FROM node:6.3.1

# Let's create our app folder and copy our project files
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app
COPY . /usr/src/app

# Open the port of our application
EXPOSE 10010

# Start the application
CMD ["node","app.js"]
```

### Building the first container

If you need to install Docker, have look on:  
https://docs.docker.com/engine/installation/
If asked during the docker tools installation, I recommend for you to keep the Docker tools console and not the Docker tools UI.  

Now that we have the Docker file and Docker installed we can start.  
On Windows or MacOs X, run Docker tools and get the Docker console. On Linux, simply open a console.  
Browse to the UserManager project folder (The folder where lives the Dockerfile) and now, we can build the container.  
To do that, simply run a build with:

```
docker build -t part1usermanager .
```

You should see the below:


What happened here?
Simply, Docker got the container Node 6.2.1 and it ran the commands requested in the Dockerfile on it in order to install our UserManager.  
The option -t specified a tag for our build and the . requested to use the local directory.  
Let's now try to run our container!

```
docker run -P -d part1usermanager
```

- -P option maps the ports exposed to the same port of the container.  
- -d run the container as a daemon

You should see something like:

```python
$ docker run -P -d part1usermanager
2448e96f7c5bef1252f8463c56b6a431c788cc4db7796b130f29a738d970e754
```
We ran our container as a daemon and Docker assigned an ID to our daemon and returned it.  
Please try:

```
docker ps
```
You should see:

![firstDockerPs](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/firstDockerPs.png "First Docker Ps")

On this screenshot, you see that Docker mapped the port 32768 to our 10100 port. We can specify with the Docker run command this port mapping but when we don't do so, Docker does it automatically.  
So, we know we should connect on 32768 but which IP?  
To get it, simply do:  

```
docker-machine ip
```
On my side, I got:

```python
$ docker-machine ip
192.168.99.100
```

So, let's open a browser on (Replace 192.168.99.100 with the IP returned by docker-machine ip command): http://192.168.99.100:32768/hello?name=Scott
You should see "Hello, Scott!" as before.  
Well done! You built the first OpenYeller container and managed to make it run.

For more information on the Docker file and build:  
[Docker builder reference](https://docs.docker.com/engine/reference/builder/)

## Docker Hub and automatic container build
### Push your container on Docker Hub

Go to https://hub.docker.com/  
Create an account or login if you have one already. Then create a repo called usermanager (private or public, up to you).

Open a terminal window

```
docker login
```

Let's build the container using the tag latest to specify this is the latest version.  
The tagging should be used to identify the version of the container for production in order to always be able to deploy the same containers for any version of our WebApp.  
Obviously, for production we should never use the tag latest but a proper versioning tag.  
For me, my login is "mvincke" but you should change it in the following commands with yours.

```
docker build -t mvincke/usermanager:latest .
```

This command built again our container but with another tag. Hence, it was very fast as we didn't change anything on our container.  
Next, we can push our container to Docker Hub.

```
docker push mvincke/usermanager
```

This is one way to push your container in Docker Hub: you build it and you push it.  
Another way is to build it automatically from the sources on Github.

### Using Docker Hub to build the container automatically

_Note_: This section about automatic build is for information only as I don't plan to use it in the next stages.  

**_Create and push on Github project_**  
Let's assume that you already have an account on GitHub. If not Github, maybe BitBucket?  
If not, please create an account on one of them or you will not be able to follow this step as Docker Hub integrates the automatic builds only with these two.  

**_Automatic build_**  
Create a new repository "OpenYeller" in which you will push the files we just created.  
Once the files are in the repository, go to Docker Hub and login, then click on "create" and select "Create Automated Build"  

![DockerHubCreateAutomatedBuild](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/DockerHubCreateAutomatedBuild.png "DockerHub Create Automated Build")

Select Github or Bitbucket depending on which of them you pushed the sources.  
Then, select the folder "OpenYeller" from your repository list.  
Enter a description and select a visibility "private" or "public", up to you. Please note that by default, Docker Hub offers only one private build. You need to pay a monthly fee to get more of them. [More info on Docker pricing](https://www.docker.com/pricing#) (Tab cloud)
Once created, go to "Build settings". By default, Docker Hub will try to find the DockeFile on the root of the project so, we need to specify where it lives.   


![AutoBuildFirstSetup](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/AutoBuildFirstSetup.png "AutoBuild First Setup")

Click on Save Changes and then, "Trigger" to trigger the build.  
Please note that this build will be done automatically in the future after each push in your Github or Bitbucket repository.  
Check on the tab "Build Details" the status of your build. Once it is done, you can switch to the next section: "Deployment on Digital Ocean".


### Deployment on Digital Ocean

Digital Ocean in one example but you can use any of the cloud provider. Most of them allows you to create a VM with Docker installed.  
For Digital Ocean, simply create a droplet choosing the Docker application.  
I recommend using a SSH key but if it is just for test purposes as of now, no need to.

![DockerAppDigitalOcean](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/DockerAppDigitalOcean.png "Docker App DigitalOcean")

Click on create and wait for the droplet to be ready.  
Log into it using SSH (root password is sent by email) and type:

```
docker login
```

Enter your login/password of Docker hub.  
Then, we can deploy the container we automatically build earlier.

```
docker run -p 80:10010 -d mvincke/usermanager
```
You can notice the option -p followd by 80:10100. This means that we map the port 10100 of our container to the port 80 of our VM. We don't let Docker assign automatically the mapping.  
You should see something like:

![DockerFirstDeployment](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/DockerFirstDeployment.png "Docker First Deployment")

Now, get the IP of your VM and launch a browser on it:

http://139.59.166.171/hello?name=Scott

![HelloScottDigitalOcean](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/HelloScottDigitalOcean.png "Hello Scott DigitalOcean")

Hurray!  
Note that this is a bit tedious as we need to go to Digital Ocean, create a droplet, log into it and finally get the container.  
This process will be dramatically improved during the part3 when we will use Docker-machine that will create our droplet and deploy the code automatically.  
This was to deploy the container we built locally and pushed to Docker Hub. To deploy the container built automatically, you should use the repository "openyeller" instead of "usermanager"


## Conclusion of the first part

We have seen how to create our first microservice using Swagger, to package it in a container and to deploy it on a VM.  
Next, we will finalize the code of this first microservice and package it with a Mongodb and Redis containers and deploy this to a virtual machine.


---


## Resources

- **Docker Orchestration and Microservices**
_by Charles Crawford  
Published by Addison-Wesley Professional_   
Available on Safari Books  
- Swagger:
  - http://swagger.io/
  - https://github.com/apigee-127/volos
  - https://github.com/apigee-127/a127-samples


## Contacts
You can contact me on Twitter ![Twitter](https://dl.dropboxusercontent.com/u/52579856/OpenYeller/Img/TwitterBird-40x40.png): [Matthieu Vincke](https://twitter.com/MatthieuVincke)
