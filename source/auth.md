---
title: Authentication
order: 20
---

Some applications don't deal with sensitive data and have no need to authenticate users, but most applications have some sort of users, accounts and permissions systems. If different users have different permissions in your application, then you need a way to tell the server which user is associated with each request. Over HTTP, the most common way is to send along an authorization header.

Apollo Client has a pluggable network interface that lets you modify requests before they are sent to the server.
That makes it easy to add a network interface middleware that adds the `authorization` header to every HTTP request:

```js
import ApolloClient, { createNetworkInterface } from 'apollo-client';

const networkInterface = createNetworkInterface('/graphql');

networkInterface.use([{
  applyMiddleware(req, next) {
    if (!req.options.headers) {
      req.options.headers = {};  // Create the header object if needed.
    }
    // get the authentication token from local storage if it exists
    req.options.headers.authorization = localStorage.getItem('token') ? localStorage.getItem('token') : null;
    next();
  }
}]);

const client = new ApolloClient({
  networkInterface,
});
```

The example above shows how to send an authorization header along with every request made. The server can use that header to authenticate the user and attach it to the GraphQL execution context, so resolvers can modify their behavior based on a user's role and permissions.

In order for the example above to work properly, another part of your application has to obtain an authentication token (eg. JWT) when the user logs in, and store it in local storage. It is also important to clear the token from local storage when the user logs out and clear ApolloClient's if it contains sensitive information.
