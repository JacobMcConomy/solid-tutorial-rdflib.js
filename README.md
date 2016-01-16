# Tutorials for using rdflib.js

This tutorial will walk you through what [rdflib.js](https://github.com/linkeddata/rdflib.js) can do, and how you can build Solid apps with it.

## Namespace shortcuts

The system uses Javascript objects for RDF symbols. Generating them for predicates is simplified by using a namespace function, turning them into prefixes:

```Javascript
var RDF = Namespace("http://www.w3.org/1999/02/22-rdf-syntax-ns#")
var RDFS = Namespace("http://www.w3.org/2000/01/rdf-schema#")
var FOAF = Namespace("http://xmlns.com/foaf/0.1/")
var XSD = Namespace("http://www.w3.org/2001/XMLSchema#")
```

So now, instead of typing the whole URI for your predicate, i.e.:

```
$rdf.sym('http://xmlns.com/foaf/0.1/knows')
```

You can type:

```
var foafKnows = FOAF('knows');
```

## RDF term types

These are types of nodes in the RDF graph. We call them terms because they are like terms in a language when you think of RDF as a real language.

| What | How | Term type |
|--------|------|--------|
| Node identified by a URI | `x = $rdf.sym(uri)` | symbol	|
| Blank node | `x = $rdf.bnode()` | 'bnode' |
| Untyped Literal | `x = $rdf.literal('abc')` | literal |
| Typed Literal | `x = $rdf.literal('8080', undefined, XSD('int'))` | literal |
| Literal with language | `x = $rdf.literal('car', 'en')` | literal |
| Ordered list | `x = $rdf.list([node1, node2])` | collection	|

## Creating a data store (graph)

```Javascript
var store = new $rdf.graph()
```

You can then load a graph from an existing URL:

```Javascript
var store = new $rdf.graph()
store.load('https://www.w3.org/People/Berners-Lee/card')
```

## Using data in the store

There are two ways to look at RDF data in a store. You can synchronously use the `each()` and `any()` methods and `statementsMatching()`, or you can do a query which returns results asynchronously.

The `each()`, `any()` and `statementsMatching()` take a pattern of subject, predate, object and source, where for `each()` and `any()` one of s p and o are undefined and source may be undefined or not. For example, using `$rdf.sym()` to make an object for an RDF node (symbol),

```Javascript
var me = $rdf.sym('https://www.w3.org/People/Berners-Lee/card#i');
var knows = FOAF('knows')
var friend = store.any(me, knows)  // Any one person 
```

or using the vocabulary namespace straight away

```Javascript
var me = $rdf.sym('http://www.w3.org/People/Berners-Lee/card#i');
var friend = store.any(me, FOAF('friend'))
```

## Wildcards

When one of the terms of the triple is set as *undefined*, it then serves as a wildcard. In this case, the formula returns the object of a matching triple. 

```Javascript
var friend = store.any(me, FOAF('age'), undefined)  // Any one age value
```

Alternatively, you get a javascript array of objects using `each()`.

```Javascript
var friends = store.each(me, FOAF('knows'), undefined)
for (var i=0; i<friends.length;i++) {
    friend = friends[i]
    console.log(friend.uri) // the WebID of a friend
    ...
}
```

## Adding data

The `add(s, p, o, w)` method allows a statement to be added to a formula. The optional `w` argument can be used to keep track of which resource was the source (URI) for each triple.

```Javascript
store.add(me, FOAF('knows'), $rdf.sym('https://fred.me/profile#me)
store.add(me, FOAF('name'), "Albert Bloggs")
```
 
Note above where the RDF thing in question is a literal, you can just pass a Javascript literal and a string, boolean, or numeric literal of the appropriate datatype will be used.

The `each()`, `any()` and `statementsMatching()` functions in fact can take a fourth argument to select data from a particular source. For example, I want to find **only** the friends in my foaf file, so I force use of my foaf file as the `w` argument.

```Javascript
var myFoafFile = $rdf.sym(uri)
var friendsInMyFOAF = store.each(me, FOAF('knows'), undefined, myFoafFile)
```