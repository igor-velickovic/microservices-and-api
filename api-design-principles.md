# API Development principles

## KISS - Keep it simple, stupid

Main objective of API is to reach out to as many developers as possible, opening our systemto the Internet. It is therefore critical that the API be self-describing and as simple as possible, so that developers barely need to refer to the documentation. We refer to this as affordance: the API suggests its own usage.

When designing an API, the following principles should be kept in mind:

- The API semantics must be intuitive. URI, payload, request or response: a developer should be able to use them without referring to the API documentation.
- The terms must be common and concrete, rather than emanate from a functional or technical jargon. Accounts, orders, addresses, assets are all good examples.
- There should not be different ways to achieve the same action.
- The API is designed for its clients, the developers, and should not be a simple access layer above the domain model. The API must provide simple features that fit developers requirements.
- In the early design phase, focus on the main use-cases and leave exceptional ones
  for later phases.

### cURL examples

Always illustrate your API call documentation by cURL examples. Readers can simply cut-and-paste them, and they remove any ambiguity regarding call details.

```curl
CURL -X POST \
-H "Accept: application/json" \
-d '{"state":"running"}' \
https://api.conda.online/v1/accounts/007/orders
```

### Average granularity

It's important to keep a reasonable limit. Do not make unnecessary API calls especially when the information are commonly used together.

```curl
CURL https://api.conda.online/v1/accounts/1234
200 OK
{"id":"1234", "name":"Han Solo", "address":"https://api.conda.online/v1/addresses/4567"}
```

```curl
CURL https://api.conda.online/addresses/4567
200 OK
{"id":"4567", "street":"sunset bd", "country": "http://api.conda.online/v1/countries/98"}
```

```curl
CURL https://api.conda.online/v1/countries/98
200 OK
{"id":"98", "name":"France"}
```

This would be a bad example. Instead:

- Group only resources that are almost always accessed together
- Do not embed collections having many components. For example, a list of current orders is limited (it's unlikely to have more than 2 or 3 orders at the same time) but a list of past orders can be much longer.
- Have at most 2 levels of nested objects (e.g. `/v1/accounts/addresses/countries`)

### Security

Always use oAuth2 security protocol to secure the API (http://tools.ietf.org/html/rfc6749)

### URIs

To describe your resources, always use concrete plural names and not action verbs.

Do not use:

- getAccounts
- getOrders
- createOrder
- updateAccount

Use RESTful approach:

- GET /clients/1
- POST /clients
- PATCH /accounts/1
- PUT /orders/1
- DELETE /addresses/1

### Case consistency

For URI's case always use 'spinal-case' (https://tools.ietf.org/html/rfc3986#section-3.3)

`POST /v1/specific-orders`

For body case always use 'lowerCamelCase'

`POST /orders {"accountID":"007"}`

### Versioning

Always include a compulsory one digit version at the highest level of the URI's path. The version number refers to a major release of the API as to a resource. We will support at most 3 versions at the same time.

`POST /v1/accounts/007/orders`

### I18N

Use ISO 8601 standard for Date/Time/Timestamp:

`1978-05-10T06:06:06+00:00` or `1978-05-10`

Add support for different Languages:

`Accept-Language: fr-CA, fr-FR not ?language=fr`

### CRUD

Systematically use HTTP verbs (GET, POST, PUT, DELETE) to describe what actions are performed on the resources. We will never use PATCH verb, to do a partial update of resource. For partial answers (STATUS CODE 200) we use 'fields' query string

`GET /accounts/007?fields=firstName,lastName,address`

### NON RESOURCE SCENARIOS

In a few use cases we have to consider operations or services rather than resources. You may use a POST request with a verb at the end of the URI.

`POST /emails/42/send`

### Query strings

####Paging

We use `offset` and `limit` query string to define paged results. It is important to include
resource count and previous/next href to API response. Paging will be impacted by sorting and filtering. The combination of these 3 parameters should be usable with consistency in the requests to API.

`GET /accounts?offset=0&limit=25`

By default, if paging query strings are not provided, always respond will all resources.

#### Sorting

By default, always sort resources in ascending order. We use `+` and `-` in front of attributes.

`GET /accounts?sort=-firstName` (sort in descending order)

#### Filtering

Always use attribute's name with an operator and the expected values, each of them separated by a comma. We need to use `=`, `<=`, `>=` as operators

`GET /accounts?lastName=Solo`
`GET /orders?amount<=500`

#### Fields

We need to be able to select the attributes, separated by comma, to be retrieved, over 1 evel of resource.

`GET /accounts/007?fields=firstName,lastName,address`

#### Searching

Parameters are provided the same way as for a filter, through the query-string, but they are not necessarily exact values, and their syntax permits approximate matching.

Being itself a resource, the search must support paging like all the other resources of API.

Resource search is a sub-resource of our collection. As such, its results will have a different format than the resources and the collection itself.

`GET /accounts/search?firstName=Han`

### URL RESERVED WORDS: OFFSET, LIMIT, SORT, SEARCH, FIRST, LAST, COUNT, HISTORY

Use /first to get the 1st element

```curl
GET /orders/first
200 OK
{"id":"1234", "state":"paid"}
```

Use /last to retrieve the latest resource of a collection

```curl
GET /orders/last
200 OK
{"id":"5678", "state":"running"}
```

Use /count to get the current size of a collection

```curl
GET /orders/count
200 OK
{"2"}
```

Use /history to return history of a resource (data revision with all the changes)

```curl
GET /orders/:orderID/history
200 OK
{"current":{"id":"5678", "state":"running"},"history":{...}}
```

### Content negotiation

By default, the API will share resources in the JSON format.

### Cross-domain

Implement CORS protocol by adding instructions to NodeJS HTTP server ()

### HATEOAS

Your API should provide Hypermedia links in order to be completely discoverable. Each call to the API should return in the Link header every possible state of the application from the current state, plus self.

Implement HATEOAS, using the following method, compliant with RFC5988

```curl
GET /customers/007
200 Ok
{ "id":"007", "firstName":"Han",...}
Link : <https://api.conda.online/v1/customers>; rel="self"; method:"GET",
<https://api.conda.online/v1/addresses/42>; rel="addresses"; method:"GET",
<https://api.conda.online/v1/orders/1234>; rel="orders"; method:"GET"
```

Error Structure
We use the following JSON structure:

```curl
{
"error": "short_description",
"error_code": 1233,
"error_uri": "URI to a detailed error description"
}
```

### Status codes

#### SUCCESS

- 200 - Success with content
- 201 - Created
- 202 - Accepted (to be processed later for async calls)
- 204 - Success without content
- 206 - Partial content (as response to requests with 'fields' query strings)

### CLIENT ERROR

- 400 - Bad Request
- 401 - Unauthorized
- 403 - Forbidden
- 404 - Not Found
- 405 - Method not allowed (Calling a method on this resource has no meaning, or the user is not authorized to make this call.)
- 406 - Not Acceptable (Nothing matches the Accept-\* Header of the request)

### SERVER ERROR

- 500 - Server error
