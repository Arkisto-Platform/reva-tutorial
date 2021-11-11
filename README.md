# Working with Reva via nodejs API's

- [Working with Reva via nodejs API's](#working-with-reva-via-nodejs-apis)
  - [Reva development environment](#reva-development-environment)
  - [API Documentation](#api-documentation)
    - [Versions](#versions)
  - [Code examples](#code-examples)
    - [Requests to reva](#requests-to-reva)
  - [Working with the CS3APi's](#working-with-the-cs3apis)
  - [Inspecting the available methods](#inspecting-the-available-methods)
  - [Performing a request with a token](#performing-a-request-with-a-token)

This is a tutorial for working with reva via the CS3 node js API's.

## Reva development environment

If you need a reva development environment to work with see:
[https://github.com/Arkisto-Platform/describo-reva](https://github.com/Arkisto-Platform/describo-reva)

## API Documentation

The CS3 API documentation is at
[https://cs3org.github.io/cs3apis/](https://cs3org.github.io/cs3apis/)

### Versions

From the CS3API repo page ([https://github.com/cs3org/cs3apis](https://github.com/cs3org/cs3apis))
there are a number of language bindings referenced. Note that the version numvers in those
respective repo's are `not` consistent. To see which version is built look for the commit message
like `Synced to https://github.com/cs3org/cs3apis/tree/{commit id}`

## Code examples

Code examples for authenticate, whoAmI, uploadFile and downloadFile are in
[src/index.js](src/index.js).

### Requests to reva

There are two types of requests when dealing with reva: GRPC and HTTP. GRPC is used as the
communication channel (authenticate, create shares, setup uploads / downloads etc) whilst http is
used to performing certain operations (actually do the download / upload etc).

Look at the methods `uploadFile` and `downloadFile` to see this in action. First there are the GRPC
operations which define what we want to do and these are then following by standard HTTP requests to
perform the actual operation.

## Working with the CS3APi's

The basic pattern is as follows:

-   get a handle to the gateway client
-   set up the request type
-   pass the request to the client to action

Following is an authentication example (from [src/index.js](src/index.js)):

```
async function authenticate({ username, password, gateway }) {
    // get the client
    const client = new GatewayAPIClient(gateway, credentials.createInsecure(), {});

    // get the authenticate method from the client - promisified
    //  promisifyAll method in src/index
    const { authenticate } = promisifyAll(client);

    // set up the authentication request
    let req = new AuthenticateRequest();

    // set the required properties
    req.setType("basic");
    req.setClientId(username);
    req.setClientSecret(password);

    // pass the request object to the authenticat method
    let response = await authenticate(req);

    // extract the require info from the response
    let token = response.getToken();
    let user = response.getUser().toObject();
    return { token, user };
}
```

## Inspecting the available methods

The API docs don't provide the method names so to see what is available use `Object.getProtoypeOf`.
Following is an example:

```
async function authenticate({ username, password, gateway }) {
    // get the client
    const client = new GatewayAPIClient(gateway, credentials.createInsecure(), {});

    // inspect the available methods in the client
    console.log(Object.getPrototypeOf(client))

    // get the authenticate method
    const { authenticate } = promisifyAll(client);

    // set up the authentication request
    let req = new AuthenticateRequest();

    // inspect the methods available on the authentication request
    console.log(Object.getPrototypeOf(req))
}
```

## Performing a request with a token

Most methods will require a credential to be passed (e.g. the token you got back from the
authentication step) in a specific way. Following is part of the `listFolder` example in
[src/index.js](src/index.js) showing how this is done.

```

export async function listFolder({ gateway, token, folder = "/" }) {
    // set up a metadata object with the token as the value of 'x-access-token'
    const meta = new Metadata();
    meta.set("x-access-token", token);

    // get the client and extract the methods you need
    const client = new GatewayAPIClient(gateway, credentials.createInsecure(), {});
    const { listContainer, stat } = promisifyAll(client);

    // in this example we're doing a list folder request so
    // * set up a reference with the path we want to list
    let ref = new Reference().setPath(folder);

    // set up the request
    let req = new ListContainerRequest();

    // set up the reference in the request
    req.setRef(ref);

    // pass the request and the `meta` object with the token to the listContainer method
    let containers = await listContainer(req, meta);

    ...
```
