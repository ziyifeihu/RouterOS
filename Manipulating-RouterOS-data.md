# Manipulating RouterOS data

## Getting individual properties
If you try to use shell's "get" command with the API protocol, you'll find it doesn't really work. It returns just a single empty response, as if only to confirm the command's existence.

Getting an individual property is not directly possible with the API protocol, but it can be closely emulated by the use of the "print" command.

A "print" command can take queries, which are described in more detail in [a separate tutorial](Using-queries). Combined with the API specific ".proplist" argument (also described in that tutorial), you can make the print command return just one reply, containing just the one property of the one entry that interests you.

For example, if you were interested in getting the MAC address of a certain "arp" entry you've targeted by its IP address:

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Net/RouterOS/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');

$printRequest = new Request('/ip/arp/print');
$printRequest->setArgument('.proplist', 'mac-address');
$printRequest->setQuery(Query::where('address', '192.168.0.100'));
$mac = $client->sendSync($printRequest)->getArgument('mac-address');
//$mac now conains the MAC address for the IP 192.168.0.100
?>
```

## The "numbers" argument
All commands that require you to use the "numbers" argument to target an entry (enable, disable, set, unset and remove), ironically enough, do not accept numbers as their value. Instead, they accept a list of IDs.  __The IDs you get from API are NOT the same as the numbers you see in the "#" column in shell/Winbox.__

There is no way of knowing the IDs without doing a "print" of the targeted entry - from the API protocol - at some point. Ideally, you should do it right when you're about to target the entry. When you do that, the ID is available from an API specific argument called ".id".

So for example, to delete an entry in the ARP list that has a comment saying "del", we'd do:

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Net/RouterOS/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');

$printRequest = new Request('/ip/arp/print');
$printRequest->setArgument('.proplist', '.id');
$printRequest->setQuery(Query::where('comment', 'del'));
$id = $client->sendSync($printRequest)->getArgument('.id');
//$id now contains the ID of the entry we're targeting

$removeRequest = new Request('/ip/arp/remove');
$removeRequest->setArgument('numbers', $id);
$client->sendSync($removeRequest);
?>
```

or if we wanted to change the comment of that entry, we would do:

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Net/RouterOS/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');

$printRequest = new Request('/ip/arp/print');
$printRequest->setArgument('.proplist', '.id');
$printRequest->setQuery(Query::where('comment', 'del'));
$id = $client->sendSync($printRequest)->getArgument('.id');
//$id now contains the ID of the entry we're targeting

$setRequest = new Request('/ip/arp/set');
$setRequest->setArgument('numbers', $id);
$setRequest->setArgument('comment', 'new comment');
$client->sendSync($setRequest);
?>
```

Keep in mind we're talking about a list - a comma separated list to be exact. So if you need to, you can have several IDs lined up, like if you had multiple ARP entries for an IP, you can remove them all at once with something like the following:

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Net/RouterOS/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');

$printRequest = new Request('/ip/arp/print');
$printRequest->setArgument('.proplist', '.id');
$printRequest->setQuery(Query::where('address', '192.168.0.100'));

$idList = array();
foreach ($client->sendSync($printRequest)->getAllOfType(Response::TYPE_DATA) as $entry) {
    $idList[] = $entry->getArgument('.id');
}
//$idList now contains an array of all targeted entries' IDs.

$removeRequest = new Request('/ip/arp/remove');
$removeRequest->setArgument('numbers', implode(',', $idList));
$client->sendSync($removeRequest);
?>
```
