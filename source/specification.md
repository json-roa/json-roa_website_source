---
title: Specification
---

{::options parse_block_html="true" /}


# Specification and Interpretation
{: #specification .no_toc}

This is an informal specification of the JSON-ROA extension and how clients
must interpret it.

We also provide some examples for illustration. These are mostly taken form the
[JSON-ROA Demonstration Application][]. 


## Table of Contents 
{:.no_toc}
* Will be replaced with the ToC, excluding the "Contents" header
{:toc}


## General Properties
{: #general}

### Semantic Versioning 
{: #semantic-versioning}

JSON-ROA uses [Semantic Versioning][].  The semantics apply to the [JSON-ROA
Object][] itself and precisely whether changes thereof would break JSON-ROA
clients implemented for an earlier version. Experimental properties are
excluded from this rule.

A client **must** evaluate the given semantic version and it **must fail** with
an appropriate error if it is not compatible with the given _major number_. It
may indicate discrepancies of the _minor number_ but it **must process** the
request. 

The semantic version of JSON-ROA is strictly limited to JSON-ROA itself. It
does not apply to the application.
{: .text-warning}

### The Content-Type 
{: #content-type}

The JSON-ROA extension shall be present if and only if the resource is
delivered with the content type `application/json-roa+json`.

## The JSON-ROA Object 
{: #json-roa-object }

<div class="row"> <div class="col-md-6">

The JSON-ROA Object is located under the `_json-roa` key if the top level
element is an object. If the top level element is an array the first element is
a map with the `_json-roa` key and the corresponding map underneath. 

In this sense, a given json-object can be extended by adding  a `_json-roa` key
including its object.  A given json-array can be extended by placing an object
which includes a `_json-roa` key and its corresponding object in the first
position.
{: .text-info}

It is discouraged to use arrays as the outer most data structure. Maps 
are more flexible and can be extended in less obtrusive way. 
{: .text-warning}

A minimal valid JSON-ROA extension must contain the key `version`
where the value must be formatted according to [Semantic Versioning][]. See
also the [section Semantic Versioning][].

It may further contain the keys `relations`, `collection`, `name`, and
`self-relation`. See the [relations object][], the [collection object][],
and the [relation object][] for the specification of the corresponding
values.

</div> <div class="col-md-6">

    { "_json-roa": { "version" : "1.0.0" }
      , "x": "some other data"
      , "y": 42
      , "z": false }
  {: .language-json}

    [ {"_json-roa": {"version" : "1.0.0" }},
      "some other data",
      42,
      false, ]
  {: .language-json}

</div> </div>


### The Relations Object
{: #relations-object}

<div class="row"> <div class="col-md-6">

The key `relations` introduces a relations object. The keys in the next level
within are _relation-identifiers_ and the corresponding object is
a _relation-object_, see the section [The Relation Object][].

</div> <div class="col-md-6">

    {"_json-roa" :
      {"version": "1.0.0",
        "relations": {
          "messages": {
            "href": "/messages/" }}}}
  {: .language-json}

</div> </div>



#### The Relation Identifier
{: #relation-identifier}

<div class="row"> <div class="col-md-6">

The **identifier of a relation** is meant to be used in clients to dereference
the relation object and in particular use the `href`-value therein. The example
illustrates how to request the `messages` resource given a `root` resource with
the [JSON-ROA Ruby Client][]. 

</div> <div class="col-md-6">
    root.relation('messages').get()
  {: .language-ruby}
</div> </div>


#### The Relation Object
{: #relation-object}
<div class="row"> <div class="col-md-6">

A relation is an object which must contain the key `href`. It may
contain the keys `embedded`,`name`, `methods`, and `relations`. See the
section [Meta Relations][] for the interpretation of `relations`. See
the section [Embedding Resources][] for usage and interpretation of
`embedded`.

</div> <div class="col-md-6">
    { "relations": 
      { "messages": 
        { "href": "/messages/" }}}
  {: .language-json}
</div> </div>


##### The Href Value
{: #href-value}
<div class="row"> <div class="col-md-6">

The value of the key `href` is a string and represents an URI according to
[RFC3986 Uniform Resource Identifier][]. It must at least contain a path
segment. The values before the path segment are inferred form the current
resource if they are not given. The URI may be templated according to [RFC6570
URI Template][].

The second example shows how values for templated parameters can be submitted
with the [JSON-ROA Ruby Client][]. 


</div> <div class="col-md-6">

    { "relations": 
      { "message": 
        { "href": "/messages/{id}"}}}
  {: .language-json}

    root.relation('message') \
      .get('id' => '4e762513-d903-4228-b92c-da4f0cb3094b')
  {: .language-ruby}

</div> </div>


##### The Name Value
{: #name-value}
<div class="row"> <div class="col-md-6">

The name should not be used for technical purposes. The name is used, e.g. by
the [JSON-ROA Browser][], to give a understandable output for the human user.

</div> <div class="col-md-6">

    { "relations": 
      { "message": 
        { "href": "/messages/{id}", 
          "name": "Message"}}}
  {: .language-json}

</div> </div>



##### The Methods Object 
{: #methods}
<div class="row"> <div class="col-md-6">

The methods object may contain the keys `get`, `put`, `patch`, `post`, and `delete`. 
Each value is an empty object at this time. 

Clients should interpret the keys as the available HTTP-method to act on the
given url respectively resource they address. 

If the **no** `methods` are  given, clients **must assume** that `get` and only
`get` is available. 

</div> <div class="col-md-6">

    { "relations": 
      { "messages": 
        { "href": "/messages/",
          "name": "Messages",
          "methods": 
            { "get": {},
              "post": {}}}}}
  {: .language-json}

</div> </div>


#####  Meta Relations 
{: #meta-relations}
<div class="row"> <div class="col-md-6">

A [Relation Object][] can itself contain a [Relations Object][] under the key
`relations`. The primary use is to link to the corresponding resource
documentation.

</div> <div class="col-md-6">

    {"_json-roa" :
      {"version": "1.0.0",
        "relations": {
          "messages": {
            "href": "/messages/",
            "name": "Messages",
            "relations": {
              "messages-documentation": {
                "name": "API Messages Resource Documentation",
                "href": "/docs/index.html#messages" }}}}}}
  {: .language-json}

</div> </div>


### The Collection Object
{: #collection-object}
<div class="row"> <div class="col-md-6">

The _collection_ object semantically links to a number of structurally
equivalent resources. Conventionally only url paths ending with a slash , e.g.
`/tasks/`, will contain a collection. This convention is apparently violated
by many sites and even frameworks. 

The collection object must contain a `relations` key 
which contains as described in the section [relations object][].
It is suggested to use integers as keys which represent 
the index of the given link within the collection.

The collection can contain a `next` key with the value of a [relation
object][] with the following restriction: the included url may not be
templated. Clients use `next` to iterate of a paginated collection. API
providers should keep the search parameters between successive next link
(with the exception of the parameter used for pagination if present).

Any of the following two conditions signals the end of the collection for
a client: 

1. The relations object is empty. 
2. There is no next key.

</div> <div class="col-md-6">


    {"_json-roa" :
      { "version": "1.0.0",
        "collection": 
          { "next": { "href": "/messages/?page=1" },
            "relations": 
              { "1": 
                { "href": "/messages/2f09edb9-5aec-460f-9e6a-5e9b980e8f05"}, 
                "2": 
                { "href": "/messages/4e762513-d903-4228-b92c-da4f0cb3094b"}}}}}
  {: .language-json}

</div> </div>



## Embedding Resources

<div class="row"> <div class="col-md-6">

Embedding resources is an experimental proposal. There does not exist any
implementation or proof of concept at this time. Any changes related to this
feature do not warrant a lifting of the major number!
{: .text-warning}

A [Relation Object][] can contain the key `embedded`. The corresponding value
map must be equal to the to JSON transformed response body given by requesting
the resource with HTTP GET and receiving the representation of content-type
`application/json-roa+json`. Therefore the following restrictions apply: 

1. The `href` value of the relation must not be a template url.  
2. The http get method must be applicable to the resource. 

Handling of embedded resources must be completely transparent. I.e. clients
must not depend on the existence of an embedded object. On the other hand are
clients not required to honor `embedded` resources.

JSON-ROA does not define any technique how to negotiate respectively request
embeddings. This is application and context dependent. A custom HTTP header
seems to be appropriate for many cases. 


</div> <div class="col-md-6">
</div> </div>


<div class="row"> <div class="col-md-6">
</div> <div class="col-md-6">
</div> </div>


  [Embedding Resources]: #embedding-resources
  [JSON-ROA Browser]: http://json-roa.github.io/browser/index.html
  [JSON-ROA Demonstration Application]: https://json-roa-demo.herokuapp.com/api-browser/index.html#/
  [JSON-ROA Object]: #json-roa-object
  [JSON-ROA Ruby Client]: https://github.com/json-roa/json-roa_ruby-client
  [Meta Relations]: #meta-relations
  [RFC3986 Uniform Resource Identifier]: https://tools.ietf.org/html/rfc3986
  [RFC6570 URI Template]: https://tools.ietf.org/html/rfc6570
  [Relation Identifier]: #relation-identifier
  [Relation Object]: #relations-object
  [Semantic Versioning]: http://semver.org/
  [collection object]: #collection-object
  [relation object]: #relation-object
  [relations object]: #relations-object
  [section Semantic Versioning]: #semantic-versioning

