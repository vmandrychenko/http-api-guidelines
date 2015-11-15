# HTTP API Guidelines

## RESTful consistency

I love consistency and one of the things that frustrate me is when number of devs working on the project
make up their own standards. It is especially noticeable in the APIs. Many of the APIs start as internal project
and at some point get exposed to the world. Once API is exposed, it is very difficult to change; its important
to get a good start.

I have built number of HTTP APIs over the years and in fact its one of my favourite things to do. I have used
various formats to represent data and have gathered number of guidelines that can help develop clean, easy to
consume RESTful api.

### API (Application Programming Interface)
My philosophy of the API is that it should be very easy to consume and must be human readable. API is
there for computers to communicate, but in the end it will be built and maintained by developers.

### REST
REST (Representational state transfer) is yet another buzzword that has taken over IT world.
I think of REST API as pure HTTP API that allows programmatic navigation through hyperlinks.

### HTTP Headers
Use HTTP Headers to represent API metadata. Headers should be used to specify Media Type, Caching Strategy,
Accepted formats and API version, Resource Location, Encoding. Please, do not use headers to represent the data.
There is payload for that.

### HTTP Payload
There is where data goes. XML is widely used, but my preference is to use JSON to represent the data.
JSON is very light weight and very human readable. JSON can also be very easily consumed by clients
running in the browser. Please avoid any kind of envelopes, like SOAP for example.
Use headers to represent metadata.

### API Version
This is very touchy subject as there are number of preferred ways to deal with versions of the API and
backwards compatibility.

Couple of common ways to version APIs:
   * version in the URI /v1/resource/123
   * version as request param /resource/123?v=1
   * use Media Type:
     * in the media type name
     * added as qualifier

#### Use Media Type headers to represent resource version
I think using Media type is the most appropriate way where version is added as qualifier.
URL should contain resource location info only, while header represents the version.

```
Accept: application/vnd.hostname.userprofile+json;v=1
Content-Type: application/vnd.hostname.userprofile+json;v=1

Accept: application/vnd.hostname.user+json;v=1.2
Content-Type: application/vnd.hostname.user+json;v=1.2
```
As you can see, client can specify the version required in the Accept header
and server specifies the version served in content type header.

Note that Media Type is specified as custom `vnd` (vendor) type with format
in `json`.

This version method is sometimes difficult to implement as it may require
networking equipment to operate on and parse HTTP headers instead of URL if
different versions are served from different servers or network segments.

### Hostname
Use hostname (incl sub name) to represent the domain identifier.
```
{domain}-api.hostname.com

user-api.hostname.com
profile-api.hostname.com
user-preference.api.hostname.com
```
Note: domain word identifies your domain concept, use - to separate if domain concept
consists of more then 1 word.

### Resource names
Resource names must be nouns (with some exceptions, like search, for example).
Resource names should represent you business domain model.

Use plural to represent collections.

```
{domain}.api.hostname.com/resource-collection/{resource-id}/resource-collection/{resource-id}

user.api.hostname.com/users/1234/profiles/52342
user.api.hostname.com/users/1234/account
```
### URI (Unique Resource Identifier)
Do not include query params as part of resource identifier as they are optional as per HTTP specification.
```
{domain}.api.hostname.com/resource-collection/{resource-id}/resource-collection/{resource-id}
{domain}.api.hostname.com/resource-collection/{resource-id}
```
### Format
Type                | Standard                                                | Format                                                     | Examples
------------------- | ------------------------------------------------------- | --------------------------------------------------------   | --------
Resource Name       |                                                         | hyphen-separated, lower case                               | /users, /user-preferences
Query Parameters    |                                                         | camelCase string (low 1st)                                 | ?sortBy=name&order=desc
Headers             | HTTP 1.1                                                | Snake-Case (hyphen, 1st upper                              | Content-Type, If-Match
Media Type          | HTTP 1.1                                                | application/vnd.hostname.{resourcetype}+json|xml;v=version | application/vnd.hostname.user+json;v=2.1
Date                | ISO 8601                                                | YYYY-MM-DD                                                 | 2015-11-14
Date Time           | ISO 8601 (UTC)                                          | YYYY-MM-DDThh:mm:ssZ (Z for UTC, no other timezones)       | 2015-11-14T21:48:13Z
Duration            | ISO 8601                                                | PnYnMnDTnHnMnS                                             | "P23DT23H" and "P4Y" are both acceptable duration representations as all values are optional and can be omitted if value is 0
Currency Code       | ISO 4217                                                | 3 alpha (Caps)                                             | USD, EUR, CAD, AUD
Currency Amount     |                                                         | Number                                                     | 45.99, 2003.56, 5000
Country Code        | ISO 3166-1                                              | 2 alpha (Caps)                                             | US, CA, CH, KZ, UA
Region Code         | ISO 3166-2                                              | CountryCode-RegionCode (2 alpha)                           | CA-ON, CA-AB, UA-71, US-CA
Language Code       | ISO 6391 + ISO 6393 (for dialects not covered by 6391)  | 2 alpha or 3 alpha                                         | en, es, it, hi, zh, cmn (chinese mandarin), cjy (chinese jin)
Airport Code        | IATA                                                    | 3 alpha (Caps)                                             | YYZ,LAX,AMS

### Request
#### HTTP Methods
* POST - create, can only be done on collection (/users)
* PUT/PATCH - update/partial update, can only be done on existing resource URI, idempotent
* GET - idempotent
* DELETE - idempotent
* OPTIONS - to see what methods are available on the resouces

### Response
#### Status Codes
* Use one of 200s to represent success
  * Use 201 for successful POST with Location header
  * Use 202 for asynchronous interactions
  * Use others appropriately as per spec
* Use one of 300s to represent redirections
  * Use others appropriately as per spec
* Use one of 400s to represent user error. User MUST make changes to the request and try again.
  * Use 400 for malformed payload (malformed JSON)
  * Use 401 to communicate that client is unauthorized
  * Use 403 for invalid permissions (read vs write)
  * Use 422 for valid JSON/XML but invalid semantic errors, like invalid airport code, currency code or invalid date precedence.
  * Use others appropriately as per HTTP spec
* Use one of 500s to represent server error. Users should not retry requests with 500 responses.
  * 502 - to represent failure in the upstream response (provider is down for example).
  * Use other appropriately as per spec

#### Error Response
Header:
```
Content-Type: application/vnd.hostname.errors+json;v=1
```
Payload Body
```javascript
{
  "errors": [
    {
      "code": "mandatory|error code |snake case, all caps|USER_NOT_FOUND",
      "description": "mandatory|verbose description of the error that developer will use for troubleshooting, user will not see",
      "field": "optional|field name, shown during validation errors for specific fields"
    }
  ]
}
```
Example
```javascript
{
  "errors": [
    {
      "code": "USER_NOT_FOUND",
      "description": "user with registrationDate=2014-05-14T15:22:11Z and lastLoginDate=2014-05-16T15:22:11Z cannot be found."
    }
  ]
}
```
“code” field in the error response will be used to for specifying exactly what happened and may be used for driving UI logic. Code field should be self descriptive with all capital letters separated by the underscore (USER_NOT_FOUND, INVALID_LAST_NAME)

There are 2 types of codes:
* HTTP Status codes, integers
  * client can get these from HTTP request
* Custom codes that go in addition to HTTP Status codes - Error object, Code field
  * These are Enum values, constants.
  * Client will get these from Error object in the response
  * error.code is generally required in addition to HTTP status code to precisely identify the issue.
  * These are also checked in addition to HTTP status codes against to determine Error message/Error screen to display for the user.

So in example where POST /users request is made to the server and payload contains invalid email (johngmail.com as an example)

From API perspective you will return:
* HTTP 422 - as it is semantic error where serve fully understands the request (don't use 400 in this case)
JSON payload with header Content-Type: application/vnd.hostname.errors+json;v=1
```javascript
[
  {
    "code": "INVALID_EMAIL",
    "description": "email johngmail.com is invalid, its missing @ character",
    "field": "email"
  }
]
```
As you can see, from HTTP 422 it's impossible to determine that there is a problem with the email, this is when the code and field combination is used to determine the action on the UI side.

### Media Type
use `Accept` and `Content-Type` http headers to specify custom media type.

### Encoding
* Use UTF-8
* JSON is UTF-8 by default

### Retrieval and Search
##### specific resource by id
```
/users/12345
```
##### collection of resources by attributes
```
/users?countryCode=US
```
##### collection of resources by range
use startXXX and endXXX query parameters
```
/users?registrationStartDate=2014-05-14T15:22:11Z&registrationEndDate=2014-05-14T18:30:00Z
```


### Pagination
use offset and limit query parameters
* offset = 0, limit = 10 is 0 - 9
* offset = 10, limit = 10 is 10 - 19
```
/users?offset=10&limit=10
```

### Sorting
use sortBy and order query parameters
* sortBy - sort criteria with values comma separated
* order - specify sort order
```
?sortBy=X,Y,Z&order=asc|desc
?sortBy=FirstName,LastName&order=asc
```
### Hyperlinking and HATEOAS
As per rfc5988 Link HTTP header should be used to define links.

According to Roy Fielding (principal author of HTTP):
> What needs to be done to make the REST architectural style clear on the notion that hypertext is a constraint? In other words, if the engine of application state (and hence the API) is not being driven by hypertext, then it cannot be RESTful and cannot be a REST API. Period. Is there some broken manual somewhere that needs to be fixed?

Note: there are other common ways to do hyperlinking
* HAL
* ATOM

### Resource Versioning
Use Etag headers as per HTTP to represent Resource version.

### Cache Control
Use HTTP built in Cache mechanisms and Cache Control Headers  as these are supported by wide varieties of servers, switches and other network devices.
When we talk about Cache, I am talking about Caching HTTP Resources (like we cache all other HTTP resources: Images, CSS, JS, etc)

### Optimistic Concurrency Control
Etags in combination with HTTP Cache Headers (e.g If-Match) should be used for Optimistic Concurrency Control

### Security
* APIs exposed to the world require Authentication and Authorization
* APIs that contain security credentials and must be exposed over HTTPS

##### HTTP Basic
Http Header: `Authorization`
Http Header Value: Basic {base64 encoded username and password}

As per wiki page HTTP header constructed as follows:
* Username and password are combined into a string "username:password"
* The resulting string literal is then encoded using the RFC2045-MIME variant of Base64, except not limited to 76 char/line[9]
* The authorization method and a space i.e. "Basic " is then put before the encoded string.

HTTP Header Example:
```
Authorization: Basic QWxhZGRpbjpvcGVuIHNlc2FtZQ==
```

### Gzip Compression
For the performance reason gzip compression is required to ensure payloads are transferred faster over the wire.



