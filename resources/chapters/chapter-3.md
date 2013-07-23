# Parameterized queries

Looking at the query

    [:find ?title
     :where 
     [?p :person/name "Sylvester Stallone"]
     [?m :movie/cast ?p]
     [?m :movie/title ?title]]

It would be great if we could reuse the above query to find movie
titles of any actor and not just "Sylvester Stallone". This is
possible with an `:in` clause which provides the query with input
parameters much in the same way as function or method arguments in
your programming language. 

Rewriting the above query to accept the name of any person:

    [:find ?title
     :in $ ?name
     :where 
     [?p :person/name ?name]
     [?m :movie/cast ?p]
     [?m :movie/title ?title]]

This query accepts two arguments: `$` is the database
itself (implicit, if no `:in` clause is specified) and `?name` which
presumably will be the name of some actor. The above query could be
executed like `Peer.q(query, conn.db(), "Sylvester Stallone")` where
`query` is the query above. It's possible to have any number of inputs
to a query

In the above query the input pattern variable `?name` is bound to a
scalar (a string in our example). There are four different kinds of
input: scalars, tuples, collections and relations.

## Tuples

A tuple input is written as e.g. `[?name ?age]` and can be used when
you want to destructure an input. Let's say you have the vector
["James Cameron", "Arnold Schwarznegger"] and you want to use this
as input to find all movies where these two people collaborated:

    [:find ?title
     :in $ [?director ?actor]
     :where 
     [?d :person/name ?director]
     [?a :person/name ?actor]
     [?m :movie/director ?d]
     [?m :movie/cast ?a]
     [?m :movie/title ?title]]

## Collections

You can use collection destructuring to implement a kind of logical **or** in your query. Say, you want to find all movies directed by either James Cameron **or** Ridley Scott:

    [:find ?title
     :in $ [?director ...]
     :where
     [?p :person/name ?director]
     [?m :movie/director ?d]
     [?m :movie/title ?title]]

This query could be called with `(q query (db conn) ["James Cameron" "Ridley Scott"])`.

## Relations

Say you have found some really interesting data on imdb:

    [
     ... 
     ["Die Hard" 8.3]
     ["Alien" 8.5] 
     ["Lethal Weapon" 7.6]
     ["Commando" 6.5]
     ...
    ]

and you'd like to query this data and join it with data from the database. The following query finds the ratings for a given director:

    [:find ?rating
     :in $ ?director [[?title ?rating]]
     :where 
     [?p :person/name ?director]
     [?m :movie/director ?p]
     [?m :movie/title ?title]]