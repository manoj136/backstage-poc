# Architecture Overview !

## 1. Introduction

Representational State Transfer (REST) APIs in our case are exposed via HTTP protocol. Thus, one needs to be sure that follow a common convention when defining *resource routes*. After all, REST API design goes around [*Convention over Convention*](https://github.com/MapsterMapper/Mapster) paradigm, and this means that *resource routes* must follow a conventional and predictable schema in any area. This ensures a predictable consumption and RESTful API implementors require less documentation to explain the design.

This document addresses the need to define a predictable resource naming convention.

## 2. Basic routes: resource collection and resource

Glossary:

| Route kind | Convention | Example |
| - | - | - |
| Resource collection | Resource route must be the resource in plural. | `/articles` |
| Resource | Resource route must follow up the collection plus the resource identifier. | `/articles/65d1e6c5-2a05-4d8a-8532-29b62e79f99d` |

## 3. Nested resource routes

Nested resource routes must be meaningfull.

### 3.1 Common rule

The common rule is nested resources are accessed via collection or one resource routes, and they must provide just the most nested resource **starting from the aggregate root** based on domain-driven design bounded context. Also, this applies to retrieving a resource by its identifier.

Example 1: 
- ❌ Wrong: `/contracts/applications/catalogues/articles` (aggregate roots mustn't be abused: API roots should be segregated by, at least, bounded contexts in terms of domain-driven design)
- ✅ OK: `/catalogues/articles` and `/contracts/applications`


Example 2:

- ❌ Wrong: `/contracts/65d1e6c5-2a05-4d8a-8532-29b62e79f99d/apps/f9fc61ec-172e-4314-938f-19abfda7049c/sites/2cb72e4e-1514-4fa1-9b4d-26849453d4ef`
- ✅ OK: `/contracts/apps/sites/2cb72e4e-1514-4fa1-9b4d-26849453d4ef`


### 3.2 Access to nested collection

The common rule is `/collection/[resource id]/nestedcollection`.

But when one needs to expose an item of the whole collection, the *Nested resource routes' common rule is applicable*. That is, once one has retrieved `nestedcollection`, the API must expose a resource route as follows: `/collection/nestedcollection/[nested collection resource id]`.

## 4. Resource access kinds

There're *resource access kinds*. Check the following table to get in touch:

| Resource access kind | HTTP Verb | Explanation |
| - | - | - |
| **Read collection** | GET | When one accesses a resource collection. |
| **Read resource** | GET | When one accesses a resource. |
| **Resource header** | HEAD | When both read collection and read resource request just resource metadata. |
| **Create resource** | POST | When one adds a resource to a given collection. |
| **Patch resource** | PATCH | When one edits a part of an existing resource in some given collection. |
| **Put resource** | PUT | When one replaces an existing resource in some given collection. |
| **Delete resource**  | DELETE | When one deletes a resource in some given collection. |

**Resource route conventions are applicable to all resource access kinds**. What changes the resource access behavior is the *HTTP VERB*.

For example:
- `GET` /articles gets an entire collection, while `DELETE /articles` drops the entire collection.
- `PATCH` `/articles/1` with `{ title: "hello world" }` updates the article's title property. The same request using `PUT` verb would replace the article and now it would only own a title, and the other properties would've been erased.

## 5. Resource access parametrization

### 5.1 Filtering, ordering and pagination
Resource routes provide direct access to the resources, but sometimes resources should be filtered, ordered, and/or paginated.

RESTful APIs must use [Sieve (.NET)](https://github.com/Biarity/Sieve) library conventions, and [Sieve](https://github.com/Biarity/Sieve) provides the API consumption specification in terms of filtering, sorting and pagination.

Taken from its [GitHub repository](https://github.com/Biarity/Sieve#send-a-request):

```
?sorts=     LikeCount,CommentCount,-created         // sort by likes, then comments, then descendingly by date created 
&filters=   LikeCount>10, Title@=awesome title,     // filter to posts with more than 10 likes, and a title that contains the phrase "awesome title"
&page=      1                                       // get the first page...
&pageSize=  10                                      // ...which contains 10 posts
```

## 6. Resource views

A given resource may provide one or many views. The default view is mandatory and resources must be exposed using the default view if no other is present (i.e. there's a single view).

Views are consumed by `view` query string parameter. View names must be case insensitive and words must be separated by dashes. For example:

- `/catalogues/887caf95-03c0-4a0e-b66a-8ac84d9fa044/articles?view=most-viewed`.
- `/catalogues/887caf95-03c0-4a0e-b66a-8ac84d9fa044/articles?view=popular`.

## 6. Resource rankings and stats

One may find that require to expose rankings and stats via RESTful API. This must be addressed in the following way:

For example:

- `/articles` is the regular resource route.
- `/stats/articles` is the article stats route.
- `/articles?view=most-viewed`

More in depth conventions in this area will evolve as in-development projects evolve too.
