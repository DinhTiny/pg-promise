# SELECT ⇒ INSERT

The following scenario is very common in adding new records:

* try finding a specific record, and if found - return its id;
* if no record found, add a new one and return the new id.

```sql
-- a simple table example
CREATE TABLE Users(
	id SERIAL PRIMARY KEY,
	name TEXT UNIQUE -- unique user name
);
```

Let's implement a function, according to that logic, to search by user's `name`, and return
a new or existing `id` of the record. 
 
```js
function getInsertUserId(name) {
    return db.task(function (t) {
            return t.oneOrNone('SELECT id FROM Users WHERE name = $1', name)
                .then(function (user) {
                    return user || t.one('INSERT INTO Users(name) VALUES($1) RETURNING id', name);
                });
        })
        .then(function (user) {
            return user.id;
        });
}
```

Example of calling the function:

```js 
getInsertUserId('name')
    .then(function (userId) {
        // use the id;
    })
    .catch(function (error) {
        // something went wrong;
    });
```

The same function `getInsertUserId`, simplified for the ES6 syntax:

```js
function getInsertUserId(name) {
    return db.task(t=>t.oneOrNone('SELECT id FROM Users WHERE name = $1', name)
        .then(user=>user || t.one('INSERT INTO Users(name) VALUES($1) RETURNING id', name)))
        .then(user=>user.id);
}
```

The same function `getInsertUserId`, using ES6 generators:

```js
function getInsertUserId(name) {
    return db.task(function *(t) {
        let user = yield t.oneOrNone('SELECT id FROM Users WHERE name = $1', name);
        user = yield user || t.one('INSERT INTO Users(name) VALUES($1) RETURNING id', name);
        return user.id;
    });
}
```

## Single-query alternative

It is possible to wrap the whole operation into a single query, which would offer a better
performance, executing always just one query, and more importantly - proper data integrity,
by making sure that a parallel request wouldn't try to execute a second `INSERT`. 

Implementing such a query however isn't trivial, and can vary based on whether it is for
PostgreSQL 9.5+ or an older version of the server.

The following posts will help you get started writing your own single-query solution for this:

* [Id from a conditional INSERT](http://stackoverflow.com/questions/36083669/get-id-from-a-conditional-insert)
* [SELECT ⇒ INSERT in a function](http://stackoverflow.com/questions/15939902/is-select-or-insert-in-a-function-prone-to-race-conditions)
