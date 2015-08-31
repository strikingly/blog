title: Optimizing HTTP Cache in Rails
date: 2015-08-31 14:46:21
author: danielglh
tags:
- rails
- http cache
categories:
- Backend
- English

---

This is the first blog post of a series I'd like to write about caching and how caching improves the performance of Strikingly.

# Introduction to HTTP Cache

The basic idea of caching is based on the principle of locality, a phenomenon where the same data is accessed frequently within a relatively short time, which means we can store this data in media with higher access speed to significantly boost system performance.

Caching is one of the most important concepts in the evolution of computer/network architecture. Your computer's CPU can be faster than others' because it has a larger cache; it becomes even faster with a larger RAM, which serves as the cache of your hard disk. A hybrid hard disk has faster read times than a traditional hard disk because it has a small chunk of SSD serving as disk cache. When you visit [Strikingly](https://www.strikingly.com/), the IP address of our server is cached in numerous DNS servers around the world so you can quickly get the address from the nearest one. When you use Strikingly and navigate from one page to another, your browser properly caches certain page content so that it doesn't end up requesting the same content again and again if it hasn't changed at all.

The browser cache is just one typical type of HTTP cache (or web cache). An HTTP cache temporarily stores web documents/data in order to reduce bandwidth usage, lag, and server load. Besides the browser cache, some routers, proxies, and network gateways have built-in caching mechanisms as well.

# Getting Started

HTTP cache behavior is well defined in the HTTP specification. It's rather boring to list out the specification without any context. Since we use Rails, let's start with [a sample Rails project](https://github.com/danielglh/http-cache-example).

You can start the sample Rails server by running the following commands:

```
git clone https://github.com/danielglh/http_cache_example.git
cd http_cache_example
gem install bundler
bundle install
rake db:migrate db:seed
rails s -p 8000
```

Open [http://localhost:8000/](http://localhost:9000/) and you'll see a list of students.

# Cache Control

Now open your browser's developer tools to the “Network” tab, refresh the page, and inspect the corresponding response of the “students” document. We should see the following two response headers.

```
Cache-Control: max-age=0, private, must-revalidate
```

The values of Cache-Control here are the default cache control settings provided by Rails.

```
Cache-Control: max-age=<s>
```

Here, `max-age` is the number of seconds after the cache receives a document for which the document is still considered fresh. If `max-age` is 0, it means the cache will still keep a cached copy of the document, but it **immediately** becomes stale.

```
Cache-Control: private|public
```

If the origin server attaches the `private` header, it means the document is intended for a single user and **must not** be cached by any shared cache, while if `public` is attached, the shared cache **may** choose to cache the document (it can still choose to not cache the document).

```
Cache-Control: must-revalidate
```

If the origin server attaches `must-revalidate` header, the cache **must not** serve a stale cached copy without revalidating it with the origin server. If this header is not attached, the cache **may** choose to keep serving the stale cached copy. However, most browsers will revalidate with the origin server when the cached copy becomes stale.

As we can see, the default Rails settings provide the best practice for us to start with if we are not familiar with cache controlling. However, with the default settings, the server is not getting any benefit from HTTP cache at all. Try visiting http://localhost:8000/students repeatedly (opening new tabs with the same URL). You'll find that the server needs to fetch the data and render the view every time the browser issues a request:

```
Started GET "/students" for 127.0.0.1 at 2015-08-27 22:00:15 +0800
Processing by StudentsController#index as HTML
  Student Load (0.2ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (2.4ms)
Completed 200 OK in 49ms (Views: 48.2ms | ActiveRecord: 0.2ms)


Started GET "/students" for 127.0.0.1 at 2015-08-27 22:00:17 +0800
Processing by StudentsController#index as HTML
  Student Load (0.3ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (2.6ms)
Completed 200 OK in 49ms (Views: 48.3ms | ActiveRecord: 0.3ms)


Started GET "/students" for 127.0.0.1 at 2015-08-27 22:00:18 +0800
Processing by StudentsController#index as HTML
  Student Load (0.5ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (3.4ms)
Completed 200 OK in 65ms (Views: 63.2ms | ActiveRecord: 0.5ms)
```

Notice that the majority of request processing time is spent on view rendering.

Now let’s make some changes to benefit from browser caching by uncommenting the following code in `students_controller.rb`.

```ruby
##################################################################
# Code Snippet 1
# Uncomment to change Cache-Control header to:
# Cache-Control: max-age=86400, public, must_revalidate
##################################################################
expires_in 1.day, public: true, must_revalidate: true
```

With this change, the origin server tells the cache that the document is fresh for one day's time. Now if we visit the URL repeatedly, the request doesn’t hit the server at all — it’s served directly from the browser cache.

Notice I chose to set the cache control to public. I imagine this system would be used in a school intranet, where it's OK to make it public to a shared cache (if any).

Like I said, most browsers’ cache will revalidate stale documents with the origin server even if there’s no `must_revalidate` attached. However it’s still a good practice to explicitly attach it to make sure **all** HTTP caches follow the revalidation rule and avoid serving stale content.

# Revalidation

This begs the question: What exactly is revalidation? To understand what it is and how it works, we need to shorten the max-age time first by commenting code snippet 1 and uncommenting code snippet 2:

```ruby
##################################################################
# Code Snippet 2
# Uncomment to change Cache-Control header to:
# Cache-Control: max-age=10, public, must_revalidate
##################################################################
expires_in 10.seconds, public: true, must_revalidate: true
```

Now we can make repeated requests for 10 seconds and it won’t hit the server. After 10 seconds, it will hit the server again and get a usual 200 response, then it’s cached for another 10 seconds, and so on. In this case, revalidation is simply fetching another fresh copy from origin server.

```
Started GET "/students" for 127.0.0.1 at 2015-08-27 22:10:39 +0800
Processing by StudentsController#index as HTML
  Student Load (0.5ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (2.1ms)
Completed 200 OK in 59ms (Views: 58.2ms | ActiveRecord: 0.5ms)


Started GET "/students" for 127.0.0.1 at 2015-08-27 22:10:51 +0800
Processing by StudentsController#index as HTML
  Student Load (0.3ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (3.5ms)
Completed 200 OK in 49ms (Views: 48.3ms | ActiveRecord: 0.3ms)
```

However, during the 10 seconds, we're not changing any student data. It will be nice if the origin server can tell the browser cache this fact so the browser cache can just serve a cached copy to the user.

Fortunately this can be done via conditional GET. A conditional GET request is a GET request that carries certain conditional headers. The origin server returns a fresh copy only if the conditions are true (which means the cached copy is stale), or returns a message to the cache if the cached copy is still fresh and can be served to the client.

To enable conditional GET support on the server side, you can uncomment code snippet 5:

```ruby
##################################################################
# Code Snippet 5
# Uncomment the following code to specify revalidation rule
##################################################################
fresh_when(etag: @students, last_modified: @students.first.updated_at)
```

Again, we can repeat the request for 10 seconds and it won’t hit the server at all. After 10 seconds, it will hit the server and get a 304 response:

```
Started GET "/students" for 127.0.0.1 at 2015-08-27 22:15:41 +0800
Processing by StudentsController#index as HTML
  Student Load (0.2ms)  SELECT  "students".* FROM "students"  ORDER BY "students"."updated_at" DESC LIMIT 1
  Cache digest for app/views/students/index.html.erb: 146e6d9cd79eda46b0b42ba6c3ca01d4
  Student Load (0.2ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
  Rendered students/index.html.erb within layouts/application (1.2ms)
Completed 200 OK in 45ms (Views: 41.1ms | ActiveRecord: 0.4ms)


Started GET "/students" for 127.0.0.1 at 2015-08-27 22:15:54 +0800
Processing by StudentsController#index as HTML
  Student Load (0.6ms)  SELECT  "students".* FROM "students"  ORDER BY "students"."updated_at" DESC LIMIT 1
  Cache digest for app/views/students/index.html.erb: 146e6d9cd79eda46b0b42ba6c3ca01d4
  Student Load (0.2ms)  SELECT "students".* FROM "students"  ORDER BY "students"."updated_at" DESC
Completed 304 Not Modified in 8ms (ActiveRecord: 0.8ms)
```

The `304 Not Modified` message basically tells the cache that the cached copy is still fresh and we can just serve that. Notice that the request process time is significantly decreased (from 45ms to 8ms, decreased by 82%).

So conditional GET seems to be pretty useful, but how does it work exactly? Let’s take a look at the requests and responses.

The first response with `200 OK` status code comes with the following headers:

```
Cache-Control: max-age=10, public, must-revalidate
Etag: "6eae1c9b5780f3ae4abf8212cc5a566a"
Last-Modified: Thu, 27 Aug 2015 13:43:32 GMT
```

The second response, however, has something special:

```
If-Modified-Since: Thu, 27 Aug 2015 13:43:32 GMT
If-None-Match: "6eae1c9b5780f3ae4abf8212cc5a566a"
```

By matching the headers of the first response and the second request, we can already have a rough guess on the revalidation mechanism. Our origin server sets something called `Etag` and `Last-Modified` in the response. The browser cache saves them with the cached copy and uses them to revalidate in subsequent requests.

## Etag & IF-None-MATCH

`The Etag`, or *entity tag*, is the version identifier of the resource. We can use **any** version management algorithm on the server side to generate etags, as long as different Etags can identify different resource content during a relatively long period of time. Rails by default calculates a message digest from the resource data. When the Etag is sent in subsequent requests in `If-None-Match` header, the origin server will calculate the Etag of the resource again and compare the two. If they don’t match, that means the cached copy is stale.

## LAST-MODIFIED & IF-MODIFIED-SINCE

`Last-Modified` is just the last modified timestamp of the resource. When the subsequent requests send it via `If-Modified-Since` header, the origin server fetches the last modified timestamp of the resource again and compares the two. If the resource has been modified after the timestamp in the request, the cached copy is considered stale.

Although these two methods of revalidation seem to both work fine, it’s highly recommended for the origin server to provide both — if and only if both of them consider the cached copy fresh, the cache is then allowed to serve the cached version. Why? Because both of them have some shortfalls. For `Last-Modified` header, its time precision is not high enough to handle very frequently changed resources (changed on the order of milliseconds). For `Etag`, since it’s generally not realistic to manage a version control system on server side for all resources, message digest algorithms (like MD5) are often used instead and they are prior to collision. Using both makes it almost impossible for the origin server to consider a stale copy as fresh.

# Best Practice in Rails

To make sure that revalidation works properly, here is something that has to be taken care of on Rails side.

First of all, if the resource is passed directly as the value of the `etag` key as we just did (`etag: @students` in Code Snippet 5), the `cache_key` method of the resource will be used to calculate the Etag. In this case, we should make sure that the `cache_key` method reflects the definition of fresh/stale resource in your business domain. Sometimes we may not want all properties of the resource go to into the calculation, because the more properties involved, the more strict caching control becomes. Since the client request is a representation of the resource (notice it’s never the resource itself), it’s not necessary to include some less important properties or user invisible properties.

Besides that, the last modified timestamp is usually the `updated_at` property of the resource (or the maximum among all resources in a collection if the collection is requested). It gives us two hints:

* for all models that we wish to cache, make sure the `timestamps` macro is used in the migration
* if you update any properties of the model that are use visible and should be involved in the revalidation, make sure you use `update_attributes` , `save` or `save!` instead of `update_attribute`, even if only one property is updated, because `update_attribute` doesn’t changed `updated_at`

There are other methods provided by Rails for cache control:

* `stale?`  - This method behaves almost the same as `fresh_when` . In fact, it calls `fresh_when` to handle the revalidation logic underneath. The only difference is that `stale?` allows us to customize the render behavior (for example rendering a different view, as shown in code snippet 6), while `fresh_when` only follows the default rendering behavior.
* `expires_now` - This method simply attaches a `Cache-Control: no-cache` header to the response (as shown in code snippet 3). When a `no-cache` header is present, it shadows other Cache-Control headers, and it’s pretty much identical to `Cache-Control: max-age=0, must-revalidate`. Notice that it doesn’t mean the resource is not allowed be cached; it just means each request of the resource has to revalidate before the cached copy is served.

```ruby
##################################################################
# Code Snippet 3
# Uncomment to make the response stale immediately
# Cache-Control: no-cache
##################################################################
expires_now
```

Last of all, to completely forbid the cache from caching resources, a `Cache-Control: no-store` header should be used. When this header is present, the cache **must not** keep any copy of the resource at all. There’s no method in Rails for this but you can manually attach it in the headers, as shown in code snippet 4.

```ruby
##################################################################
# Code Snippet 4
# Uncomment to stop generating cached copy at all
# Cache-Control: no-store
##################################################################
headers['Cache-Control'] = 'no-store'
```

# Summary

From the above results, we can see that:

* Caching and revalidation can significantly reduce the server load and improve the user experience
* Rails's default caching behavior works well, but it also gives us more fine-grained control of caching from server side

However, HTTP cache still has the following limitations:

* The client can still decide to **not accept** the cached copy
* Some HTTP clients don’t have built-in cache and don’t work in a network with shared cache
* When revalidation results in a stale copy, the server side will have to load the data and do some complex processing if necessary to return a fresh copy

In the next post of this series I will talk about our practice of server-side caching in Strikingly.

# References

1. Rails Conditional Get API document: http://api.rubyonrails.org/classes/ActionController/ConditionalGet.html
2. RFC 2616 (HTTP/1.1), section 14: http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html

> Thanks to Dafeng Guo, Teng Bao and Florian Dutey, who helped review this article.

Daniel Gong

Backend Engineer @ Strikingly.com
