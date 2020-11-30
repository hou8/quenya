# Quenya extension for OpenAPI

There are several rules for Quenya specification:

1. all routes MUST have a unique `operationId`. We will use this as the identifier for generated routes.
2. severs in OpenAPI object shall contain at least one "http://localhost:{port}". We will use this to generate which API path we should forward in `gen/YourApp.Gen.Router.ex`.
3. all routes MUST have at least one and only one `2xx` response. We will use this to generate fake responses.

## Server Object

We propose an `x-env` field in the [Server Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md#serverObject):

| Field Name |   Type   | Description                                        |
| ---------- | :------: | -------------------------------------------------- |
| x-env      | `string` | environment for ['dev', 'test', 'staging', 'prod'] |

## Operation Object

We proprose an `x-grpc` field in the [Operation Object](https://github.com/OAI/OpenAPI-Specification/blob/master/versions/3.1.0.md#operationObject):

| Field Name |     Type      | Description                          |
| ---------- | :-----------: | ------------------------------------ |
| x-grpc     | `XGrpcObject` | grpc service to call to get response |

### X GRPC Object

X GRPC Object defines how to proxy REST API to GRPC service. If this object is defined in the operation, a gRPC handler will be generated by Quenya. The handler will map the request context (all parameters processed by `RequestValidator`) to the gRPC requestObject and then call the gRPC service. Once the response is returned, it will be mapped to the corresponding response body of the REST API.

Normally the authentication / authorization will be processed by the REST API layer, so we don't need to handle that in the gRPC layer, but if we do need to process it, we can do a headerMapping for the `Authorization` header.

| Field Name      |          Type           | Description                                                 |
| --------------- | :---------------------: | ----------------------------------------------------------- |
| endpoint        |        `string`         | (required) url for the service                              |
| requestObject   |        `string`         | (required) request object name for the grpc endpint         |
| responseObject  |        `string`         | (required) response object name for the grpc endpint        |
| requestMapping  | `map[string -> string]` | (required) HTTP request params map to gRPC request object   |
| responseMapping | `map[string -> string]` | (required) gRPC response object map to HTTP response object |
| headerMapping   | `map[string -> string]` | (optional) REST API header map to gRPC header               |

Example:

```yaml
endpoint: "https://domain.com/api/GetTodo"
requestObject: GetTodoRequest
responseObject: GetTodoResponse
requestMapping:
    filter: filterBy
responseMapping:
    id: id
    title: title
    body: body

```