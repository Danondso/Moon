---
layout: post
title: "Debugging a Spring Boot App with docker-compose"
date: 2019-01-13
excerpt: "Set up a docker-compose for your spring boot app real fast!"
tags: [blog, docker-compose, docker, java, spring boot]
comments: true
---

With the rise of container frameworks. Running applications locally is now easier that ever. 
Something I noticed while working on dockerizing my applications was that it was hard to find an easy setup out there for running my application with remote debugging enabled.

With this article I'll go over what I did to get my app setup with debug using docker-compose. You can view the complete example [here](https://github.com/Danondso/dublin_rest_demo).

First I dockerized my app:

```Docker 
FROM  openjdk:9-jre # Specifying the base image, I'm using Java 9
ADD target/dublin_rest_demo.jar . # Copies the compiled jar file to the root of the container -- DOUBLE CHECK THIS. 
EXPOSE 8080 8000 # Sets up ports the container listens on: 8080 (spring boots default) and 8000 (debugger port)
CMD java -jar dublin_rest_demo.jar # Specifies the command that will start up the application. 
```
Once I had it setup I ran it to make sure everything worked. 
```cmd
mvn clean install && docker build -t rest_demo:latest . && docker run -p 8080:8080 -p 8000:8000 
```

Then I setup my docker-compose yml file: 

```yml
version: '3.7'
services:
  web:
    build: .
    ports:
      - "8080:8080" # Mapping the internal ports to the external ones. 
      - "8000:8000"
```

Then I ran that to make sure everything worked all right. 
```cmd
docker-compose up
```
Then I went about trying to get it setup to run with debug enabled. What I got finally was adding a command to the docker-compose file that started the app with debug enabled.

```yml
version: '3.7'
services:
  web:
    build: .
    ports:
      - "8080:8080"
      - "8000:8000"
    command: java -agentlib:jdwp=transport=dt_socket,server=y,suspend=n,address=*:8000 -jar dublin_rest_demo.jar
```

For a while I did get stuck with the command I used. I didn't realize the suspend arg was set to 'y'. This tripped me up for a while. I didn't realize I was setting my app to be suspended until I attached to the debugger. Once I realized that the application started up without issue. 

Since I'm not always running my app through docker-compose in debug mode. I'll usually create a second file and throw debug in the name somewhere. Then run docker-compose with the -f flag and my file name. 



Hope this helps alleviate someones struggle out there. 