# Public API

Public API is REST API that is consumed by the public facing application. You can find the Open Api specification over
here <https://public-api.movement-pass.com/v1/index.html>. The `identity` endpoint is responsible for user
registration/login and the `passes` endpoint which can be only accessed by authenticated users for applying and viewing
the pass/passes. As mentioned previously we have both NodeJS and .NET implementation but the input/output of the API are
identical.

## NodeJS

[NodeJS public api repository](https://github.com/movement-pass/node-public-api)

The NodeJS implementation is an Express.js application written in typescript. Most of the common construct of express is
used but the implementation is not typical that you usually see in regular express applications. The patterns and
principles that are used here are originated from the .NET community, I find nothing wrong even do recommend bringing
the principles and practises from other communities which helps you to build better software. Let's see the high level
diagram of this express application:

![Express App](media/nodejs-public-api.png)

The app contains two routers which itself tied with middlewares, when a request arrives the middleware ensures whether
it can continue processing the request(request validation, authorization) or to early return, if the middleware allows,
the router delegates the request processing to its associate controller. When the controller is called, based on the
method it instantiates an application request class from the incoming http request and sends it to mediator for
handling. Based on the request the mediator knows which handler is responsible for handling the request and delegates
the request to that handler, when the mediator receives the result from the handler, it returns the result back to the
calling controller which in turn constructs the proper http response and returns back to the client. You may wonder in
typical express application, application codes are usually written in routers why do we need so many layers, well the
primary reason to separate out the core application logic from its underlying infrastructure, as you can see in express
all it needs to know the mediator and how to construct the application request object from the http request and when
result is returned from mediator how to translate it back to proper response, rest of the part it completely opaque to
it. Since the idea of the mediator is, one object in and one object out, testing the handlers in isolation becomes
completely frictionless. Another benefit of this kind of separation is that if we decide to move to other web
frameworks(though very unlikely) like koa or fastify we can reuse our application core logic without any modification.
Another important thing you would encounter in the codebase is the use of Dependency Injection(again it is heavily used
in .NET and Java world) or DI in short, unless you have experience in Angular or NestJS you may find it a new thing to
explore. If you are not familiar with dependency injection, lets take the above case in example, the router requires
controller to function, the controller requires mediator, the mediator needs the handlers and each of the handler itself
has its own set of dependencies and this chain can go on and on and while in development when adding new or refactoring
existing features we often have to add/remove these dependencies. We can of course create these whole objects graph
manually, but it becomes tedious and painful very quickly. So rather than doing it manually we register our dependencies
in DI container and when required we ask the container to create for us. There are many DI container available in
typescript but in this solution we are using [tsyringe](https://github.com/microsoft/tsyringe) by Microsoft.
