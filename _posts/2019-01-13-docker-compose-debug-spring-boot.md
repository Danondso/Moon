---
layout: post
title: "Debugging a Spring Boot App with docker-compose"
date: 2019-01-13
excerpt: "Set up a docker-compose for your spring boot app real fast!"
tags: [blog, docker-compose, docker, java, spring boot]
comments: true
---

With the trending of containerizing applications, setting up your local environment is now easier than ever. Setup times can be reduced from possibly hours to mere minutes. Saving devs time and frustration.

While setting up one of my apps to run with Docker. I realized there weren't any guides out there that suited my fancy for running apps with debug enabled through docker.
With this article I'll go over what I did to get my app setup with debug using docker-compose. You can view the complete example [here](https://github.com/Danondso/dublin_rest_demo).

First I created the Dockerfile:

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
      - "8080:8080" # Mapping the internal container ports to the external ones.
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

Since I'm not always running my application in debug. I'll usually create a second file and call it docker-compose-debug.yml. Then run:

```cmd
docker-compose -f docker-compose-debug.yml up
```

And that was it, I was on my way to debugging my app!

To confirm it was actually it was setup correctly. I setup a remote config in IntelliJ:

<figure>
  <img src="/assets/img/post-images/2019-01-13/run-configuration-intellij.png">
</figure>

Then I ran my docker-compose-debug file:

<figure>
  <img src="/assets/img/post-images/2019-01-13/run-docker-compose-debug.png">
</figure>

Finally, I set a breakpoint and hit it:

<figure>
  <img src="/assets/img/post-images/2019-01-13/debug-hit.png">
</figure>

And there it is, we're running our app via docker-compose with debug enabled.

Overall it was a pretty easy setup. When I first tried to setup debug I did get stuck with the command I used originally. I didn't realize the suspend arg was set to 'y'. This tripped me up for a while. I didn't realize I was setting my app to be suspended until I attached to the debugger. Once I realized that the application started up without issue.