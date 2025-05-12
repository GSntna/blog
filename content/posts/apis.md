---
date: 2025-05-12T10:36:03+02:00
# description: ""
# image: ""
lastmod: 2025-05-12
showTableOfContents: true
tags: ["http", "web", "api", "notes"]
title: "API Basics"
type: "post"
---

## Introduction

Application Programming Interfaces (APIs) act as a software intermediary,
allowing applications to exchange data and requests without needing to know
the internal workings of the other application

| Letter | Word | Meaning |
| ------ | ---- | ------- |
| A | Application | Software that does a task |
| P | Program | Program (P) that does the task in the Application (A) |
| I | Interface | Place (I) to tell the program (P) to run |


Some of the advantages of using an API are:

* The user just uses the program, no need to write it
* Platform independent
* Upgrade safe

## Web services and HTTP

An API made over the internet is called *web service*.

A web service calls the *program* (in API) sending an HTTP (Hyper Text Transfer
Protocol) *request*, it will return a *response* in HTTP as well.

### HTTP Request Structure

The parts of HTTP are:

1. **Start line**:
    * For requests it contains the HTTP method, the requested resource path and
    the HTTP version
    * For response it contains the HTTP version, a *status code* and a status
    message
2. **Headers**: Key-value pairs that provide additional information about the
request or response, such as content type, authentication or caching
instructions.
3. **Blank line**: Separates the headers from the body.
4. **Body**: Contains the data being sent or received, such as HTML, JSON or
other binary.

#### 1. Start Line

| Part | Request | Response |
| ---- | ------- | -------- |
| Name | Start line, Request line | Start line, Response line, Status line |
| HTTP Version | HTTP/1.1 | HTTP/1,1 |
| Method | GET, POST, PUT, DELETE, etc. | None |
| API Program Folder Location | Yes (ex. `/search`) | None |
| Parameters | Yes (ex. `?q=tuna`) | None |
| Status Code | None | Yes (ex. `200 OK`) |
| Format | `Method API_Progam_Folder_Location+Parameters HTTP_Version` | `HTTP_Version Status_code` |
| Example | `GET /search?q=tune HTTP/1.1` | `HTTP/1.1 200 OK` |

#### 2. Headers

[Wikipedia list of headers that can be used](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields)

The most common are:

| Type | Field | Request |
| ---- | ---- | ------- |
| Request | Accept-language | Only accept responses in given language |
| Request | Authorization | Token or API key or username/password |
| Request | Cache-control | Use cache or not |
| Request | Content-type | The media type of the body of the request (json, xml, etc.) |
| Request | Date | Date the request is being sent to the API |
| Request | Host | Domain name of the server (ex. www.google.com) |
| Response | Date | Date that the response is sent |
| Response | Expires | The date/time after wich the response is considered stale |
| Response | Server | A name of the server |
| Response | Set-Cookie | An HTTP Cookie |

#### 4. Body

It contains the content to send to the API if a request is being done, or the
content that comes out from the API if the HTTP is a response.

The type of content that is sent or received is specified in the `content-type`
header. To specify this parameter the syntax `type "/" subtype *[";" parameter]`.

The most [common types of content](https://en.wikipedia.org/wiki/Media_type)
are:

* `application/json` \***Most common for sending/receiving**\*
* `application/ld+json` (JSON-LD)
* `application/msword` (.doc)
* `application/pdf`
* `application/sql`
* `audio/mpeg`
* `audio/ogg`
* `image/avif`
* `image/jpeg`
* `image/png`
* `text/plain`
* `text/css`
* `text/csv`
* `text/html`
* `text/xml` \***Most common for sending/receiving**\*
* `text/javascript`(.js)

### HTTP Methods and Status Codes

#### HTTP Methods

The main methods (CRUD) are these:

* **GET**: READ, get information
* **POST**: CREATE, create information
* **PUT**: UPDATE, change information
* **DELETE**: DELETE, delete information

#### HTTP Status Codes

* **1XX**: Information response
* **2XX**: Success
* **3XX**: Redirection
* **4XX**: Client error (from where it was sent)
* **5XX**: Server error (to where it was sent)

## Types of content

### XML

XML stands for E**X**tensible **M**arkup **L**anguage, it holds data sent or
returned from the API.
* HTTP Header line: `Content-Type:application/xml`
* the body will contain the XML itself.

It is similar to HTML, the difference is that the tags can be anything, as
opposed to HTML where there can only be certain predefined tags.

A schema for the XML can be defined using the
[XSD Schema](https://en.wikipedia.org/wiki/XML_Schema_(W3C)).

```xml
<!--pizza_api.xml-->

<?xml version="1.0"?>
<Pizza>
    <Size>Medium</Size>
    <Toppings>
        <Topping>ketchup</Topping>
        <Topping>onion</Topping>
        <Topping>goat_cheese</Topping>
    </Toppings>
</Pizza>
```

### JSON

**J**ava**S**cript **O**bject **N**otation (JSON) is the part of Javascript that
holds sent/received data.

* HTTP Heder line: `Content-Type:application/json`
* HTTP Body: JSON
* It uses `Key`: `Value`

```json
{"Pizza": [
    {"Size": "Small",
    "Toppings":["ketchup", "onion", "goat_cheese"]
    },
    {"Size": "Large",
    "Toppings":["olives", "anchoes"]
    }
]}
```

*NOTE*: JSON is smaller than XML, which makes it better for larger transactions.

## SOAP and REST

### SOAP

**S**imple **O**bject **A**ccess **P**rotocol (SOAP) is a way to access a web
service by following certain rules to form the HTTP request/response.

* Every API that uses SOAP **uses WSDL** (Web Services Description Language),
which is an XML that describes the web service and **must be followed** when
making a request.
* Always uses XML

### REST
 
**Re**presentational **S**tate **T**ransfer (REST) was introduced a year afer
SOAP and is a more versatile method then SOAP as it can use more
[methods](#http-methods) other than POST.

* REST is more flexible and therefore is more used than SOAP.

### Differences

* Where something is stored in memory. REST uses cache memory to avoid processing
the request if the response is already in memory and the cache expiration has
not been exceeded.
* SOAP uses XML and REST uses JSON, XML, images, or any other format.
* SOAP can only use the HTTP method POST (not specified since it's default).

## API Access

### HTTPS

Whenever data is sent or received using HTTP, anyone can see it. The transfer
can be encrypted and made *secure* using HTTP**S**.

### Authorization and authentication

The difference between these two common concepts is:

* **Authorization**: Limits access to resources
* **Authentication**: Proving your identity using credentials

#### Examples of athentication and authorization in APIs

| Name | Authentication | Authorization | Examples |
| ---- | -------------- | ------------- | -------- |
| No Auth(entication) | N | N | Google search page |
| Basic Auth(entication) | Y | N | email account (user/pwd) |
| Bearer Token | N | Y | Access to google drive file through link |
| OAuth (Open Authorization) | Y | Y | WAZE app to location (give another entity *limited* access to a resource through an access token) | 
| Multi-factor Authentication | Y | N | High security environments |

#### Types of Authorization

##### Basic Authentication

* Uses simple username/password credentials sent in the HTTP header
* Encoded in Base64 format (username)
* Easy to implement but less secure as credentials are passed with every request
* Should only be used with HTTPS/TLS

##### Bearer Authentication

* Uses tokens (typically JWT) instead of credentials
* Client includes token in the Authorization header ("Bearer [token]")
* More secure than Basic Auth as no credentials are transmitted
* Tokens can expire, limiting the risk if intercepted
* Common for single-service authentication

##### OAuth (Open Authorization)

* Complete authorization framework rather than just authentication
* Enables secure delegated access without sharing credentials
* Uses access tokens with configurable permissions and expiration
* Supports different flows (Authorization Code, Client Credentials, etc.)
* Ideal for third-party API integrations and microservices
* Separates authentication from authorization
* Often involves refresh tokens to obtain new access tokens


## Apps

The typs of apps are:

* **Native app**: runs on operating system (OS). Only uses APIs for your OS.
* **Web app**: runs in a browser. Only uses web services.
* **Hybrid app**: has aspects of both

## Python example

Example of using an API in Python

```py
import requests

# Start line components
method = "POST"
url = "https://jsonplaceholder.typicode.com/posts"

# Headers
headers = {
    "Content-Type": "application/json",
    "Authorization": "Bearer abc123token",
    "User-Agent": "MyApp/1.0"
}

# Body (payload)
payload = {
    "title": "Sample Post",
    "body": "This is the content of my post",
    "userId": 1
}

# Send the request
response = requests.request(method, url, headers=headers, json=payload)

# Print the response
print(f"Status Code: {response.status_code}")
print(f"Response Body: {response.text}")
```