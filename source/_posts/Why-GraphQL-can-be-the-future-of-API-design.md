title: Why GraphQL can be the future of API design
date: 2015-09-06 20:38:06
author: JC
tags:
- graphql
category:
- Backend
- English
---
### Introduction

>GraphQL is a data querying language designed to describe the complex, nested data dependencies of modern applications.

This is the official definition of GraphQL from the GraphQL team. GraphQL has nothing to do with SQL or non-SQL. It’s just an application level data fetching protocol, which, in my mind, will change the way developers write APIs.

<!-- more -->

### How does it look like?

You can get a very distinct feeling for what GraphQL is after you see this demo.

In GraphQL, suppose the client wants to know the `name,gender,age` of the `User` with `id=1` . A valid GraphQL query looks like this:

```json
{
  user(id: 1){
    name
    gender
    age
  }
}
```

And a GraphQL server, with a correct type definition, will return something like:

```json
{
  "user":{
    "name": "JC"
    "gender": "male"
    "age": 25
  }
}
```

The response has the same **shape** as the request. This is one of the design principles of GraphQL. The query is totally specified by client, and the server responds exactly what a client asks. Also the query itself is a hierarchical set of fields, which  naturally describes the data requirements.

### Why using GraphQL?

In Strikingly, like most other modern startups, we adopt RESTful API design in most of our work. RESTful design is neat and easy to implement and can be easily understood. We enjoyed it a lot, but it also has two main problems:

#### Version hell

{% asset_img version_hell.png [] %}
API versioning increases duplicated code and complexity in the codebase. And with multiple clients, although it’s not purely RESTful, we do have some custom endpoints for specific usages, which can also be annoying when maintaining the codebase.

#### Round-trips

{% asset_img round_trip.png [] %}
In a typical client use case, for example, an UI of the user's profile picture and his recent blog post requires at least two requests: one for `User` and one for `BlogPost`. The 2 requests have dependency over each other. Client first needs to get the blog post, and then get the user id from the blog post. This causes additional round-trips. We can combine the two requests using custom endpoints, but it's not RESTful like and it's hard to maintain.

#### Data over-fetching

In some use cases, the responses from backend are beyond enough than what clients really need. This will cause a data over-fetching problem, which in general should be considered as a performance issue.

#### Why GraphQL can be a game changer

In GraphQL, based on its design principles, all queries are customized by clients. So there’s no need for endpoints with multiple versions any more. Different versions of different clients make separate GraphQL queries to the same endpoint and get the desired shaped responses accordingly.

{% asset_img good_version.png [] %}
Thus, as long as the type definition on the GraphQL server is backward compatible - which is another design principle in GraphQL - all the old versioned clients can keep working smoothly, and the codebase of the backend server will no longer be polluted by those versions.

Since all GraphQL queries are specified on clients, queries will only ask for the data which clients need under a certain context. This means that all the queries are specific under certain contexts, and the data can be nested together into one query. This simply solves the round-trips and data over-fetching problems. A possible GraphQL query for the above example can be:

```json
query compactBlogPosts{
  blog(id: 123){
    posts.first{
      author{
        picture
      }
      title
      date
      blurb
    }
  }
}
```



### Future

GraphQL will also change our daily develop cycle. Because the frontend no longer needs to cooperate with backend about the API data, instead, they just need to read the latest type definition of the application and request the exact data they want under different client contexts. This can be illustrated by the image below. The type definition on the backend actually defines all the possibilities of all the available queries and fields, which can also be queried by clients because the introspectiveness of GraphQL. Client builds up the queries, which can also considered as data requirements, and send them to the backend to get the expected data.

{% asset_img big_picture.png [] %}
Another reason that I think GraphQL is great is the positioning of the project. The GraphQL team from Facebook announced GraphQL as a spec of a data-fetching framework. This means GraphQL has no technical restriction at all and can be implemented in different languages and frameworks. The team just released the implementation in JS recently. The whole GraphQL ecosystem can be divided into two directions:

{% asset_img ecosystem.png [] %}
The client and server both need GraphQL implementations, which needs a lot of dev cost. This is also why the GraphQL team released as many things as they can at this early stage, they want to build up a proactive community quickly and call for more people to contribute to the ecosystem.

### Reference

[Data fetching for React applications at Facebook](https://www.youtube.com/watch?v=9sc8Pyc51uU)
[Lee Byron - Exploring GraphQL](https://www.youtube.com/watch?v=WQLzZf34FJ8)
[Nick Schrock & Dan Schafer - Creating a GraphQL Server](https://www.youtube.com/watch?v=gY48GW87Feo)
[GraphQL specification][graphqlspec]
[JS implementation of GraphQL][jsproject]
[graphqlspec]: https://facebook.github.io/graphql/
[jsproject]: https://github.com/graphql/graphql-js


