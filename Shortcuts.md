# Shortcuts
PEAR2_Net_RouterOS allows you to write not just code that works, but code that looks readable.

A lot of the examples you see in other tutorials are intentionally verbose, so that you can clearly see what's happening. But once you get used to these constructs, you'll probably start to find this verbosity annoying. Here are some shortcuts PEAR2_Net_RouterOS allows you to use.

## set*() chaining
The setArgument(), setTag() and setQuery() from the Request object all return the request object itself. This means you can chain one after the other. For example, you can write

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');
 
$addRequest = new Request('/ip/arp/add');
 
$addRequest->setArgument('address', '192.168.0.100');
$addRequest->setArgument('mac-address', '00:00:00:00:00:01');
if ($client->sendSync($addRequest)->getType() !== Response::TYPE_FINAL) {
    die("Error when creating ARP entry for '192.168.0.100'");
}
 
$addRequest->setArgument('address', '192.168.0.101');
$addRequest->setArgument('mac-address', '00:00:00:00:00:02');
if ($client->sendSync($addRequest)->getType() !== Response::TYPE_FINAL) {
    die("Error when creating ARP entry for '192.168.0.101'");
}
 
echo 'OK';
?>
```

as

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');
 
$addRequest = new Request('/ip/arp/add');

if ($client->sendSync(
    $addRequest
        ->setArgument('address', '192.168.0.100')
        ->setArgument('mac-address', '00:00:00:00:00:01')
    )->getType() !== Response::TYPE_FINAL
) {
    die("Error when creating ARP entry for '192.168.0.100'");
}

if ($client->sendSync(
    $addRequest
        ->setArgument('address', '192.168.0.101')
        ->setArgument('mac-address', '00:00:00:00:00:02')
    )->getType() !== Response::TYPE_FINAL
) {
    die("Error when creating ARP entry for '192.168.0.101'");
}
 
echo 'OK';
?>
```

In addition to those, the removeAllArguments() returns the request object too. Client::sendAsync() is also another method that returns the object itself, which means you could chain several requests one after another, e.g. the code

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');
 
$addRequest = new Request('/ip/arp/add');
 
$addRequest->setArgument('address', '192.168.0.100');
$addRequest->setArgument('mac-address', '00:00:00:00:00:01');
$addRequest->setTag('arp1');
$client->sendAsync($addRequest);
 
$addRequest->setArgument('address', '192.168.0.101');
$addRequest->setArgument('mac-address', '00:00:00:00:00:02');
$addRequest->setTag('arp2');
$client->sendAsync($addRequest);
 
$client->loop();
```
could be written as

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');
 
$addRequest = new Request('/ip/arp/add');

$client
    ->sendAsync(
        $addRequest
            ->setArgument('address', '192.168.0.100')
            ->setArgument('mac-address', '00:00:00:00:00:01')
            ->setTag('arp1')
    )
    ->sendAsync(
        $addRequest
            ->setArgument('address', '192.168.0.101')
            ->setArgument('mac-address', '00:00:00:00:00:02')
            ->setTag('arp2')
    )
    ->loop();
```

## \_\_invoke() magic
Most objects can be invoked as functions, and which point they're like a shorthand for the most common functionality of the object. You can see full details in the API reference, under the \_\_invoke() magic method's description for that object. Using that, the example above could be written as

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
$client = new Client('192.168.0.1', 'admin');
 
$addRequest = new Request('/ip/arp/add');
 
$addRequest('address', '192.168.0.100');
$addRequest('mac-address', '00:00:00:00:00:01');
$addRequest->setTag('arp1');
$client($addRequest);
 
$addRequest('address', '192.168.0.101');
$addRequest('mac-address', '00:00:00:00:00:02');
$addRequest->setTag('arp2');
$client($addRequest);
 
$client();
```

## I wish...
There are currently [an RFC about function chaining](https://wiki.php.net/rfc/fcallfcall), with a patch in it. Unfortunately, the patch is not yet implemented in a stock PHP. If this patch is ever implemented (or if you're willing to compile PHP yourself), you'd be able to write some extremely fancy code. The above for example could be rewritten as

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';
 
(new Client('192.168.0.1', 'admin'))
    (
        ($addRequest = new Request('/ip/arp/add'))
        ('address', '192.168.0.100')
        ('mac-address', '00:00:00:00:00:01')
        ->setTag('arp1')
    )
    (
        $addRequest
        ('address', '192.168.0.101')
        ('mac-address', '00:00:00:00:00:02');
        ->setTag('arp2');
    )
 ();