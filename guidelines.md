# Concur API Guidelines

* [Guidelines](#guidelines)
	* [Transport Level Security (TLS)](#tls)
	* [Domain Name System (DNS)](#dns)
	* [Authorization](#authorization)
	* [Security Architecture Review Board](#security)
	* [API Gateway (Reverse Proxy)](#api-gateway)
	* [Headers](#headers)
		* [Concur Correlation Identifier Custom Header](#header-concur-correlationid)
		* [Content-Type](#header-content-type)
		* [Accept](#header-accept)
		* [Accept-Language](#header-accept-language)
		* [Date](#header-date)
		* [Location](#header-location)
		* [Vary](#header-vary)
		* [Cache Headers](#header-cache)
		* [Last-Modified](#header-last-modified)
		* [ETag](#header-etag)
		* [Precondition Headers](#header-precondition)
		* [Authorization](#header-authorization)
	* [Resource Naming](#resource-naming)
		* [Collection and Item Pattern](#collection-item-pattern)
		* [Hierarchical Pattern](#hierarchical-pattern)
		* [Creation of Resources and Representations](#creating-resources)
		* [Use of the query component in naming resources](#resource-naming-query)
		* [Use of the #fragment identifier](#uri-fragment-identifier)
		* [Resource Naming Syntax](#resource-naming-syntax)
		* [Friendly resource name pattern](#resource-naming-friendly)
	* [Hypermedia as the Engine of Application State](#hypermedia)
	* [Related Data](#related-data)
	* [Custom Data](#custom-data)
	* [Service Index](#index)
	* [Pagination](#pagination)
	* [Data Design](#data)
		* [Reserved Key Names](#data-reserved-keys)
		* [Identifiers](#data-identifiers)
		* [User Identification](#data-user-identification)
		* [Company Identification](#data-company-identification)
		* [Self-describing Data](#data-self-describing)
		* [Dates and Times](#data-date-time)
		* [Currency](#data-currency)
		* [Key-Value Pair Names](#data-key-names)
	* [HTTP Status Codes](#http-status-codes)
		* [Errors when HTTP Status Code is 4xx](#errors-when-4xx)
	* [Service Changes](#service-changes)
		* [Breaking](#service-changes-breaking)
		* [Non-Breaking](#service-changes-non-breaking)
	* [Documentation](#documentation)
		* [Markdown](#documentation-markdown)
		* [Schema](#documentation-schema)
		* [OpenAPI](#documentation-openapi)
* [Works Cited](#works-cited)

## <a name="guidelines"></a>Guidelines

### <a name="tls"></a>Transport Level Security (TLS)

* Services MUST communicate using HTTPS.

### <a name="dns"></a>Domain Name System (DNS)

* Services MUST identify and discover one another using [RFC 1034 Domain Names](#RFC-1034).

### <a name="authorization"></a>Authorization

> For more information on the OAuth2 service see [https://github.concur.com/core/oauth2/wiki](https://github.concur.com/core/oauth2/wiki).

* Services SHOULD make use of X.509 client certificates for service-to-service (internal) authorization.
* Services SHOULD leverage the access token (in the form of a [RFC 7519 JSON Web Token (JWT)](#RFC-7519)) for user oriented authorization and / or external access to APIs.
* Services SHOULD use data contained in an access token only for authorization and for no other purpose. For more information see [Using JSON Web Token (JWT) access token for purposes other than authorization](./using-jwt-for-purposes-other-than-authorization.md).

#### Comparison of Authorization and Authentication

* Authorization (AuthZ) is "the function of specifying access rights to resources related to information security and computer security in general and to access control in particular. More formally, 'to authorize' is to define an access policy." (Source: [Wikipedia](https://en.wikipedia.org/wiki/Authorization))
* Authentication (AuthN) is "the act of confirming the truth of an attribute of a single piece of data (a datum) claimed true by an entity." (Source: [Wikipedia](https://en.wikipedia.org/wiki/Authentication)). For API usage at Concur this most often occurs with a user login, basic (username + password) or single sign on (SSO).
* Authorization and authentication are usually tightly coupled (referred to as AuthX); Client code typically enters the [RFC 6749 OAuth 2.0 Authorization Framework](#RFC-6749) authorization process which in turn authenticates the user. In API usage it is rare for client code to perform the authentication directly.

#### API Gateway and Authorization

> * For a video on AuthX and the API Gateway see [https://concur.circlehd.com/play/rkMIU1yU-Concur-749_023-Auth-Overview-Video-1](https://concur.circlehd.com/play/rkMIU1yU-Concur-749_023-Auth-Overview-Video-1).
> * For specific information on AuthX and the API Gateway see [http://developer.concurasp.com/apigateway/apigw_auth.html](http://developer.concurasp.com/apigateway/apigw_auth.html).
> * For specific information on the API Gateway custom headers see [http://developer.concurasp.com/apigateway/apigw_usage.html#gateway-headers](http://developer.concurasp.com/apigateway/apigw_usage.html#gateway-headers)

The API Gateway does authentication for services and provides authorization context. At a very high level the API Gateway validates both access tokens (which are in the form of JWTs) and X.509 certificates. Services are still responsible for inspecting the authorization context in access tokens (via claims in the JWT) and / or information about the callers X.509 certificates passed in API Gateway custom headers.

This subject is very deep so teams are encouraged to follow the links above to learn more.

### <a name="security"></a>Security Architecture Review Board aka SARB

The purpose of the SARB is to review projects and significant material changes to SAP Concur Global IT infrastructure and Applications. To ensure that your service is compliant with SAP Concur's security and privacy controls, please engage the [Security Architecture Review Board](https://wiki.concur.com/confluence/display/SEC/Security+Architecture+Review+Board). The SARB is a part of the [SAP Concur Delivery Framework](https://wiki.concur.com/confluence/display/SDLC/SAP+Concur+Delivery+Framework).

For more information contact the Security Team at the [Security Architecture Review Board DL](mailto:DL_05B049A106C04F72BC8DA0C76FEB8053@sap.com) or in #ask-sarb.


### <a name="api-gateway"></a>API Gateway (Reverse Proxy)

> For more information on the API Gateway see [https://github.concur.com/core/api-gateway/wiki](https://github.concur.com/core/api-gateway/wiki).

* Services SHOULD make themselves available for public consumption through the [Concur API Gateway](https://github.concur.com/core/api-gateway/wiki).

### <a name="headers"></a>Headers

* Services SHOULD limit themselves to standards based HTTP headers as defined in the [Internet Assigned Numbers Authority (IANA) Message Headers](http://www.iana.org/assignments/message-headers/message-headers.xhtml) (Protocol=HTTP, Status=Standard)
* Services SHOULD limit themselves to custom headers as defined for usage by the whole of Concur in this document.

#### <a name="header-concur-correlationid"></a>Concur Correlation Identifier Custom Header

The Concur Correlation Identifier (`concur-correlationid`) in the form of a [RFC 4122 Universally Unique IDentifier (UUID)](#RFC-4122) UUID4 value is a custom header designed to aid in workflow tracking and troubleshooting for both client and server code.

* Services SHOULD generate when not present in a request.
* Services SHOULD be careful to pay attention to incoming `concur-correlationid` values and pass them along, otherwise the service will risk masking bugs or making it more difficult to troubleshoot.
* Services SHOULD include in all requests and responses.
* Services SHOULD store in their data (including logs).
* Services SHOULD NOT document externally as a request header for any API calls by client code.
* Services SHOULD document as a response header.

##### <a name="header-concur-correlationid"></a>Lifecycle

* The `concur-correlationid` is designed to trace each single API call to a single starting endpoint and the resulting routing throughout the entire system.

```
concur-correlationid: "abc-123"

--> Request
<-- Response

Client  "abc-123"  -->  Server A

                    |
                    |             ---
                    |              |   "abc-123"  -->  Server B
                    |              |  <-------------- "abc-123"
                    |   Server A   |
                    |   (Client)   |   "abc-123"  -->  Server C
                    |              |  <-------------- "abc-123"
                    |             ---
                    |        

Client  <--  "abc-123"  Server A
```

#### <a name="header-content-type"></a>Content-Type

* Services SHOULD always provide the [RFC 7231 Content-Type](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) header field in all responses.

#### <a name="header-accept"></a>Accept

* Services SHOULD respect the [RFC 7231 Accept](https://tools.ietf.org/html/rfc7231#section-5.3.2) request header field whenever possible.
* Services SHOULD provide a [406 Not Acceptable](https://tools.ietf.org/html/rfc7231#section-6.5.6) HTTP status code when unable to respect the Accept header field.
* Services MAY disregard the Accept header field and treat the response as if it is not subject to content negotiation provided the [Content-Type](#header-content-type) header is present in the response.

#### <a name="header-accept-language"></a>Accept-Language

* Services SHOULD respect the [RFC 7231 Accept-Language](https://tools.ietf.org/html/rfc7231#section-5.3.5) request header field and respond with the appropriately localized data when applicable.
* The Accept-Language request header field uses [RFC 5646 Tags for Identifying Languages](#RFC-5646). Simplest examples are:
	* en-US
	* de-DE
* Services MAY default to US English (en-US) in the absence of the Accept-Language request header field.

#### <a name="header-date"></a>Date

* Services SHOULD provide the [RFC 7231 Date](https://tools.ietf.org/html/rfc7231#section-7.1.1.2) response header field.

#### <a name="header-location"></a>Location

* Services SHOULD provide the [RFC 7231 7.1.2. Location](https://tools.ietf.org/html/rfc7231#section-7.1.2) response header field with the value expressed as a fully qualified domain name (FQDN) [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) for all operations which use the `POST` HTTP method.
* Services MAY provide the Location header as a relative value noting this requires client code to build URIs which should generally be avoided.

#### <a name="header-vary"></a>Vary

* Services MAY choose to include the [RFC 7231 Vary](https://tools.ietf.org/html/rfc7231#section-7.1.4) header field in a response.

#### <a name="header-cache"></a>Cache Headers

For a full understanding of caching see [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234). This section along with the [Last-Modified](#header-last-modified), [ETag](#header-etag) and [Precondition](#header-precondition) sections cover the caching responsibilities for a service.

* Services SHOULD help clients with cacheability of responses though the use of headers.
	* [RFC 7234 Age](https://tools.ietf.org/html/rfc7234#section-5.1)
	* [RFC 7234 Cache-Control](https://tools.ietf.org/html/rfc7234#section-5.2)
		* [RFC 7234 Response Cache-Control Directives](https://tools.ietf.org/html/rfc7234#section-5.2.2)
	* [RFC 7234 Expires](https://tools.ietf.org/html/rfc7234#section-5.3)
	* [RFC 7234 Pragma](https://tools.ietf.org/html/rfc7234#section-5.4)
	* [RFC 7234 Warning](https://tools.ietf.org/html/rfc7234#section-5.5)

#### <a name="header-last-modified"></a>Last-Modified

* Services SHOULD provide the [RFC 7232 Last-Modified](https://tools.ietf.org/html/rfc7232#section-2.2) header in all responses.

#### <a name="header-etag"></a>ETag

* Services SHOULD provide the [RFC 7232 ETag](https://tools.ietf.org/html/rfc7232#section-2.3) header in all responses.

#### <a name="header-precondition"></a>Precondition Headers

* Services SHOULD support the following request header fields:
	* [RFC 7232 If-Match](https://tools.ietf.org/html/rfc7232#section-3.1)
	* [RFC 7232 If-None-Match](https://tools.ietf.org/html/rfc7232#section-3.2)
	* [RFC 7232 If-Modified-Since](https://tools.ietf.org/html/rfc7232#section-3.3)
	* [RFC 7232 If-Unmodified-Since](https://tools.ietf.org/html/rfc7232#section-3.4)
	* [RFC 7232 If-Range](https://tools.ietf.org/html/rfc7232#section-3.5)
* Services SHOULD respond with the following HTTP status codes to conditional requests:
	* [304 Not Modified](https://tools.ietf.org/html/rfc7232#section-4.1)
	* [412 Precondition Failed](https://tools.ietf.org/html/rfc7232#section-4.2)

#### <a name="header-authorization"></a>Authorization

* Services which make use of access tokens which contain data (such as [JSON Web Token](./jwt-summary.md)) SHOULD NOT leverage the data therein for any portion of the request other than authorization.

Generally speaking a well designed interface of an API will not be aware of the authorization mechanism being used and this authorization mechanism can be replaced wholesale without affecting any client code whatsoever other than authorization pieces. In other words: Authorization schemes can be changed / replaced and this action only affects the `Authorization` header.

### <a name="resource-naming"></a>Resource Naming

The following is a direct copy+paste from [RFC 3986 Uniform Resource Identifier (URI): Generic Syntax](#RFC-3986) as a baseline for the balance of this section.

```
  foo://example.com:8042/over/there?name=ferret#nose
  \_/   \______________/\_________/ \_________/ \__/
   |           |            |            |        |
scheme     authority       path        query   fragment
```

* All portions of a URI SHOULD be in lowercase.
* Included in the path in the following order:
	* Services SHOULD include their name in the first URI path segment.
		* Service names SHOULD be alphanumeric characters only.
		* Service names SHOULD NOT include `service` in their name.
		* Service names SHOULD NOT include punctuation.
	* Services SHOULD provide a [Service Index](#index) at this root path.
	* Services MAY include a version number in the URI path using the form of vX where X is a positive integer: `v1`, `v2`, ... , `v10` in the second URI path segment.
	* Service MAY include a resource name.
	* Services MAY include a resource identifier.
	* Services MAY include a representation name.
* Services SHOULD have unique names.
	* `/stores`
	* `/aisles`
* Services SHOULD NOT share paths.
	* Examples:
		* `/stores/reports/v1` and `/stores/locations/v1`
		* `/stores/v1/reports/v1` and `/stores/v1/locations/v1`
	* Reasons:
		* Service(s) would have to implement a reverse proxy; The API Gateway only handles the first path segment.
		* Versioning overload makes it difficult to independently evolve services.
		* Service(s) would have to manage top level disaster recovery; The API Gateway handles.
		* It breaks the consistency model of `{service}/{version}/{collection}/{item}/{representation}`.

#### <a name="collection-item-pattern"></a>Collection and Item Pattern

* Services SHOULD arrange and name resources according to a `/collection/item` paradigm.
  * `collection` name is plural.
  * `item` name is singular.
* Services SHOULD have a base collection, the root of which is the [Service Index](#index) resource.
* The base collection MAY contain individual items.
* The base collection MAY contain additional collections.

```
https://example.com/stores/v1                                            // The base collection.
https://example.com/stores/v1/76cc758e256c438b8e49546e0102b8c8           // A single item within the collection.

https://example.com/stores/v1/aisles                                     // A related collection within the same service.
https://example.com/stores/v1/aisles/c66f06fdb31b4882ad995e4d19ca7aed    // A single item within the collection.

https://example.com/stores/v1/schemas                                    // A schema collection within the service.
https://example.com/stores/v1/schemas/com-example-store-2018-03-01       // A single item within the collection.
```

#### <a name="hierarchical-pattern"></a>Hierarchical Pattern

> The [Collection and Item Pattern](#collection-item-pattern) is preferred.

* Services MAY use paths which describe a parent-child hierarchy of resources available within the domain of the service.
* This pattern:
  * Can cause resources to be identified in an overly complex way.
  * Can reveal / leak implementation details like data storage paradigms, business logic, business unit seams and more.
  * Can more easily lead to Remote Procedure Call (RPC) architecture style.
  * Usually duplicates data which should be or already is present in the resource itself.
* This pattern MUST NOT be a substitute for proper handling of [Hypermedia as the Engine of Application State](#links-hateoas) or [Related Data](#related-data).

```
Template: https://example.com/{service}/{version}/{identifier}{child}/{identifier}.../{child}/{identifier}
Example:  https://example.com/stores/v1/76cc758e256c438b8e49546e0102b8c8/aisles/df62491e95f54fe1a3ef9bdf40c4d1f5/shelves/c66f06fdb31b4882ad995e4d19ca7aed
```

#### <a name="creating-resources"></a>Creation of Resources and Representations

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* Services MAY create as many representations as is needed.
	* Services are encouraged to do so in order to logically order the resources and representations.
	* Services are encouraged to do so to avoid overcomplicating paths.
* Services MAY make the same resource or representation available via multiple paths.
* Services MAY make subsets of resources available in representations.

##### <a name="creating-resources-example"></a>Examples

Item: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}",
  "schema": "https://example.com/schemas/com-example-store-2018-03-01",
  "name": "Alpha",
  "phone": "(425) 555-1212",
  "operations": [
    {
      "rel": "add-aisle",
      "href": "https://example.com/stores/aisles",
      "method": "POST"
    }
  ]
}
```

* A representation containing only the metadata and one key value pair, excluding all other data and the HATEOAS operations.
* A pattern of `resource/representation` has been used in naming: `76cc758e256c438b8e49546e0102b8c8/phone`

Representation: `https://example.com/stores/76cc758e256c438b8e49546e0102b8c8/phone`

```json
{
  "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8/phone",
  "id": "76cc758e256c438b8e49546e0102b8c8",
  "template": "https://example.com/stores/{id}/phone",
  "schema": "https://example.com/schemas/com-example-store-2018-03-01",
  "phone": "(425) 555-1212"
}
```

* A representation which provides all items in the collection, excluding the HATEOAS operations.
* A naming pattern of `resource/representation` is still present: `/all`; The `resource` is implied and is the base collection.

Collection: `https://example.com/stores/all`

```json
[
  {
    "href": "https://example.com/stores/76cc758e256c438b8e49546e0102b8c8",
    "id": "76cc758e256c438b8e49546e0102b8c8",
    "template": "https://example.com/stores/{id}",
    "schema": "https://example.com/schemas/com-example-store-2018-03-01",
    "name": "Alpha",
    "phone": "(425) 555-1212"
  },
  {
    "comment": "...another store..."
  }
]
```

#### <a name="resource-naming-query"></a>Use of the query component in naming resources

* Services MAY use the query component of a URI as non-hierarchical data to identify resources.
* Services SHOULD NOT change the nature of a resource in response to the presence of the query component.
* Services MAY change the representation provided for a resource in response to the presence of the query component.

##### <a name="resource-naming-query-example"></a>Examples

```
https://example.com/things?userId=foo          // Array of things for a user.
https://example.com/things?companyId=baz       // Array of things for a company.
https://example.com/things/123?userId=foo      // Single item with user query component.
https://example.com/things/123?companyId=baz   // Single item with company query component.
```

#### <a name="uri-fragment-identifier"></a>Use of the #fragment identifier

> See [RFC 3986 Section 3.5. Fragment](https://tools.ietf.org/html/rfc3986#section-3.5)

* Services SHOULD NOT use the fragment identifier in their naming pattern.
* Services SHOULD ignore the fragment identifier (`https://example.com#fragment`) in requests.

The fragment identifier should be separated from the rest of the URI prior to a dereference by the user agent (client code). Most web browsers and development tools for HTTP behave in this manner. That said, it is possible some client code inconsistent with RFC 3986 may be authored in such a way as to send fragment identifiers to the server.

For example, use of a fragment identifier with `cUrl` shows the tool removes the fragment prior to sending the request to the server.

```
curl -v https://developer.concur.com/api-reference/receipts/get-started.html#overview <-- fragment present

> GET /api-reference/receipts/get-started.html <-- fragment removed by tool
> Host: developer.concur.com
> User-Agent: curl/7.54.0
> Accept: */*
```

#### <a name="resource-naming-syntax"></a>Resource Naming Syntax

Building upon everything in this section the following illustrates how teams should think of URIs when naming resources:

```
                  service  version   resource        identifier         representation
                        |    |       |                   |                    |
                       _|__  |   ____|__   ______________|_______________   __|_
                      /    \ /\ /       \ /                              \ /    \
  https://example.com/stores/v1/products/71b1d7acbb254e05b7f9060b0a29efab/digest?name=ferret
  \___/   \_________/ \________________________________________________________/ \_________/
    |          |                                  |                                   |
 scheme    authority                             path                               query
```

* Naming paradigms and patterns for the `path` are optional and flexible.
* All of the following are valid:

```
https://example.com/stores/v1/                                            // Base collection
https://example.com/stores/v1/{collection}                                // Collection within the service.
https://example.com/stores/v1/{identifier}                                // Single item within the base collection.
https://example.com/stores/v1/{representation}                            // Representation of the base collection.
https://example.com/stores/v1/{collection}/{identifier}                   // Single item within a collection.
https://example.com/stores/v1/{identifier}/{representation}               // Representation of a single item within the base collection.
https://example.com/stores/v1/{collection}/{identifier}/{representation}  // Representation of a single item within a collection.
```

#### <a name="resource-naming-friendly"></a>Friendly resource name pattern

There are times when resource names should have friendly names readable by humans, for example:

* When the client code chooses the resource name with a `PUT` operation creating a new resource.
* When the resource is more widely used across client code, for example: JSON Schema.

In order to avoid name collisions a pattern MAY be agreed upon and it is suggested that pattern be, in part, based on how [RFC 1034 Domain Names - Concepts and Facilities](#RFC-1034) works:

```
{generic top-level domain (gTLD)}
 -{domain}
  -{second and lower level domains separated with a -}
   -{schema name}
    -{version}

Examples:

  com-example-baseball-equipment-glove-2018-03-01
  org-example-soccer-worldcup-2018-39f6256e
```

* Services SHOULD refrain from using dots / periods (`.`) in resource names.
  * Approaching friendly names from the perspective of a file based system may come a desire to append a file name to the end a resource name, for example: `com-example-baseball-equipment-glove-2018-03-01.schema.json`. This may be shortsighted and may appear to be in conflict with the [RFC 7231 Content-Type](https://tools.ietf.org/html/rfc7231#section-3.1.1.5) header should a server decide to make multiple formats available in the future.

### <a name="hypermedia"></a>Hypermedia as the Engine of Application State

> A primer on Hypermedia as the Engine of Application State can be found [here](./hateoas-model-example.md).

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* APIs SHOULD expose the full application state to the consumers
* Services SHOULD make links available only when they are valid for the state of the service at the time of the response.
* Hypermedia as the Engine of Application State describes what client code can do with the server.
* The "application state" portion of the concept refers to the server (not the client).
* It is encapsulated within an `operations` array.
* Client code binds to the `rel` (relation) key and follows the link in the `href` key using the HTTP method in the `method` key.
  * This insulates against breaking changes as it allows the server to change the `href` value at any time and the client code will still work.

#### <a name="hypermedia-operations"></a>Operations Schema

Name | Type | Format | Description
-----|------|--------|------------
`operations`|`array`|[`operation`](#hypermedia-operations-operation)|The hypermedia as the engine of application state (HATEOAS) information.

#### <a name="hypermedia-operations-operation"></a>Operation Schema

Name | Type | Format | Description
-----|------|--------|------------
`rel`|`string`|[RFC 5988 Relation Type](https://tools.ietf.org/html/rfc5988#section-4)|**Required** Relation type as defined by the server. There are registered relation types listed in [RFC 5988 6.2.2. Initial Registry Contents](https://tools.ietf.org/html/rfc5988#section-6.2.2) including pagination relation types of `next`, `prev`, `first` and `last`.
`href`|`string`|[RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986)|**Required** Hyperlink to the resource. This key name is borrowed from the HTML5 `href` element and behaves similarly.
`method`|`string`|[RFC 7231 Methods](https://tools.ietf.org/html/rfc7231#section-4.3)|Default is GET when `method` is not specified. Valid method names are RFC 7231 GET, HEAD, POST, PUT, DELETE, CONNECT, OPTIONS, TRACE + RFC 5789 PATCH.

#### Example

* See the [Service Index](#index) or [Pagination](#pagination) sections for examples.

### <a name="related-data"></a>Related Data

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

> One example (of many approaches) for resources, representations and related data can be found in [Resources and Representations](./resource-and-representation.md).

* All relationships to data outside of the document SHOULD be described by fully qualified domain name (FQDN) `URI` resource links.
	* There are exceptions to this guideline where the key name is one of the [Reserved Key Names](#data-reserved-keys).
* It is acceptable for data to reference other data which does not have a corresponding reference back; In the example given it would be fine if the products did not have a link to the store.
* Related data can reference resources outside of the domain.

#### Example

In the following example:
* `products` is the items available for sale in a store.
* `aisles` is an array of aisles for the store.
* `map` is a hyperlink to the location for the store within a external domain.

Contents of https://example.com/stores/v1/abc:

```json
{
  "name":"Example",
  "products": "https://example.com/products/v1/123",
  "aisles": [
    "https://example.com/aisles/v1/1",
    "https://example.com/aisles/v1/2"
  ],
  "map": "https://www.google.com/maps/place/1820+Wensley+Dr,+Charlotte,+NC+28210/@35.1541824,-80.8694037,17z"
}
```

### <a name="custom-data"></a>Custom Data

> An ongoing investigation and design effort can be found at https://github.concur.com/developer/schema.

* Services SHOULD NOT use ambiguous key names such as `custom1`.
* Services SHOULD use a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986) to disambiguate data and avoid name collisions.

Name | Type | Format | Description
-----|------|--------|------------
`customData`|`array`|-|An array of `object`, each of which is disambiguated with a reference to a schema.

```json
{
  "comment": "The customData key would be in a larger JSON document.",
  "customData": [
    {
      "schema":"https://example.com/schema/be9e17e751bd4dc690ff3d079c8808fe",
      "foo": "hello",
      "baz": "world"
    },
    {
      "schema":"https://example.com/schema/someotheridentifier",
      "alpha": "bravo",
      "delta": "tango",
      "echo": "foxtrot"
    }
  ]
}
```

### <a name="index"></a>Service Index

> More information on the service index design and schema can be found in [Service Index](./service-index.md).

* Services SHOULD provide a service index of links at the base URI which will be interesting to callers of the service.
* The links SHOULD follow the [RFC 5988 Web Linking](#RFC-5988) standard.
* The links MAY use [RFC 6570 URI Template](#RFC-6570).
* The service index SHOULD be made available at the root of the service: `https://example.com/{servicename}`.

The service index is one of the keys to evolvability of services, allowing client code to:

* Reference an abstracted name rather than a link directly which allows service links to change without requiring client code changes.
* Simply follow given links rather than using `string` builders or concatenation to craft service links.

### <a name="pagination"></a>Pagination

* Services SHOULD always determine the pagination scheme rather than allowing client code to define (for example, with a query string parameter) to allow for service design and performance tuning independent of client code.
* Services SHOULD provide pagination operations in all responses using the following relation names according to the [RFC 5988 Web Linking](#RFC-5988) standard:
	* `next` - refers to the next resource in a ordered series of resources.
	* `prev` - refers to the previous resource in an ordered series of resources.
* Services MAY also use the following relation names as desired:
	* `first` - refers to the furthest preceding resource in a series of resources..
	* `last` - refers to the furthest following resource in a series of resources.
* Services MAY use any sort of name to denote pages of data. `page` is acceptable as would be any other name which makes sense internally to the service itself. Naming is of less importance here because client code simply follows the links rather than crafting URIs.
* Services MAY return a `totalCount` property that indicates the total number of items for a collection resource. 
* For more information see [Pagination Design](/pagination-design.md)

#### Schema

Pagination leverages the [Hypermedia as the Engine of Application State](#hypermedia) [`operations`](#hypermedia-operations) schema.

#### Example

```json
{
  "data": [
    { "id": "Alpha" },
    { "id": "Bravo" },
    { "id": "Charlie" }
  ],
  "operations": [
    {
      "rel": "next",
      "href": "https://example.com/service/v1/page4"
    },
    {
      "rel": "prev",
      "href": "https://example.com/service/v1/page2"
    },
    {
      "rel": "first",
      "href": "https://example.com/service/v1/first"
    }
    {
      "rel": "last",
      "href": "https://example.com/service/v1/last"
    }
  ],
  "totalCount": 3
}
```

### <a name="data"></a>Data Design

#### <a name="data-reserved-keys"></a>Reserved Key Names

The following key names are important across all domains at Concur. They can be used in data only within the context of the service defining the key name.

Name | Type | Format | Description
-----|------|--------|------------
`userId`|`string`|[`UUID4`](#RFC-4122)|A user within the Concur system: [User Identification](#data-user-identification)
`companyId`|`string`|[`UUID4`](#RFC-4122)|A company within the Concur system: [Company Identification](#data-company-identification)

#### <a name="data-identifiers"></a>Identifiers

> The differences between [Hypermedia as the Engine of Application State](#hypermedia), [Related Data](#related-data) and [Identifiers](#data-identifiers) are explained [here](./id-related-data-hateoas.md).

* Services SHOULD identify unique resources with a string in the format of a [RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace](#RFC-4122) UUID4 without dashes in a key named `id`.
	* This value SHOULD be separate from database key values (i.e., primary key) to avoid leaking implementation details.
* Services SHOULD provide a URI as the identifier (including the UUID4 value stored in `id`) for the resource in a key named `href`.
	* The key name `href` is based on the [HTML5 concept of links](https://www.w3.org/TR/html5/links.html), [RFC 5988 Web Linking Appendix A Notes on Using the Link Header with the HTML4 Format](https://tools.ietf.org/html/rfc5988#appendix-A) and is consistent with other sections of the guidelines where hyperlinks are provided using the Hypermedia as the Engine of Application State [Operations Schema](#hypermedia-operations-operation) such as [Related Data](#related-data), [Service Index](#index) and [Pagination](#pagination).
* Services SHOULD provide a [RFC 6570 URI Template](#RFC-6570) in a key named `template`.
	* The combination of `template` + `id` = `href` allows legacy relational database systems at Concur the flexibility to store URI values minimally to avoid database bloat (mainly due to indexing) and therefore storage costs.
* `href`, `id` and `template` SHOULD be considered reserved keywords.
  * `href` and `id` SHOULD only be used in the root of a representation document.
  * `template` SHOULD be used as a key name only when the value is a URI template.

##### Example

```json
{
  "href": "https://example.com/service/v1/ae7d9679708f48e2951bbefd478b3d16",
  "id": "ae7d9679708f48e2951bbefd478b3d16",
  "template": "https://example.com/service/v1/{id}",
  "comment": "...and additional resource data follows."
}
```

#### <a name="data-user-identification"></a>User Identification

> For more information on the Profile service see [https://github.concur.com/core/profile/wiki](https://github.concur.com/core/profile/wiki).

* Services SHOULD store the user identifier with `userId` as the key name and the [RFC 4122 Universally Unique IDentifier (UUID)](#RFC-4122) UUID4 value as provided by the Profile service.

```json
{
  "userId": "6a269d1c-5317-40c4-b0f2-fd7aca682070"
}
```

#### <a name="data-company-identification"></a>Company Identification

> For more information on the Profile service see [https://github.concur.com/core/profile/wiki](https://github.concur.com/core/profile/wiki).

* Services SHOULD NOT store the legacy `outtask.companyId` (travel) nor the `expense.entityId` (expense) as these are internal database keys.
* Services SHOULD store the company identifier with `companyId` as the key name and the [RFC 4122 Universally Unique IDentifier (UUID)](#RFC-4122) UUID4 value as provided by the company service.

```json
{
  "companyId": "ae343a7e-e921-420f-8dec-b0b36d9Bffc1"
}
```
#### <a name="data-self-describing"></a>Self-describing Data

* Representations SHOULD be self-describing through the use of a `schema` key in the root of all documents.
* The `schema` value SHOULD be in the form of a [RFC 3986 Uniform Resource Identifier (URI)](#RFC-3986).
* Examples of schema illustrating the paradigms found in this document can be found in [schema](./schema).

```json
{
  "schema": "http://example.com/schema/com-example-base-2018-03-01"
}
```

#### <a name="data-date-time"></a>Dates and Times

> Because the Concur business involves many partnerships and resource models outside of direct influence (especially within travel systems) there are times when incoming values look like a timestamp but because of the business domain it is treated differently than a traditional date or timestamp. An example of this is a time associated with a travel itinerary which, to be fully understood, must contain additional data like an IATA code.

* Services SHOULD choose from the following formats to accept and represent dates and timestamps:
  * [RFC 3339 Date and Time on the Internet: Timestamps](#RFC-3339): `YYYY-MM-DDThh:mm:ss.nnn-hh:mmZ`
* Services MAY choose from the following formats when there is a business or partnership need to do so.
  * [ISO 8601:2004](#ISO-8601) UTC + Offset: `YYYY-MM-DDThh:mm:ss.nnn-hhmmZ`
  * [Unix Time](https://en.wikipedia.org/wiki/Unix_time) also known as `POSIX` and `Epoch` time.
  * For services which deal with travel related data:
    * Portion of timestamp + [IATA Code](http://www.iata.org/services/Pages/codes.aspx)
    * Portion of timestamp + [Time Zone Abbreviation](https://en.wikipedia.org/wiki/List_of_time_zone_abbreviations)
* Services MAY only include only the date portion when including time is irrelevant to the value.

##### Examples

```json
RFC 3339
{
  "keyName": "2016-10-18T03:56:06.798-07:00",
  "keyName": "2016-10-18T03:56:06.798Z"
}

ISO 8601
{
  "keyName": "2016-10-18T03:56:06.798-0700",
  "keyName": "2016-10-18T03:56:06.798Z"
}

Unix
{
  "keyName": 1477246370
}

IATA Code
{
  "keyName": {
    "time": "2016-10-18T03:45",
    "iataCode": "SEA"
    }
}

Time Zone Abbreviation
{
  "keyName": {
    "time": "2016-10-18T03:45",
    "timezone": "EST"
    }
}
```

#### <a name="data-currency"></a>Currency

* All currency values SHOULD follow [ISO 4217:2015 Codes for the representation of currencies](#ISO-4217).
* A currency value SHOULD be an JSON object with the following keys:
 	* `amount`
		* Stored as `string` data type due to poor handling of floating point numbers in Javascript.
		* Only valid numbers.
		* Optional single decimal point.
		* Optional negative (-) sign.
		* The value SHOULD NOT contain formatting:
			* No monetary symbols.
			* No thousands separator.
	* `currency` - The [ISO 3166](#ISO-3166) country code.
* Schema: [./schema/com-example-currencyamount-2018-03-01.schema.json](./schema/com-example-currencyamount-2018-03-01.schema.json)

```json
{
  "subtotal": {
    "amount": "-1234.56",
    "currency": "USD"
  },
  "tax": {
    "amount": "1.00",
    "currency": "CHF"
  },
  "total": {
    "amount": "100",
    "currency": "JPY"
  }
}
```

#### <a name="data-key-names"></a>Key-Value Pair Names

* Services SHOULD use [Camel Case](https://en.wikipedia.org/wiki/CamelCase) with the first letter in lower case for all key-value pair names. Example: `total`, `currencyCode` and `createDate`.

### <a name="http-status-codes"></a>HTTP Status Codes

* Services SHOULD always use the appropriate HTTP status codes as defined in [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231).

The following table summarizes the status code ranges and is a direct copy from the aforementioned RFC.

Status Code Range|Definition
-----------------|----------
1xx|The 1xx (Informational) class of status code indicates an interim response for communicating connection status or request progress prior to completing the requested action and sending a final response.
2xx|The 2xx (Successful) class of status code indicates that the client's request was successfully received, understood, and accepted.
3xx|The 3xx (Redirection) class of status code indicates that further action needs to be taken by the user agent in order to fulfill the request.
4xx|The 4xx (Client Error) class of status code indicates that the client seems to have erred.
5xx|The 5xx (Server Error) class of status code indicates that the server is aware that it has erred or is incapable of performing the requested method.

#### <a name="errors-when-4xx"></a>Errors when HTTP Status Code is 4xx

* Services SHOULD provide additional context via JSON entity document when 4xx HTTP status code is provided in the response.
* Generally speaking, most 4xx errors occur occur during a `PUT` or `POST` operation.
* There may not be a need for returning 4xx errors for a service that is primarily `GET` operations noting there are exceptions for `GET` operations, for example:
	* A `GET` should return a 4xx when there are invalid parameters in the query string.
	* A 404 Not Found should be returned when the `GET` URI is dynamic and a value is wrongly formatted, or does not exist, or the user does not have permission.
* The schema starts with an array which allows the services to expand the items within the errors without breaking the contract of the service itself. It is expected many services will only ever return a single item in the array.

##### <a name="errors-when-4xx-errors"></a>Errors Schema

Name | Type | Format | Description
-----|------|--------|------------
`errors`|`array`|[`error`](#errors-when-4xx-error)|**Required** An array of errors.

##### <a name="errors-when-4xx-error"></a>Error Schema
Name | Type | Format | Description
-----|------|--------|------------
`errorCode`|`string`|-|**Required** Machine readable code associated with the error which is static and never localized. Examples: `dateTimeMissing`, `OutOfMem` and `invalidUser`. These could also be UUID4 (`a1d7bb3bb19348b0858687acc9e303ec`), number (`123456`) or a URI (`https://example.com/errors/invaliduser`) which ideally provides additional information when dereferenced. Whatever form is chosen it's worth noting contextual strings are helpful to developers reading the code.
`errorMessage`|`string`|-|**Required** Message associated with the error.
`dataPath`|`string`|-|Relative data path.
`schemaPath`|`string`|-|Relative schema path.
`errors`|`array`|[`error`](#errors-when-4xx-error)|An array of errors. Note: this points to this schema as errors can nest.

##### Examples

**Minimum**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime"
    }
  ]
}
```

**Non-nested**

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime"
    },
    {
      "errorCode": "parseDataFailed",
      "errorMessage": "Unable to parse data."
    }
  ]
}
```

**Nested**

* Also included in this example are the optional keys.

```
HTTP/1.1 400 Bad Request
Date: Tue, 19 Jul 2016 18:23:16 GMT
```

```json
{
  "errors": [
    {
      "errorCode": "missingRequiredProp",
      "errorMessage": "Missing required property: dateTime",
      "dataPath": "/merchant/location/address",
      "schemaPath": "/allOf/0/required/0",
      "errors": [
        {
          "errorCode": "parseDataFailed",
          "errorMessage": "Unable to parse data.",
          "dataPath": "/merchant/location/address/state",
          "schemaPath": "/allOf/0/required/0/0"
        }
      ]
    }
  ]
}
```

### <a name="service-changes"></a>Service Changes

#### <a name="service-changes-breaking"></a>Breaking

* Use versioning.
* Removing key-value pairs is usually considered a breaking change.
* Changing key names is usually considered a breaking change.

#### <a name="service-changes-non-breaking"></a>Non-breaking

* Adding keys to data is generally not considered a breaking change.

### <a name="documentation"></a>Documentation

Please refer to the [API Documentation Requirements](https://github.concur.com/developer/documentation) repo for information. 


## <a name="works-cited"></a>Works Cited

### <a name="rest"></a>Representational State Transfer (REST) Dissertation by Roy Fielding, 2000

* [https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm)

Also included inline are the references to the parts of the dissertation outside of chapter 5.

* [5.1 Deriving REST](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1)
	* [5.1.1 Starting with the Null Style](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_1)
	* [5.1.2 Client-Server](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_2)
		* Reference: [3.4.1 Client-Server (CS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_1)
	* [5.1.3 Stateless](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_3)
		* Reference: [3.4.3 Client-Stateless-Server (CSS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_3)
	* [5.1.4 Cache](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_4)
		* Reference: [3.4.4 Client-Cache-Stateless-Server (C$SS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_4)
	* [5.1.5 Uniform Interface](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_5)
	* [5.1.6 Layered System](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_6)
		* Reference: [3.4.2 Layered System (LS) and Layered-Client-Server (LCS)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_4_2)
	* [5.1.7 Code-On-Demand](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_7)
		* Reference: [3.5.3 Code on Demand (COD)](https://www.ics.uci.edu/~fielding/pubs/dissertation/net_arch_styles.htm#sec_3_5_3)
	* [5.1.8 Style Derivation Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_1_8)
* [5.2 REST Architectural Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2)
	* [5.2.1 Data Elements](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1)
		* [5.2.1.1 Resources and Resource Identifiers](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_1)
		* [5.2.1.2 Representations](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_1_2)
	* [5.2.2 Connectors](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_2)
	* [5.2.3 Components](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_2_3)
* [5.3 REST Architectural Views](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3)
	* [5.3.1 Process View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_1)
	* [5.3.2 Connector View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_2)
	* [5.3.3 Data View](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_3_3)
* [5.4 Related Work](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_4)
* [5.5 Summary](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm#sec_5_5)

### <a name="hateoas"></a>Hypermedia as the Engine of Application State

* [https://en.wikipedia.org/wiki/HATEOAS](https://en.wikipedia.org/wiki/HATEOAS)

### <a name="RFC-7230-7237"></a>RFC 7230-7237 Hypertext Transfer Protocol (HTTP/1.1)

* [RFC 7230 Hypertext Transfer Protocol (HTTP/1.1): Message Syntax and Routing](https://tools.ietf.org/html/rfc7230)
* [RFC 7231 Hypertext Transfer Protocol (HTTP/1.1): Semantics and Content](https://tools.ietf.org/html/rfc7231)
	* [GET](https://tools.ietf.org/html/rfc7231#section-4.3.1)
	* [HEAD](https://tools.ietf.org/html/rfc7231#section-4.3.2)
	* [POST](https://tools.ietf.org/html/rfc7231#section-4.3.3)
	* [PUT](https://tools.ietf.org/html/rfc7231#section-4.3.4)
	* [DELETE](https://tools.ietf.org/html/rfc7231#section-4.3.5)
	* [CONNECT](https://tools.ietf.org/html/rfc7231#section-4.3.6)
	* [OPTIONS](https://tools.ietf.org/html/rfc7231#section-4.3.7)
	* [TRACE](https://tools.ietf.org/html/rfc7231#section-4.3.8)
	* [Request Header Fields](https://tools.ietf.org/html/rfc7231#section-5)
	* [Response Status Codes](https://tools.ietf.org/html/rfc7231#section-6)
	* [Response Header Fields](https://tools.ietf.org/html/rfc7231#section-7)
* [RFC 7232 Hypertext Transfer Protocol (HTTP/1.1): Conditional Requests](https://tools.ietf.org/html/rfc7232)
* [RFC 7233 Hypertext Transfer Protocol (HTTP/1.1): Range Requests](https://tools.ietf.org/html/rfc7233)
* [RFC 7234 Hypertext Transfer Protocol (HTTP/1.1): Caching](https://tools.ietf.org/html/rfc7234)
* [RFC 7235 Hypertext Transfer Protocol (HTTP/1.1): Authentication](https://tools.ietf.org/html/rfc7235)
* [RFC 7236 Initial Hypertext Transfer Protocol (HTTP) Authentication Scheme Registrations](https://tools.ietf.org/html/rfc7236)
* [RFC 7237 Initial Hypertext Transfer Protocol (HTTP) Method Registrations](https://tools.ietf.org/html/rfc7237)

### <a name="RFC-5789"></a>RFC 5789 PATCH Method for HTTP

* [https://tools.ietf.org/html/rfc5789](https://tools.ietf.org/html/rfc5789)

### <a name="RFC-5646"></a>RFC 5646 Tags for Identifying Languages

* [https://tools.ietf.org/html/rfc5646](https://tools.ietf.org/html/rfc5646)

### <a name="json"></a>Javascript Object Notation (JSON)

* [http://www.json.org](http://www.json.org)
* [http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-404.pdf)
* [https://tools.ietf.org/html/rfc7159](https://tools.ietf.org/html/rfc7159)

### <a name="json=patch"></a>JavaScript Object Notation (JSON) Patch

* [https://tools.ietf.org/html/rfc6902](https://tools.ietf.org/html/rfc6902)

### <a name="json-schema"></a>JSON Schema

* [http://json-schema.org](http://json-schema.org)

### <a name="RFC-7519"></a>RFC 7519 JSON Web Token (JWT)

* [https://tools.ietf.org/html/rfc7519](https://tools.ietf.org/html/rfc7519)

Links to claims within the RFC:

* [4.1 Registered Claim Names](https://tools.ietf.org/html/rfc7519#section-4.1)
	* [4.1.1 "iss" (Issuer) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.1)
	* [4.1.2 "sub" (Subject) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.2)
	* [4.1.3 "aud" (Audience) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.3)
	* [4.1.4 "exp" (Expiration Time) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.4)
	* [4.1.5 "nbf" (Not Before) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.5)
	* [4.1.6 "iat" (Issued At) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.6)
	* [4.1.7 "jti" (JWT ID) Claim](https://tools.ietf.org/html/rfc7519#section-4.1.7)

#### Additional Resources

* [JWT Summary (Concur)](./jwt-summary.md)
* [https://jwt.io](https://jwt.io)
* [Public Claim Names](http://www.iana.org/assignments/jwt/jwt.txt)

### <a name="RFC-3339"></a>RFC 3339 Date and Time on the Internet: Timestamps

* [https://tools.ietf.org/html/rfc3339](https://tools.ietf.org/html/rfc3339)

### <a name="RFC-5280"></a>RFC 5280 Internet X.509 Public Key Infrastructure Certificate and Certificate Revocation List (CRL) Profile

* [https://tools.ietf.org/html/rfc5280](https://tools.ietf.org/html/rfc5280)

### <a name="RFC-6749"></a>RFC 6749 OAuth 2.0 Authorization Framework

* [https://tools.ietf.org/html/rfc6749](https://tools.ietf.org/html/rfc6749)

### <a name="RFC-1034"></a>RFC 1034 Domain Names - Concepts and Facilities

* [https://tools.ietf.org/html/rfc1034](https://tools.ietf.org/html/rfc1034)

### <a name="RFC-3986"></a>RFC 3986 Uniform Resource Identifier (URI): Generic Syntax

* [https://www.ietf.org/rfc/rfc3986.txt](https://www.ietf.org/rfc/rfc3986.txt)

### <a name="RFC-4122"></a>RFC 4122 A Universally Unique IDentifier (UUID) URN Namespace

* [https://tools.ietf.org/html/rfc4122](https://tools.ietf.org/html/rfc4122)
* Services SHOULD use all lowercase with dashes preserved.
	* This RFC generally recommends a lower case when producing values (while allowing for uppercase) and requires case-insensitivity when parsing.
* For more information on UUID4 see [4.4. Algorithms for Creating a UUID from Truly Random or Pseudo-Random Numbers](https://tools.ietf.org/html/rfc4122#section-4.4).

### <a name="RFC-5988"></a>RFC 5988 Web Linking

* [https://tools.ietf.org/html/rfc5988](https://tools.ietf.org/html/rfc5988)

### <a name="RFC-6570"></a>RFC 6570 URI Template

* [https://tools.ietf.org/html/rfc6570](https://tools.ietf.org/html/rfc6570)

### <a name="IANA-headers"></a>Internet Assigned Numbers Authority (IANA) Message Headers

* [http://www.iana.org/assignments/message-headers/message-headers.xhtml](http://www.iana.org/assignments/message-headers/message-headers.xhtml)

### <a name="ISO-3166"></a>ISO 3166-1 Country Codes

* Online Browsing Platform: [https://www.iso.org/obp/ui/#search/code/](https://www.iso.org/obp/ui/#search/code/)
* [http://www.iso.org/iso/home/standards/country_codes.htm](http://www.iso.org/iso/home/standards/country_codes.htm)

### <a name="ISO-4217"></a>ISO 4217:2015 Codes for the representation of currencies

* [ISO 4217:2015 Codes for the representation of currencies](https://github.concur.com/developer/api/tree/main/standards/ISO_4217_2015_en.PDF)

### <a name="ISO-8601"></a>ISO 8601:2004 Data elements and interchange formats -- Information interchange -- Representation of dates and times

* [https://www.iso.org/standard/40874.html](https://www.iso.org/standard/40874.html)

### ECMAScript® 2017 Language Specification (ECMA-262, 8th edition, June 2017)

* [https://www.ecma-international.org/ecma-262/8.0/index.html](https://www.ecma-international.org/ecma-262/8.0/index.html)

### <a name="RFC-2119"></a>RFC 2119 Key words for use in RFCs to Indicate Requirement Levels

* [https://www.ietf.org/rfc/rfc2119.txt](https://www.ietf.org/rfc/rfc2119.txt)
* All requirements are noted with uppercase (MUST, SHOULD) in bulleted lists to disambiguate from narrative prose using lower case (should, must).

The following table has been added for ease of understanding and is a direct copy of the definitions contained in RFC 2119.

Keyword|Synonyms|Definition
-------|--------|----------
MUST|Required, Shall|Absolute requirement.
MUST NOT|Shall Not|Absolute prohibition.
SHOULD|Recommended|There may exist valid reasons in particular circumstances to **ignore a particular item**, but the full implications must be understood and carefully weighed before choosing a different course.
SHOULD NOT|Not Recommended|There may exist valid reasons in particular circumstances when the particular **behavior is acceptable** or even useful, but the full implications should be understood and the case carefully weighed before implementing any behavior described with this label.
MAY|Optional|An item is truly optional
