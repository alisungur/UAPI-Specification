# REST Guidelines

The following are guidelines for REST services. Your API doesn't need to follow all of these guidelines to be functional, but failing to follow too many guidelines will result in a REST API that is confusing and hard to consume.

**Page Contents**

- [Terminology](#terminology)
- [Examples Background](#examples-background)
- [API Request Inputs](#api-request-inputs)
- [API Response Outputs](#api-response-outputs)
- [HTTP Methods](#http-methods)
    - [GET](#get)
    - [POST](#post)
    - [PUT](#put)
    - [DELETE](#delete)
- [Versioning](#versioning)

## Terminology

- *API* - An interface that accepts inputs and produces associated outputs. For example, adding `2 + 3` results in `5`. The inputs are `2 + 3` and the output is `5`. REST API's provided a standardized interface for interacting with web resources.

- *Collection* - Multiple resources together. For example a `Person` can be a resource and the collection for this resource would be a list of `Person` resources.

- *Endpoint* - A location within the REST API (specified by domain, path, and [method](#http-methods)) that is used to retrieve or affect resources.

- *Idempotent* - Calling an endpoint more than once with the same input parameters will have no additional effect.  

- *Resource* - A set of related data. For example, a `Person` resource could include related data such as `first name`, `last name` and `birthdate`.

## Examples Background

For the examples contained within this document we'll be using:

1) A fictitious database table that stores email addresses

    | id  | email |
    | --- | ----- |
    | 1 | bob@fake.net |
    | 2 | amanda@fake.net |
    | 3 | chris@fake.net |

2) A fictitious REST API with the following endpoints:

    - `GET /`
    - `POST /`
    - `GET /{id}`
    - `PUT /{id}`
    - `DELETE /{id}`

3) A fictitious domain: `http://email-registry.com`

## API Request Inputs

There are five ways to provide input to an API:

1. The method, discussed in detail below in the [HTTP Methods section](#http-methods).

2. The path.

    - The path is part of the endpoint identifier.
    
    - A path may be static or fixed in that it is always the same.
    
    - A path may be dynamic and allow for inputs, although path input be identifiers that uniquely identify a single resource.
    
3. The query string.

    - Can be used to identify a single resource or a search criteria for multiple resources.
    
    - Can define filtering and pagination inputs.
    
    - Order of the query string parameters should not matter.

3. The headers.

    - Headers contain information about the request and information about how the response should be formulated.
    
    - The `Authorization` header is used to pass credentials or OAuth tokens to the endpoint.
    
4. The body.

    This is the payload for the request. The content type is generally `application/json` but can be of any type.

## API Response Outputs

After making an API request, the response returns three pieces of information:

1. The response status code.

    This code should always be the first thing that an application looks at after receiving and API response and it will indicate the success or failure to fulfill the API request. For a list of common response codes see https://en.wikipedia.org/wiki/List_of_HTTP_status_codes.

2. The headers. These headers contain the metadata about the response, including the content type being returned.

3. The body.

    - Response bodies should return the requested resource or resources.
    
    - The response may return some additional computed attributes that are associated with the resource that may not have been included in the request that stored the resource. For example, an auto incrementing unique identifier, or a "last updated date" property.
    
    - Avoid presentation layer content. For example, don't send back and HTML page when returning a JSON document would suffice. This lets many different presentation layers use the same set of data.

## HTTP Methods

The HTTP methods specify the action to take on a resource. 

The most common methods include `GET`, `POST`, `PUT` and `DELETE`. DO NOT confuse CRUD (create, read, update, delete) with the `GET`, `POST`, `PUT` and `DELETE` methods. There are important differences.

**Methods Summary**

| Method | Description | Idempotent | Body OK |
| ------ | ----------- | ---------- | ------------ |
| GET | Request data about a resource or resources. | Yes | No |
| POST | Create a new resource. | No | Yes |
| PUT | Put a resource to the specified state. | Yes | Yes |
| DELETE | Remove a resource. | Yes | Yes |

### GET

- Used to retrieve a single resource or a collection of resources.

- Calling a REST endpoint with `GET` has no side-effects; you can't use `GET` to create, update, or delete a resource.

- The `GET` method can use the path, query parameters, and headers as input but should avoid using the request body.

- For getting a single resource it is common to put the resource identifier in the path.

**Common Response Codes**

- `200` - Resource or resources successfully retrieved.

    *Get a collection with three items*

    ```
    GET /
    ["bob@fake.net", "amanda@fake.net", "chris@fake.net"]
    ```
  
    *Get a collection with zero items*

    ```
    GET /
    []
    ```
  
    *Get a resource*
    
    Notice how the resource identifier is included in the endpoint's path.
    
    ```
    GET /1
    bob@fake.net
    ```

- `400` - There is something wrong with the request. The response body would ideally include what was wrong with the request.

- `401` - The request requires authentication and the request did not have authentication information.

- `403` - The request provided authentication information but the authenticated user is not authorized to get the results.

- `404` - Generally this indicates that the requested endpoint was not found. If the endpoint itself addresses a resource through path parameters then a `404` would indicate that the resource does not exist. If the endpoint returns a collection then a `404` should not be used if the collection exists but has no items, instead use a `200` and send back an empty collection.

    ```
    GET /foo
    ``` 

### POST

- Each `POST` request should generate a new resource.

- Calling the same `POST` API endpoint with the same input parameters and body should either create a new resource or return an error.

- The `POST` is not used to update existing resources.

**Example**

If we `POST` twice to the endpoint `POST /` with the body `sam@fake.net`, then our database would look like this below. Notice that `sam@fake.net` exists twice. This is how a `POST` should work.

| id  | email |
| --- | ----- |
| 1 | bob@fake.net |
| 2 | amanda@fake.net |
| 3 | chris@fake.net |
| 4 | sam@fake.net |
| 5 | sam@fake.net |

Common response codes for a `POST` request include:

- `201` - The resource was created. If a response body is provided then it should represent the resource created and look very similar (ideally identical) to the resource returned by `GET /{id}`.

- `400` - There is something wrong with the request. The response body would ideally include what was wrong with the request.

- `401` - The request requires authentication and the request did not have authentication information.

- `403` - The request provided authentication information but the authenticated user is not authorized to get the results.

- `404` - The requested endpoint was not found.

### PUT

- A `PUT` method signifies that the state of a resource should be set to the provided input.

- This `PUT` method is idempotent. Calling the same endpoint multiple times with the same input will not change the resource to be anything different than the first request. It also will have no side-effects for the system. 

- `PUT` should not be used for partial updates of a resource.

- `PUT` endpoints will commonly include the identifier in the endpoint's path.

**Example**

Calling the endpoint `PUT /1` multiple times with the body `sam@fake.net` will always result in the email address being set to `sam@fake.net` for ID `1`.

| id  | email |
| --- | ----- |
| 1 | sam@fake.net |
| 2 | amanda@fake.net |
| 3 | chris@fake.net |

Common response codes for a `PUT` request include:

- `200` - The resource already existed and was set to the specified value. If a response body is provided then it should represent the resource created and look very similar to the resource returned by `GET /{id}`.

- `201` - The resource was created. If a response body is provided then it should represent the resource created and look very similar to the resource returned by `GET /{id}`.

- `400` - There is something wrong with the request.

- `401` - The request requires authentication and the request did not have authentication information.

- `403` - The request provided authentication information but the authenticated user is not authorized to get the results.

- `404` - The requested endpoint was not found.

### DELETE

- This `DELETE` method is idempotent and calling the same endpoint multiple times will result in the resource being deleted without side-effects for the system.

- `DELETE` endpoints will commonly include the identifier in the endpoint's path.

- `DELETE` endpoints should not return the content that would be returned by a `GET /{id}` request because if the `DELETE` is called more than once then the content for the resource will not exist on the second request.

Common response codes for a `DELETE` request include:

- `204` - The resource was deleted and no response body is being sent.

- `400` - There is something wrong with the request.

- `401` - The request requires authentication and the request did not have authentication information.

- `403` - The request provided authentication information but the authenticated user is not authorized to get the results.

- `404` - The requested endpoint was not found. Don't use this to indicate that the resource was not found, use `204` instead to indicate that it has been successfully deleted.

# Versioning

Once an API is published it must not break the contract by changing endpoints or altering existing fields. It is OK to add additional fields and endpoints.

It is important to version your API so that if the need arises to break your API contract you can create a new version that users can begin to point their applications to. You should not disable previous API versions until no one is using them, although it is important to mark them as deprecated.

Versioning your API is done by prefixing the version information to the path, following the domain.

Examples:

- `http://api.email-registry.com/v1/`

- `http://api.email-registry.com/v2/`
