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

![Servlet container and servlets](images/servletcontainer.png)


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

![Spring Boot Simple Architecture](images/spring.png)



