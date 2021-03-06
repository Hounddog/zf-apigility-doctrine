Apigility for Doctrine
======================

Installation
------------

Installation of this module uses composer. For composer documentation, please refer to
[getcomposer.org](http://getcomposer.org/).

```sh
$ php composer.phar require zfcampus/zf-apigility-doctrine:dev-master
```

To use the Admin portion of this module for creating Doctrine APIs open `config/application.config.php` 
and add `ZF\Apigility\Doctrine` to your `modules`


API Resources
-------------

***/admin/api/module[/:name]/doctrine[/:controller_service_name]***


This is a Doctrine resource creation route like Apigility Rest `/admin/api/module[/:name]/rest[/:controller_service_name]`

POST Parameters
```json
{
    "objectManager": "doctrine.entitymanager.orm_default",
    "resourceName": "Artist",
    "entityClass": "Db\\Entity\\Artist",
    "pageSizeParam": "limit",
    "routeIdentifierName": "artist_id",
    "entityIdentifierName": "id",
    "routeMatch": "/api/artist",
    "hydratorName": "DbApi\\V1\\Rest\\Artist\\AlbumHydrator",
    "hydrateByValue": true
}
```


***/admin/api/doctrine[/:object_manager_alias]/metadata[/:name]***

This will return metadata for the named entity which is a member of the
named object manager.  Querying without a name will return all metadata
for the object manager.


Hydrating Entities by Value or Reference
----------------------------------------

By default the admin tool hydrates entities by reference by setting `$config['zf-rest-doctrine-hydrator']['hydrator_class']['by_value']` to false.  


Handling Embedded Resources
---------------------------

You may choose to supress embedded resources by setting
`$config['zf-hal']['renderer']['render_embedded_resources']` to false.  Doing so
returns only links to embedded resources instead of their full details.
This setting is useful to avoid circular references.  This was created during the
development of this module, although it is part of HAL it is documented here for safe keeping.


Collections 
===========

The API created with this library implements full featured and paginated 
collection resources.  A collection is returned from a GET call to a entity endpoint without
specifying the id.  e.g. ```GET /api/data_module/entity/user_data```

Reserved GET variables

```
    orderBy
    query
```

Sorting Collections
-------------------

Sort by columnOne ascending

```
    /api/user_data?orderBy%5BcolumnOne%5D=ASC
```

Sort by columnOne ascending then columnTwo decending

```
    /api/user_data?orderBy%5BcolumnOne%5D=ASC&orderBy%5BcolumnTwo%5D=DESC
```


Querying Collections
--------------------

Queries are not simple key=value pairs.  The query parameter is a key-less array of query 
definitions.  Each query definition is an array and the array values vary for each query type.

Each query type requires at a minimum a 'type' and a 'field'.  Each query may also specify
a 'where' which can be either 'and' or 'or'.  Embedded logic such as and(x or y) is not supported.

Building HTTP GET query with PHP.  Use this to help build your queries.

PHP Example
```php
    echo http_build_query(
        array(
            'query' => array(
                array('field' =>'cycle', 'where' => 'and', 'type'=>'between', 'from' => 1, 'to'=>100),
                array('field'=>'cycle', 'where' => 'and', 'type' => 'decimation', 'value' => 10)
            ),
            'orderBy' => array('columnOne' => 'ASC', 'columnTwo' => 'DESC')
        )
    );
```

Javascript Example
```js
$(function() {
    $.ajax({
        url: "http://localhost:8081/api/db/entity/user_data",
        type: "GET",
        data: {
            'query': [
            {
                'field': 'cycle',
                'where': 'and',
                'type': 'between',
                'from': '1',
                'to': '100'
            },
            {
                'field': 'cycle',
                'where': 'and',
                'type': 'decimation',
                'value': '10'
            }
        ]
        },
        dataType: "json"
    });
});
```

Querying Relations
---------------------
It is possible to query collections by relations - just supply the relation name as `fieldName` and
identifier as `value`.

1. Using an RPC created by this module for each collection on each resource: /resource/id/childresource/child_id

2. Assuming we have defined 2 entities, `User` and `UserGroup`...

````php
/**
 * @Entity
 */
class User {
    /**
     * @ManyToOne(targetEntity="UserGroup")
     * @var UserGroup
     */
    protected $group;
}
````

````php
/**
 * @Entity
 */
class UserGroup {}
````

... we can find all users that belong to UserGroup id #1 with the following query:

````php
    $url = 'http://localhost:8081/api/db/entity/user';
    $query = http_build_query(array(
        array('type' => 'eq', 'field' => 'group', 'value' => '1')
    ));
````


Expanding Collections * in development *
---------------------

You may include in the _GET[zoom] an array of field names which are collections 
to return instead of a link to the collection.

```
    /api/user_data?zoom%5B0%5D=UserGroup
```

Format of Date Fields
---------------------

When a date field is involved in a query you may specify the format of the date
using PHP date formatting options.  The default date format is ```Y-m-d H:i:s```
If you have a date field which is just Y-m-d then add the format to the query.

```php
    'format' => 'Y-m-d', 
    'value' => '2014-02-04',
```

Available Query Types
---------------------

ORM and ODM

Equals

```php
    array('type' => 'eq', 'field' => 'fieldName', 'value' => 'matchValue')
```

Not Equals

```php
    array('type' => 'neq', 'field' => 'fieldName', 'value' => 'matchValue')
```

Less Than

```php
    array('type' => 'lt', 'field' => 'fieldName', 'value' => 'matchValue')
```

Less Than or Equals

```php
    array('type' => 'lte', 'field' => 'fieldName', 'value' => 'matchValue')
```

Greater Than

```php
    array('type' => 'gt', 'field' => 'fieldName', 'value' => 'matchValue')
```

Greater Than or Equals

```php
    array('type' => 'gte', 'field' => 'fieldName', 'value' => 'matchValue')
```

Is Null

```php
    array('type' => 'isnull', 'field' => 'fieldName')
```

Is Not Null

```php
    array('type' => 'isnotnull', 'field' => 'fieldName')
```

In

```php
    array('type' => 'in', 'field' => 'fieldName', 'values' => array(1, 2, 3))
```

NotIn

```php
    array('type' => 'notin', 'field' => 'fieldName', 'values' => array(1, 2, 3))
```

Between

```php
    array('type' => 'between', 'field' => 'fieldName', 'from' => 'startValue', 'to' => 'endValue')
````

Like (% is used as a wildcard)

```php
    array('type' => 'like', 'field' => 'fieldName', 'value' => 'like%search')
```

Decimation (mod(field, value) = 0 e.g. value = 10 fetch one of every ten rows)

```php
    array('type' => 'decimation', 'field' => 'fieldName', 'value' => 'decimationModValue')
```


ORM Only
--------

Not Like (% is used as a wildcard)

```php
    array('type' => 'notlike', 'field' => 'fieldName', 'value' => 'notlike%search')
```


ODM Only
--------

Regex

```php
    array('type' => 'regex', 'field' => 'fieldName', 'value' => '/.*search.*/i')
```

