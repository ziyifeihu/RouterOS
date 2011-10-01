# Using queries
A brief guide to using queries

## Commands handling queries
Queries are a RouterOS API specific construct that allows you to limit the results returned for a request.

Currently, the "print" command is the only one that handles queries, since version 3.21. Net_RouterOS doesn't check whether the command handles queries, so if future versions of RouterOS have other such commands, you can use queries with them right away.

## Setting a query
To set a query for a request, you need to use the Request::setQuery() method. If later in the script you want to remove the query, you can pass NULL to it. The rest of the examples in this tutorial will assume a script similar to the following, where the $query variable is defined separately:
```php
<?php
namespace PEAR2\Net\RouterOS;
include_once 'PEAR2/Net/RouterOS/Autoload.php';
$client = new Client('192.168.0.1', 'admin');
 
$request = new Request('/ip/arp/print');
 
//Define $query here
 
$request->setQuery($query);
$responses = $client->sendSync($request);
 
foreach($responses as $response) {
    foreach($response->getAllArguments() as $name => $value) {
        echo "{$name}: {$value}\n";
    }
    echo "====\n";
}
?>
```
## A simple query
You can create a query by calling the static Query::where() method, along with the first criteria of the query. For example, if you wanted to limit results to the entry about 192.168.0.100, you can use a query like:
```php
$query = Query::where('address', '192.168.0.100');
```
Using the optional third parameter, you can specify exactly what do you want to do with the value. Possible values are the Query::ACTION_* constants. For example, if you wanted to get all addresses greather than 192.168.0.100, you can use:
```php
$query = Query::where('address', '192.168.0.100', Query::ACTION_GREATHER_THAN);
```
## Chaining conditions
The Query class uses a "fluent" interface, i.e. it always returns the query object itself, similarly to how [jQuery](http://jquery.com) and [Zend_Db_Select](http://framework.zend.com/manual/en/zend.db.select.html) do it. Thanks to that, you can chain conditions right when defining the $query variable (though you can also alter it later). For example, if you wanted to get all addresses greather than or equal to 192.168.0.100, you can do:
```php
$query = Query::where('address', '192.168.0.100', Query::ACTION_GREATHER_THAN)->orWhere('address', '192.168.0.100');
```