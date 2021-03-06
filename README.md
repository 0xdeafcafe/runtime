# Runtime

> Opinionated framework for abstracting JSON-RPC services written in Go, so they can run in multiple execution environments

It is currently designed for AWS Lambda and Amazon API Gateway HTTP APIs, but an HTTP implementation is included for local development.

An example app can be found [here](https://github.com/g-wilson/runtime-helloworld).

## Components

### RPC

The main abstraction this library provides is the RPC Service. It represents the "glue" layer between the execution environment (e.g. Lambda on HTTP API Gateway) and a generic Go application, provided by the developer.

It uses reflection to link ordinary Go functions (from the developer's application) to an external JSON-based interface. Provide a method name `string`, a handler function `interface{}`, and a JSON-Schema for argument validation to get going.

The idea here is to provide standardised RPC behaviour regardless of execution environment. Unlike most frameworks which assume you want HTTP handling, `runtime` is designed to be portable between such environments. The best example of this is being able to run a service on AWS Lambda, and invoke it through an API Gateway, whilst also being able to run the service as part of a Go HTTP server.

Internally, it provides several utilities expected of a modern application such as a pattern for error responses, a contextual logger (logs request details but can also be used by the application), and an abstraction for authentication state.

### Hand

`hand` is an error type which represents an "error by design" - an outcome which is not the happy path but is _handled_ by the system as an expected behaviour.

When returned by an RPC method, `hand` errors are serialised into the response JSON. Therefore, an error should be _handled_ only if it is safe to return to clients. If you have debug data from errors, you should log them.

An additional benefit of this approach is that the RPC Client can coerce a JSON response body and test for conformance of the `hand` type - which means error propagation between RPC services is taken care of.

### Logging

Logging is designed to be useful but extensible. Logrus is used due to its popularity and semantics.

All invocations of RPC methods are logged. And the request logger understands the `hand` error type, so you can have useful output with little up-front effrort.

The Go context within a method is provided with a context-aware logger. This should be used within methods so that when your application writes log messages, you can have contextual data attached as fields automatically - such as the request ID, crucially!

### Authentication

A lightweight `Claims` type is provided and attached to the request context to encapsulate authentication state. It is quite specific to JWTs.

There is no built-in authentication or token validation to the RPC Service itself. It is assumed that you'd run any automatic authentication provided by the execution environment, for example JWT Authorizers in AWS API Gateway. However, there is a JWT validation utility which is designed for use by the development server.

Services can define an "Identity Provider" which can be used to convert the standard claims struct into a more useful application type.

### Development Server

A basic HTTP server is provided which allows you to invoke RPC Methods locally.

## Future scope

- API versioning
- Explain in the readme how service-to-service communication can be implemented
