## Client Server Architecture

**Clients** (Browser, Mobile App, postman, server as well can be clients) communicate with servers for response  
* HTTP is standred protocol (Rule book, Guidelines, Standardization for communication) for Requests and Responses  
  * Request Structre (Method,Host, Path, Http Version, Headers, Body)
  * Response Structure (Status code, Headers, Body)
  * Meaning of GET, POST etc
  * How the data is sent (HTTP / HTTPS)

---

## Why not use Core Java ?

**Core Java** can be used to build applications, with help of sockets and server sockets that listen to some port (core java functionality)  
Listening a network request on a port 
```java
ServerSocket server = new ServerSocket(8080);
```

### Problems Which make Development very hard in core Java
* Java Sockets read request in form of streams i.e manual conversion into HTTP Request and Response
* Manual URL / Route Parsing 
* Mapping a method to an endpoint route 
  * Huge Else if Ladder for Mimicing conroller
* This all is done through main thread and other requests have to wait, manual multi threading has to be done.
* Huge Boilerplate code to be written

## Introduction of servets and servlet containers

**Servlet container** we call it as **Server** This also listens Networking request but also performs the Stream to HTTP conversion and boilerplate problem
* Calls Appropriate Servlet for a route (Mapping problem)
* Manages Threads (No manual Multithreading)
* Tomcat is famous servlet container. keeps listening for requests.

* servlet container calls servlet based by parsing the route (eg /api/login) and chooses servlet 
* Servlet is Java Class which handles request one request is sent to servlet (Class basically)
  * GET  → doGet()
  * POST → doPost()
  * PUT  → doPut()
  * DELETE → doDelete()

<img src="images/servletcontainer.png" width="500">

## Problems and Need for Spring Framework

> When Servlet code is scaled to some enterprise level, The code is tightly coupled. 
* Manual creation of objects  
* Controlling the flow of execution 
* Changing a existing code feature would require touching code in large codebase, breakage and expensive bug fixes
All these make code Tightly coupled

## Spring Core 
It is Concept, Ideology (Dependency Injection, Inversion of control, Beans) Assumes a Configuration reduces boilerplate problem further 
solution to Tightly coupled problem.  
The spring MVC, Spring Data, Spring security all are based on spring core 

## Spring Boot 
This is Opinionated system (provides default config), This is build upon the services (Spring MVC, Spring data..)
* Internally uses servlets and modren spring a single servlet exists for req and response handeling (Evolution from many servlets to one)
* from Dispatcher Servlet the request enters Spring MVC.
* some other sevlets like default servlet for static (HTML, CSS) files and custom servlets etc.

<img src="images/spring.png" width="500">


### Port 
Just as IP address is used to identify device, Port number is used to identif application inside device.  
socket is object maintained by os which has port mapping and related information.  

### Dependency
library On which my code depends becomes my Dependency. External library required by my project 

### JAR Packaging - (Java Archive)
Containes Our code as well as setup code to run our application, so that we can have communication between server and application.

## Proxy
A server That sits beteween another server and forwards requests
* Forward proxy or proxy
* Reverse Proxy

``` text
Client -> Proxy -> server  
```
if proxy is working for user its forward and if its working for backend its reverse proxy  


Main Use cases of proxy:
* We configure Proxy for many clients so that we can Hide internal Ip's and this only represents the ip of proxy  
* Controlling what clients can request for
* Caching between requests of clients  

Simple reverse proxy is also the same 
``` text
Client -> Proxy -> server  
```

Main Use cases of reverse proxy:  
* It works for server, Hides the ports of backend Server, Topology i.e how backend is organized.
  * **DNS contains domain to Reverse proxy IP not servers** so that the rules can be enforced and actual ip is hidden.
  * core usecase of proxy is to make attackers/users cannot directly connect to your backend service and bypass the proxy's controls.
* Decides what server to handle a request. and also acts as load balancer if more instances are there.
* HTTPS encrption and decrption and forwards HTTP request.
* Rate Limiting
* Caching of responses that include (Static files like images, CSS, JS).


if Huge Application we can use multiple Reverse proxies and load balancer 
``` text
Clients
   ↓
Load Balancer
   ↓
RP1   RP2   RP3
 ↓     ↓     ↓
Backend servers  
```
Reverse proxy do jobs like caching, rate limiting, load balancing, Routing (configuration if its microservice)
Load is further reduced for Reverse proxies with load balancer and no need for extra activities on above layer so dedicated load balancer is enough and suggested.

## Client server detailed 
<img src="images/clientserver2.png" width="500">

## JAR (Java Archives)
uses similar to zip files and bundles java .class files, Images/Resources, metadata  
like mainfest.mf which tells entry point of jar file  
```java
java -jar file.jar
```

## Artifact 
Final packaged file produced after build phase of an application, that can be used, shared or deployed

## Library JAR vs application JAR
Library Jar is not executable in itslelf and generally does not contain the dependency jars installed, maven have to install from its pom.xml which maven does when we press sinc button. (resolve transitive dependencies)  

Application JAR is executable and contains/bundles all library JARS with it.its complete.
it Has all library jars bundled and independently executable. 


## Maven

Tool which we can manage Java/Spring Projects, Downloading of dependencies, Transitive Dependencies, Providing lifecycle of project, 
which includes the processes in all phases like verify,compile etc.  

Reduces manual effort of maintaining project

## Plugin
is Code that maven uses to perform some functionality 

## What is Build phase in maven Life cycle 
Entire process of Lifecycle is build phase which maven automates and we get target folder as a result which has .class files and .jar file

## Phases 
```text
clean -> independent phase used to clear target folder that contains .class and/or .jar (previous build phase result),
tip -> we generally combine and use clean initially "mvn clean install" like wise
Validate -> Checks the structure and checks weather the pom file is valid or not  
Compile -> compiles the java code   
Test -> unit tests  
Package -> Process of converting your application code into .jar file, bundling it, Distributable format 
verify -> integration Tests 
install -> save in local repository the M2 Folder
Site -> some documentation related
Deploy -> to central or remote repository
```

## pom.xml (Super, Parent, effective)
Maven uses pom.xml file as this file contains all dependencies, plugins, meta data of project  
Super -> parent of all pom files contains default configurations  
Parent -> The immediate parent pom.xml from where you are inheriting   
Effective -> the effective configurations of pom.xml code from all the chain from super pom to your pom.xml  


## How Maven searches for a Jar (Local Repository, Remote, Maven Central Repository)
First checks in local computer inside /M2 folder basically the cached jar files if not found here it eaither searches in Remote if specified if the code belongs to some organization or maven central where all repositories are published.

## Maven Archtypes 
its maven templates of projects for a quick start. 


## Tightly Coupled Design
* Insted of maintaining abstraction you specify details in our code i.e Not coding for interfaces
* When one part of code is changed the other part is also forced to change (Tight coupling) 
* Harder to maintain, code breakage risk is high.
* As some services are costly to test we create simple test services which also require to be changed in Tightly coupled system


* Better way of Dependency injection is constructor injection (java) were just main func has the control and Tight coupling is reduced.
* so Tight coupling is reduced not eliminated, so comes concept of IOC and Beans

## Inversion of control 
* We (spring core) is taking the control from main, eliminating tight coupling and it manages objects and injects where ever necessary
* so control is in IOC container now which passes dependencies to services.
* before the control stays inside main, now IOC container does all by maintaining beans (i.e creation of dependencies, lifecycle of dependency object, configuring dependencies of dependencies)
* This is called IOC, Using IOC + DI + coding for interfaces we can eliminate tight coupling 

## Loose coupling 
* Maintaing Abstraction i.e coding for inrterfaces so that code is following solid principles.
* Easier Unit testing, No changes in src code required for change of features


