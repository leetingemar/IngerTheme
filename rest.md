# REST API Standards
The main purpose of building a RESTful API is to allow the users of your API to take advantage of the services it provides in a secure, consistent, efficient, and intuitive manner. This guideline describes a set of rules and standards that follows the [RESTful](https://www.ics.uci.edu/~fielding/pubs/dissertation/rest_arch_style.htm) architectural style.

## Table of Contents

  1. [Use nouns not verbs!](#use-nouns-not-verbs)
  1. [Use snake_case for request objects, response objects and query string keys](#use-snake_case-for-request-objects-response-objects-and-query-string-keys)
  1. [Http verbs](#http-verbs)
  1. [Relationships](#relationships)
  1. [Nested resources](#nested-resources)
  1. [Filtering](#filtering)
  1. [Sorting](#sorting)
  1. [Searching](#searching)
  1. [Aliases](#aliases)
  1. [Fields](#fields)
  1. [Pagination](#pagination)
  1. [HTTP status codes](#http-status-codes)
  1. [Errors](#errors)
  1. [Versioning](#versioning)
  1. [Compatibility](#compatibility)
  1. [Additional Tips](#additional-tips)
  1. [References](#references)

------------------------------

## Use nouns not verbs!

You SHALL_NOT use verbs to describe an endpoint in your API. The next endpoints are bad practices that you MUST avoid.

    /addOrder
    /updateOrder/:id
    /getAllOrders
    /getOrder/:id

The problem with the use of verbs is that each minor change or feature in the API will need a new endpoint, to prevent this problems you MUST use nouns like this:

    /orders
    /orders/:id

The collection's name in the endpoint MUST be pluralized. The main reason is because when we refer to a collection, this collection is a set of items that we can filter using the ID like this `/orders/1`.

    BAD
    /order
    /order/2

    GOOD
    /orders
    /orders/2

According to [RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec3.html#sec3.2.3) URLs are “case sensitive” (except for the scheme and the host). If you need to use more than one word in your resource you MUST use hyphens “-” to separate words.

    BAD
    /userProfiles
    /user_profiles

    GOOD
    /user-profiles
    /user-profiles/2

All the actions over the endpoint are defined by the HTTP Verbs.

## Use snake_case for request objects, response objects and query string keys

All properties in a JSON request or response MUST be named using `snake_case` naming convention.

```
// BAD
{
    "companyName": "Your Company",
    "updatedAt": "2017/07/20"
}

// GOOD
{
    "company_name": "Your Company",
    "updated_at": "2017/07/20"
}
```

Also, the query string keys MUST be named using `snake_case` naming convention.

```
// BAD
?activecontract=true&contracttype=Trading

// BAD
?activeContract=true&contractType=Trading

// GOOD
?active_contract=true&contract_type=Trading
```

## HTTP verbs

The HTTP Verbs allow us to add actions to our resources. Each verb represents a specific action.

 - **GET**: Retrieves a collection or a specific resource by id. Also, it returns the references to linked (associated) resources.
 - **POST**: Creates a new item in the collection and returns the newly created resource.
 - **PUT**: Replaces the entire resource with a new one in the same path and returns the updated resource. Note: For partial update you should use the PATCH method.
 - **PATCH**: Partially updates the resource and returns the entire updated resource. This method MUST follow the [RFC 6902 - JavaScript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902).
 - **DELETE**: Deletes the resource from the collection. This method should not return a body. Rather, it should return a [204 NotContent](https://httpstatuses.com/204) status code.

For example, the next request performs the described actions using the HTTP Verbs

    GET     /orders                 Retrieves a list of orders.
    GET     /orders?state=active    Retrieves a list of orders where the state is equals to "active".
    GET     /orders/12              Retrieves the order with the ID 12.
    POST    /orders                 Creates a new order.
    PUT     /orders/12              Replaces the order object with the ID 12.
    PATCH   /orders/12              Updates partially the order with the ID 12.
    DELETE  /orders/12              Deletes the order with the ID 12.


## Relationships

The relationships can only exist within another resource, for example a Team Player can only exist if a Team exists, in this case we can use a relationship between Player and Team.

    GET     /teams/3/players    Retrieves all the Players of the Team with ID 3.
    GET     /teams/3/players/12 Retrieves the Player with the ID 12 inside the Team with the Id 3.
    POST    /teams/54/players   Creates a new Player in the Team 54.

You MUST avoid creating relations between resources that are not related and also you MUST avoid combining resources with actions, since that may confuse the API's users.

## Nested resources

In data models with nested parent/child resource relationships, paths could become deeply nested if you’re not careful. Related collections SHOULD be nested at most one level deep.

    BAD
    GET /users/:id/applications/:id/user-profiles/:id/scopes

    GOOD
    GET /users/:id/applications
    GET /users/:id/applications/:id
    GET /applications/:id/user-profiles
    GET /applications/:id/user-profiles/:id
    GET /user-profiles/:id/scopes
    GET /user-profiles/:id/scopes/:id


## Filtering

You MUST use the "?" character to filter the resource requests.

    BAD
    GET /orders/stateSearch/open

    GOOD
    GET /orders?state=open   Retrieves the orders with an "open" state.

## Sorting

You MUST use the "?sort=" keyword to sort the resource requests.
You also can use the "-" character to invert the order of the sorting.

    GET /orders?sort=-created_at     Retrieves a list of orders sorted by the creation date in a descending way.
    GET /orders?sort=updated_at      Retrieves a list of orders sorted by the updated date in an ascending way.

## Searching

If you need to make complex search using a query you also need to use the "?q=" character like this:

    GET /orders?q=AMX                               Retrieves all orders that contain the word AMX in any fields.
    GET /orders?q=AMX&q_fields=name,description     Retrieves all orders that contain the word AMX in the fields name and description.

## Aliases

Aliases are OPTIONAL, though not encouraged for consistency purposes. You can create aliases for common queries like this:

    GET /orders/recently-closed      Get the recently closed orders.

## Fields

In the case where a resources contains many properties, we SHOULD allow for the API user to determine which properties (or fields) the endpoint will return. This is accomplished by implementing a filtering technique using the "?fields=" filter.

    GET /orders?fields=name,created_at   Retrieves a list of orders only with the name and createdAt fields.

    Sample response
    [
        {
            name: "order name",
            created_at: "2017/12/28"
        },
        {
            name: "other order name",
            created_at: "2017/10/12"
        }
    ]

## Pagination

RESTful APIs that return collections SHOULD return partial sets to avoid a large response payload. Users of these services must expect partial result sets and correctly page through a pagination set to retrieve the entire set.

An important thing is that sorting and filtering parameters must be consistent across pages, because both client and server-side pagination are fully compatible with both filtering and sorting.

The paginated resources must return a pagination metadata in the response, for example.

    Example request:
    ?page=1,page_size=15

    Example response
    {
        "items":[
            { "id": "Item 1","price": 99.95,"sizes": null},
            { ... },
            { ... },
            ....
        ],
        "pagination_metadata": {
            total_items: 30,
            page_size: 15,
            page_count: 15,
            page: 1,
            next: "?page=2,page_size=15",
            previous: ""
        }
    }
    
  - `page_size`: Value sent by the client in the query_params. It defines the limit of results per page. **Please note** that your endpoint MUST have a default page size to mitigate large payloads and unnecessary stress on databases.
  - `total_items`: Value that represents the count of all results that match the query excluding pagination.
  - `page_count`: Value that represents the count of results in the current page.
    
If the original request contains other values in the query string then the next and previous pagination metadata MUST contain all other values of the query string, like this:

     next: "?page=3,page_size=10&sort=-created_at",
     previous: "?page=1,page_size=10&sort=-created_at"

## HTTP Status codes

HTTP describes a set of [status codes](https://tools.ietf.org/html/rfc7231#section-6) that can be returned from your API. The next list of status codes SHOULD be used to describe the response status to the users of your API.

 - `200 OK` Successful response on GET, PUT, PATCH or DELETE actions.
 - `201 Created` Successful response on a POST action.
 - `204 Not content` Successful response that won't be returning a body, commonly used in DELETE requests.
 - `400 Bad request` The request will not be processed due to something that is perceived to be a client error (e.g., malformed request syntax, invalid request message framing, or deceptive request routing).
 - `401 Unauthorized` Authentication details are not provided or are invalid.
 - `403 Forbidden` Authentication succeeded but authenticated user is not allowed to access the resource with the provided action.
 - `404 Not found` The requested resource was not found.
 - `409 Conflict` The request could not be completed due to a conflict with the current state of the target resource.
 - `415 Unsupported media type` Incorrect content type was provided as part of the request.
 - `429 Too many requests` The request is rejected due to rate limiting.

## Errors

For failed requests the developers should be able to handle consistent error objects. It is RECOMMENDED to use the following properties for error objects.

 - `event_id`: Unique identifier value of the request that allow reference to event chain.
 - `code`: Server custom error code.
 - `message`: A human-readable representation of the error.
 - `target`: Target request endpoint where the error ocurred.
 - `details`: An error array with each detail of the error ocurred.

For example:


    {
        "error": {
            "event_id": "c44fcaab-e663-4750-9ede-169e8f934e5c",
            "code": "BadArgument",
            "message": "Previous passwords may not be reused",
            "target": "POST|https://auth.company.com/v1/users",
            "details": [
                {
                    "code": "InvalidValue",
                    "message": "Previous password may not be reused.",
                    "target": "password"
                }
            ]
        }
    }

## Versioning

The services MUST be versioned using the `Major` scheme. The version will be embedded in the path of the request url.

    https://api.myapp.com/v1/orders

Some of the advantages of versioning the API through the URL are:

 - A URI-versioned API is less error prone and more familiar to the client developers. If you decided to use API versioning through the Header and the client developer forgets to include a resource version in the header, you have to decide if they should be directed to the latest version (which can cause errors when incrementing the version).
 -  URI versioning lends itself to hosing multiple versions in the same application.

Services MUST increment their version number in response to any breaking API change.

Some clear examples of breaking changes are:

 - Removing or renaming APIs or API parameters.
 - Changes in behavior for an existing API endpoint.
 - Changes in Error Codes and Fault Contracts.
 - Anything that would violate the [Principle of Least Astonishment](https://softwareengineering.stackexchange.com/questions/187457/what-is-the-principle-of-least-astonishment).

## Compatibility

For changes that don't require a version increment, they MUST NOT violate the current Contract between Service Providers (the API) and Service Consumers. Therefore, when maintaining an existing API, all changes MUST be backward compatible. By being backward compatible when making contract changes, we help the service to be more stable and robust. This is important because Consumers have their own and independent release life-cycles, which SHALL NOT be impacted by Service Providers.

Should the current contract require a change (with no version increment), these steps MUST be followed:
1. Provider only supports the Old Use Case
1. Provider will support both Old and New Use Cases
1. Provider will notify Clients about the changes
1. Clients will be given a time frame to adhere to the New Use Case
1. Provider will monitor the Old Use Case usage by Clients
1. Provider will only support the New Use Case

## Additional Tips

 - Cache the response, when possible, in order to make your API faster.
 - Validate all requests to prevent error in the request, if an error is returned, the message MUST be explicit enough to ensure that the user knows what to do to correct it.
 - Protect your API against [CSRF](https://owasp.org/www-community/attacks/csrf).
 - Limit the requests by implementing a throttling mechanism.
 - Complement you API with an SDK.
 - Document your API with an [OpenAPI Swagger](https://swagger.io/solutions/api-documentation/) document.

## References

 - [RFC 7231](http://www.ietf.org/rfc/rfc7231.txt)
 - [Microsoft REST API Guidelines](https://github.com/Microsoft/api-guidelines/blob/vNext/Guidelines.md#51-errors)
 - [Best Practices for Designing a Pragmatic RESTful API](http://www.vinaysahni.com/best-practices-for-a-pragmatic-restful-api)
 - [Http Status Codes](https://en.wikipedia.org/wiki/List_of_HTTP_status_codes)
 - [Restful-api-guidelines](https://byrondover.github.io/post/restful-api-guidelines)
 - [JavaScript Object Notation (JSON) Patch](https://tools.ietf.org/html/rfc6902)
 - [What is JSON Patch?](http://jsonpatch.com/)
