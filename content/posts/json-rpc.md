+++
date = '2024-12-30T13:06:40-05:00'
draft = true
title = 'How JSON-RPC Eases REST API Frustrations'
tags = ["technology", "architecture"]
+++

In this post, I'd like to share my thoughts on the frustrations of REST APIs and how JSON-RPC can alleviate some of these challenges. While some may argue that comparing JSON-RPC with REST is like comparing apples to oranges, I believe that if both are fruit, they can still be meaningfully compared. Before diving into the details, I’ll assume you're already familiar with REST. However, if you're unfamiliar with JSON-RPC, you can find the specification [here](https://www.jsonrpc.org/specification). It's concise and should take about 10 minutes to read through.
___

## REST APIs Are Burdened by Dispersed Parameters
REST API parameters are located in several different parts of the HTTP request. In addition to body parameters, there are path parameters, and in the case of `GET` requests, query parameters. While most developers likely do not consider this an inconvenience—probably due to the unquestioning acceptance of the status quo—I would argue that once you experience the simplicity of a JSON-RPC API, you’ll become more aware of this inconvenience yourself.

### Pesky Path Parameters
In REST APIs, path parameters are often used to identify a specific resource. For example, the path `/api/v1/books/123` would refer to the `book` object with an ID of `123`. On the server side, this requires extracting the parameter from the URL.

In most cases, you’re likely using a framework in your programming language of choice that can automatically extract the parameters and cast them to the appropriate type, so this usually isn’t an issue. However, if you don’t have access to a framework—perhaps due to restrictions like closed-source requirements or operating in a memory-constrained environment—extracting that parameter manually can be tedious, at least when compared to the simplicity of a parameter in a JSON object. In the Go programming language, the ability to easily [extract path parameters](https://go.dev/blog/routing-enhancements) using just the standard library was only recently added.

### Quirky Query Parameters
If you want to retrieve a resource from a REST API endpoint, you must use the HTTP `GET` method. Although _technically_ possible, it is [not recommended](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET) to include content in the body of an HTTP `GET` request; therefore, any parameters must be sent in the query string portion of the URL.

The problem with query parameters is that they must be URL encoded which has many limitations when compared to JSON, including:
- Lack of support for data types (everything is a string).
- Lack of support for complex data structures (everything is flat).
- The headache of URL Encoding.

I find it frustrating that for `POST`, `PUT`, and `PATCH` requests we can use JSON, but as soon as we use a `GET` request we are forced to rely on query string parameters. It feels like an unnecessary limitation..

Below are just a few examples of the frustrations with query string parameters.

#### Ambiguous Boolean Values
Since query parameters aren't typed, it is ambiguous as to what string value would represent a boolean. Therefore it is common to manually check for the following string values for each boolean:
  - `true`: "true", "t", "yes", "y", "on", "1"
  - `false`: "false", "f", "no", "n", "off", "0"

A recent [post](https://forum.djangoproject.com/t/add-getbool-to-request-query-params-querydict/30985) in the Django community asked for the addition of a `getbool()` function to simplify this common yet annoying scenario. (They did not add it.) Since JSON supports a boolean data type this is not a problem.

#### Improperly Encoded Characters
As a backend developer I have frequently encountered the issue where the client did not properly URL encode the query string leading to unexpected results on the server side. One common example is with the plus `+` sign. To send a plus `+` as a string in a query parameter, it must be encoded as `%2B`. If the client fails to encode the plus sign correctly, it will be interpreted as a space ` ` character.

Below is an example using Python:
```
from urllib.parse import parse_qs

>>> parse_qs("key=+")  # not encoded correctly
{'key': [' ']}

>>> parse_qs("key=%2B")  # encoded correctly
{'key': ['+']}
```

### JSON-RPC: A Cleaner, More Consistent Approach to Parameters
Most JSON-RPC APIs have a single endpoint, which does not contain any path parameters, and because they typically only support the `POST` method, you will never have to parse query parameters from a `GET` request. All parameters are located in one place: the body of the HTTP request, in a JSON object identified by the `params` key. The value can either be an array of positional arguments or an object containing keyword arguments. This approach greatly simplifies the development process, as you no longer need to manage or extract parameters from multiple locations.

Before I conclude this section on parameters, I want to mention that there is currently an [IETF proposal](https://datatracker.ietf.org/doc/draft-ietf-httpbis-safe-method-w-body/) for an HTTP `QUERY` method. This method would be a safe and idempotent request, similar to `GET`, but would also allow a request body. If this were to come to fruition, I think it would be a great addition to REST API architecture. However, it’s hard to say whether it will ever be adopted.
___

## When CRUD Hits the Fan
You will often encounter the acronym **CRUD** when dealing with REST APIs. It stands for **C**reate, **R**etrieve, **U**pdate, and **D**elete—the typical actions performed on a resource. These actions are represented by the following HTTP methods, respectively: `POST`, `GET`, `PUT`/`PATCH`, and `DELETE`. However, when you need to perform an action that _isn't_ part of the typical CRUD operations—such as _calculate_, _send_, _verify_, _transfer_, etc.—you’ll need to get creative with your implementation, as there isn't a formal standard for handling these types of situations.

The most common workaround I see is adding the verb to the path and using the HTTP `POST` method. For example, if you were to create an endpoint that sends a verification code, it might look like this: `POST /api/v1/verification/send`. While this is a reasonable workaround, it never sat well with me. I would often spend a lot of mental energy trying to find the most _RESTful_ way to implement this edge case, not realizing that the underlying issue stemmed from the architectural limitations of REST.

### Flexible Method Naming in JSON-RPC
This issue does not exist when designing a JSON-RPC API. Every request contains a `method` attribute that specifies the function to be executed. You are free to create your own naming convention for the method, allowing for greater flexibility and clarity in how you define API operations. Following the same example as above, a method called `send_verification_code` would be appropriate. This is particularly useful when your API needs to perform non-CRUD operations, as the method names can directly describe the action being taken, rather than relying on convoluted paths or HTTP verbs.
___

## Error Responses and Status Codes

### Error Responses
A key component of API design is the structure of error responses. With REST APIs, there is no formal structure defined for error responses. While this isn’t inherently bad, it places the burden on API producers to design their own error response structure. For API consumers, there is no guarantee of 
consistency across different REST API error response structures.

With JSON-RPC, an [Error Object](https://www.jsonrpc.org/specification#error_object) is defined in the specification. It contains two required fields: a `code`, which is always an integer, and a `message`, which is always a string. Additionally, there is an optional `data` attribute, which can be either a primitive or a structured value. The `code` and `message` attributes give every JSON-RPC API a predictable structure, while the `data` attribute provides flexibility for API producers to add custom attributes.

### Status Codes
It is impossible to discuss error responses without also mentioning HTTP Status Codes. With REST APIs, application-level error responses are typically returned with a [4XX status code](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status#client_error_responses). Although there is a formal definition for each status code, they are mostly concerned with the specifics of the HTTP protocol. The REST API producer is left with the mind-numbing task of coupling their application errors to the most appropriate HTTP status code.


> I feel like I need to be more clear on what the actual issue is here.


Most REST APIs will often return a `400 Bad Request` status code for all application errors. However, the original intention of this status code was to indicate a malformed request syntax—for example, an invalid JSON body that cannot be parsed. I think the most appropriate HTTP status code for an application level error is the `422 Unprocessable Entity` status code, which indicates that the server understood the _syntax_ but not the _semantics_ of the request. I find it very satisfying when I come across a REST API that uses the `422 Unprocessable Entity` as a catch all for application level errors and correctly uses the `400 Bad Request` status code for its original intention.

Regardless, I'd argue that the core issue stems from the fact that REST is tightly coupled to the HTTP protocol. It wasn’t until I started writing JSON-RPC APIs that I was finally able to, in my mind, decouple HTTP errors from application errors. Although it might seem unnatural at first, with JSON-RPC, all application errors are returned with a `200 OK` status code. What identifies an application error, is the presence of the `error` object in the payload. If is not present, and therfore contains a `result` attribute, then the request was successful.

The way I make sense of this in my head is that the server returns a 4XX status code **only** when there is an _HTTP_ related issue. If there isn't, then a `200 OK` status code is returned. Whether or not there is an application level error is determined by the presence of the `error` object.

Below is an outline on how I typically handle an incoming HTTP request when writing a JSON-RPC API.
- If the request fails authentication, return a `401 Unauthorized`.
- If the client is denied permission, return a `403 Forbidden`.
- If the URL path is incorrect, return a `404 Not Found`.
- If the HTTP method is not `POST`, return a `405 Method Not Allowed`.
- If the content type is not `application/json`, return a `415 Unsupported Media Type`
- If there is an error while parsing the JSON body, return a `400 Bad Request`.
- If the JSON payload is not a valid JSON-RPC request, return a `422 Unprocessable Entity`

If all of these conditions are met then the _HTTP_ portion of the request is successful, however this is separate from whether or not the executed function was successful or not.
