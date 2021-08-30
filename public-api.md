# Public API

Public API is REST API that is consumed by the public facing application. You can find the Open Api specification over
here <https://public-api.movement-pass.com/v1/index.html>. The `identity` endpoint is responsible for user
registration/login and the `passes` endpoint which can be only accessed by authenticated users for applying and viewing
the pass/passes. As mentioned previously we have both Node.js and .NET implementation but the input/output of the API are
identical. The public api is hosted in lambda and exposed by API Gateway.

## Node.js

[Node.js public api repository](https://github.com/movement-pass/node-public-api)

The Node.js implementation is an Express.js application written in typescript. Most of the common construct of express is
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
result is returned from mediator how to translate it back to proper http response, rest of the application parts is
completely opaque to it. Since the idea of the mediator is, one object in and one object out, testing the handlers in
isolation becomes completely frictionless. Another benefit of this kind of separation is that if we decide to move to
other web frameworks(though very unlikely) like koa or fastify we can reuse our application core logic without any
modification. Another important thing you would encounter in the codebase is the use of Dependency Injection(again it is
heavily used in .NET and Java world) or DI in short, unless you have experience in Angular or NestJS you may find it a
new thing to explore. If you are not familiar with dependency injection, lets take the above case in example, the router
requires controller to function, the controller requires mediator, the mediator needs the handlers and each of the
handler itself has its own set of dependencies and this chain can go on and on and while in development when adding new
or refactoring existing features we often have to add/remove these dependencies. We can of course create these whole
objects graph manually, but it becomes tedious and painful very quickly. So rather than doing it manually we register
our dependencies in DI container and when required we ask the container to create for us. There are many DI container
available in typescript but in this solution we are using [tsyringe](https://github.com/microsoft/tsyringe) by
Microsoft. To register dependencies that is coded by you all you have to do is decorate it with `@injectable` decorator,
for external dependencies string based token is used here. Now talking about the decorators, we have to write a
decorator `@handles`
of our own which adds metadata to handler that the mediator uses to find the correct handler to delegate the request
processing. Here is declaration of a typical request, result and handler:

```typescript
class TheRequest extends Request {
    /* */
}

interface TheResult {
    /* */
}

@injectable()
@handles(TheRequest)
class TheHandler extends Handler<TheRequest, TheResult> {
    constructor(
        @inject('DynamoDB') private readonly _dynamodb: DynamoDBDocumentClient,
        private readonly _config: Config
    ) {
        super();
    }

    async handle(request: TheRequest): Promise<TheResult> {
        /* */
    }
}
```

Controller:

```typescript
class TheController {
    constructor(private readonly _mediator: Mediator) {
    }

    async theAction(req: Request, res: Response): Promise<void> {
        const request = new TheRequest(req.body);

        const result = await this._mediator.send<TheResult, TheRequst>(request);

        res.send(result);
    }
}
```

Router:

```typescript
function theRouter(controller: TheController): Router {
    const router = express.Router();

    router.post(
        '/',
        validate(requestSchema),
        async (req: Request, res: Response) => controller.theAction(req, res)
    );

    return router;
}
```

Till now, we have only discussed the application core but nothing on AWS side. The first thing is, we cannot run express
application directly in lambda, in order to run it in lambda we are
using [@vendia/serverless-express](https://github.com/vendia/serverless-express) which acts as an adapter between lambda
and express environment, for this reason in the codebase you will find two entrypoint `./src/local.ts` for running
locally and `./src/lambda.ts` to run in lambda (there are other adapters are also available for other web frameworks to
run in lambda). Next, we are using the new AWS SDK for JavaScript v3 which means instead of loading gigantic package (
v2) we are only loading the aws libraries that we are using in our code that also reduces lambda package size
drastically. The last thing I want to mention when packing the code we are using another npm
package [@vercel/ncc](https://github.com/vercel/ncc) that compiles the node.js code into a single file which enhance the
runtime performance, this is not a lambda specific thing you can use the same technique in your other server
applications.

## .NET

[.NET public api repository](https://github.com/movement-pass/dotnet-public-api)

There is nothing significant that worth to mention here, because most the things that we discussed in Node.js
implementation are already available in .NET, for the mediator we are using
the [MediatR](https://github.com/jbogard/MediatR) nuget package and like Node.js the .NET implementation also has two
entrypoint, one for local and other when running in lambda. One additional thing in the .NET implementation is the Open
API specification which is missing in the Node.js implementation.
