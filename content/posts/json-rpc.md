+++
date = '2025-01-06T13:08:41-00:00'
draft = false
title = 'How JSON-RPC Eases REST API Frustrations'
categories = ["technology"]
tags = ["JSON RPC", "REST", "API"]
+++

In this post, I'd like to share my thoughts on the frustrations of REST APIs and how JSON-RPC can alleviate some of these challenges. While some may argue that comparing JSON-RPC with REST is like comparing apples to oranges, I believe that if both are fruit, they can still be meaningfully compared. Before diving into the details, I’ll assume you're already familiar with REST. However, if you're unfamiliar with JSON-RPC, you can find the specification [here](https://www.jsonrpc.org/specification). It's concise and should take about 10 minutes to read through.
___

## REST APIs Are Burdened by Dispersed Parameters
REST API parameters are located in several different parts of the HTTP request. In addition to body parameters, there are path parameters, and in the case of `GET` requests, query parameters. While most developers likely do not consider this an inconvenience—probably due to the unquestioning acceptance of the status quo—I would argue that once you experience the simplicity of a JSON-RPC API, you’ll become more aware of this inconvenience yourself.

### Quirky Query Parameters
If you want to retrieve a resource from a REST API endpoint, you must use the HTTP `GET` method. Although _technically_ possible, it is [not recommended](https://developer.mozilla.org/en-US/docs/Web/HTTP/Methods/GET) to include content in the body of an HTTP `GET` request; therefore, any parameters must be sent in the query string portion of the URL.

The problem with query parameters is that they must be URL encoded which has many limitations when compared to JSON, including:
- Lack of support for data types (everything is a string).
- Lack of support for complex data structures (everything is flat).
- Issues regarding URL Encoding.

I find it frustrating that for `POST`, `PUT`, and `PATCH` requests we can use JSON, but as soon as we use a `GET` request we are forced to use query string parameters. It feels like an unnecessary limitation.

Below are just a few examples of the frustrations with query string parameters.

#### Ambiguous Boolean Values
Since query parameters aren't typed, it is ambiguous as to what string value would represent a boolean. Therefore it is common to manually check for the following string values for each boolean:
  - `true`: "true", "t", "yes", "y", "on", "1"
  - `false`: "false", "f", "no", "n", "off", "0"

A recent [post](https://forum.djangoproject.com/t/add-getbool-to-request-query-params-querydict/30985) in the Django community asked for the addition of a `getbool()` function to simplify this common yet annoying scenario. (They did not add it.) Since JSON supports a boolean data type this is not a problem.

#### Improperly Encoded Characters
As a backend developer I have frequently encountered the issue where the client did not properly encode the query string leading to unexpected results on the server side. One common example is with the plus `+` sign. To send a plus `+` as a string in a query parameter, it must be encoded as `%2B`. If the client fails to encode the plus sign correctly, it will be interpreted as a space ` ` character.

Below is an example using Python:
```
from urllib.parse import parse_qs

>>> parse_qs("key=+")  # not encoded correctly
{'key': [' ']}

>>> parse_qs("key=%2B")  # encoded correctly
{'key': ['+']}
```

### Pesky Path Parameters
Path parameters are often used to identify a specific resource. For example, the path `/api/v1/books/123` would refer to the `book` object with an ID of `123`. On the server side, this requires extracting the parameter from the URL.

In most cases, you’re likely using a framework in your programming language of choice that can automatically extract the parameters and cast them to the appropriate type, so this usually isn’t an issue. However, if you don’t have access to a framework—perhaps due to restrictions like closed-source requirements or operating in a memory-constrained environment—extracting that parameter manually can be tedious, at least when compared to the simplicity of a parameter in a JSON object. In the Go programming language, the ability to [extract path parameters](https://go.dev/blog/routing-enhancements) using just the standard library was only recently added.

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

## Challenges in Structuring Error Responses in REST APIs
A key component of API design is the structure of error responses. With REST, there is no formal structure defined for error responses. While this isn’t inherently bad, it places the burden on API producers to design their own error responses. For API consumers, there is no guarantee of consistency across different REST APIs.

### Standardized Error Handling in JSON-RPC
With JSON-RPC, an [Error Object](https://www.jsonrpc.org/specification#error_object) is defined in the specification. It contains two required fields: a `code`, which is always an integer, and a `message`, which is always a string. Additionally, there is an optional `data` attribute, which can either be a primitive or a structured value. The `code` and `message` attributes give every JSON-RPC API a predictable structure, while the `data` attribute provides flexibility for API producers to add custom attributes.
___

## Conclusion
Although REST APIs are the norm when designing a web API, they come with several challenges, such as multiple locations for parameters, a limited naming convention for actions, and the lack of a formal error response structure. JSON-RPC resolves many of these issues and can speed up your development efforts. If this post has sparked your curiosity, I highly encourage you to try JSON-RPC.