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

#### Use Media Types for represent versions
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

