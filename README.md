# Node / Redis REST / CRUD Server Boilerplate  

This project is meant as a boilerplate for developing create-read-update-delete (CRUD) applications with Node.js and Redis using REST-like endpoints. It uses the Express.js framework, but should be adaptable to any middleware-based framework with minimal fuss.

Since this is a boilerplate, nothing extra is provided. You'll need to wire in your authentication, handlers, etc. 

The goal was to provide clean code that shows the direct use of Redis-as-a-database. The code hygiene here uses minimal string literals for flexibility and reuse. It also employs sub-routers for modularity. Finally, responses are handled in one of four ways:

1. HTTP 200 (Success) `sc200OnRedisSuccess` - If the operation is successful. It has options to return the result of the Redis operation, arbitrary data or nothing.
2. HTTP 404 (Not Found) `sc404onRedis0Result` - If the operation is successful, but returns a `0`. This failure condition is usually if it can't find an item (`exists`, `sismember`, etc.)
3. HTTP 404 (Not Found) `sc404onRedisNullResult` - If the operation is successful, but returns a `null`/`nil`. This failure condition is primarily used when something is not found and it cannot return a 0 so instead returns a `null`/`nil` (`zscore`)
4. Other responses. If a command fails for some reason (network connection, OOM, etc.) then the client library will return an error. All the above response handlers will catch the error and return it back to Express which will generate a generic error (usually response code 500).

Currently included in this module are three primary datatypes: lists, sets, and sorted set + hash combo. Lists allow for a simple linked list based on the Redis list commands. Sets are a direct translation of a Redis Set to a CRUD system. The sorted set and hash combo uses the sorted sets as a way of keeping tack of individual hashes. The sorted set scores are based on the timestamp at which the item was created.

Since this module uses sub-routers, we're using the concept of a 'base route'. In the example, this is between the first two slashes in the URL and directly corresponds to the first part of the key in your Redis database. As an example:

```
/cars/my-car
```
The base would be 'cars'. All the keys associated with this route would start with 'cars:'. In the routes, this is integrated as `req.params.base`. This opens up many possibilities for very concise code. As an example, if you wanted to create three different lists, you could specify the base with the regular expression `one|two|three` which would map to list keys 'one' 'two' and 'three'. 

Because this is based on sub-routers, you can create complex routes that just _end with these routes_. So, you can have upstream middleware that provide extras such as authentication.

**List Endpoints:** Defined in `list.module.node.js`
* **Create** POST `/`_base_`/`_value_
* **Read** GET `/`_base_`/` (read all)
* **Read** GET `/`_base_`/`_start index_ (read from index)
* **Read** GET `/`_base_`/`_start index_`/`_end index_ (read from index to index)
* **Update** PUT `/`_base_`/`_index_`/`_value_ (update a value at index e.g. replace)
* **Delete** DELETE `/`_base_`/`_value_ (delete a value)

**Set Endpoints:** Defined in `set.module.node.js`
* **Create** PUT `/`_base_
* **Read** GET `/`_base_ (read all members)
* **Read** GET `/`_base_`/`_member_ (read a member e.g. does it exist)
* **Update** _this doesn't make sense in context of a set, so it's omitted_
* **Delete** DELETE `/`_base_`/`_member_ (remove a member from a set)

**Sorted Set + Hash Endpoints** Defined in `sorted-hash.module.node.js`
* **Create** POST `/`_base_
* **Read** GET `/`_base_`/`_uniqueId (read a member)
* **Read** GET `/`_base_`/`_min_` (read starting at min)
* **Read** GET `/`_base_`/`_min_`/`_max_ (read starting at min and ending at max)
* **Update** PATCH `/`_base_`/`_uniqueId (update a member)
* **Delete** DELETE `/`_base_`/`_uniqueId (delete a member)

*MIT License*

```
Copyright 2018 Kyle J Davis

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
```
