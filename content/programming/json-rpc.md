+++
date = '2024-12-30T13:06:40-05:00'
draft = true
title = 'JSON RPC'
+++

## Outline of points I want to make.

- JSON-RPC requests are always JSON. Now this may seem true for RESTful APIs, but there is a common scenario where a different content type is used. When making an HTTP GET request to a REST API, any parameters are to be sent in the query string portion of the URL. The query parameters are encoded using the application/x-www-form-urlencoded content type which is not JSON.

- It is ambiguous to how one should handle actions outside of the typical CRUD operations. What if what you are trying to accomplish doesn't involve creating, retrieving, updating, or deleting a resource? What if you want to send, verify, transfer, or build something? What HTTP method are you supposed to use?

- PUT and PATCH: ...


- Error Responses: In REST, there is no formal structure for errors. It is up to the producer of the API to decide what an error response will look like.

- HTTP Status codes: In REST, there seems to be this coupling of application level errors with HTTP errors. There are many APIs that will usually use a 400 status code and use the structure of the JSON body to contain more details. Other APIs will attempt to use more than just the 400 status code. Regardless, the underlying issue stems from the fact that HTTP status codes were meant to identify specific errors regarding the HTTP Protocol and often have nothing to do with your application.

...

## Why I prefer JSON-RPC over REST.

In this post I'd like to go over the many reasons why I prefer JSON-RPC over REST when designing APIs.Now some may argue that comparing JSON-RPC with REST is like comparing apples to oranges, but I think if they are atleast both fruit then they can be compared. Before I get into the details, I think it'd be best to have a quick overview of REST, and little intro into JSON-RPC in case anyone reading is unfamiliar with it.


## A Review of REST.
Every modern day developer has surely heard of REST or RESTful APIs. The term is everywhere. REST is an acronym that stands for **Re**presentational **S**tate **T**ransfer. The term was first introduced in the year 2000 by Roy Fielding in his [doctoral dissertation](https://ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm). I think the best way to explain REST is to go over the different components of the architecture. First off REST is tightly coupled to the HTTP protocol, so a REST API request always starts with an HTTP request. The URL, of the http request is going to represent some sort of resource. so if we are dealing with an API of books, then the path may look like `/api/v1/books`, and if we are talking about a specific book then the URL would include the id of the book `/api/v1/books/123`. Now we can apply the basic CRUD operations, **Create**, **Read**, **Update**, and **Delete** on the books collection or a specific book. Each CRUD operation is mapped to an HTTP Method.
- GET: Retrieve (Read) several or a single book.
- POST: Create a new book.
- PUT: Replace (update) a book.
- PATCH: Modify (update) a book.
- DELETE: Delete a book.

And there we go, that's a basic review of REST.

## An Intro to JSON-RPC
If you are unfamiliar with JSON-RPC, you can checkout the specification [here](https://www.jsonrpc.org/specification). It takes about 10-15 minutes to read the whole thing.

## JSON-RPC over REST


