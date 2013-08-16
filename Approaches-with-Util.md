# What is Util?
Util is a class which wraps around a Client object to make frequent tasks easier. It doesn't give you full control however, which is why you may still find yourself using Client from time to time, even if you're using Util most of the time.

# Basic setup
## New instance
Similarly to how you do it with Client. In fact, as already mentioned, you need to create a Client to wrap around, e.g.:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util(new RouterOS\Client('192.168.0.1', 'admin'));
//Use $util from here on
```

If you also need the raw power of Client, you may want to also save it into an additional variable, e.g.:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
//Use $util or $client from here on
```

## Navigating around
Before executing a command with Util, you must first navigate to the menu you'll perform operations on. You do that with the changeMenu() method. Without arguments, it will give you the current menu. With an argument, you switch to a specified menu, and the method returns the previous menu you were on. By default you're at the root menu "/". If, for example, you want to edit ARP entries, you'd do:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/ip arp');
//Target entries in the "/ip arp" menu from here on
```

Notice that you can use shell syntax, as well as API syntax. In addition, you can also use relative shell paths, e.g.
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/ip arp');//We're now at "/ip arp"
$util->changeMenu('.. addresses');//We're now at the "/ip addresses" menu.
//Target entries in the "/ip addresses'" menu from here on
```

# CRUD operations
Util has an add(), get(), set(), move(), remove(), enable() and disable() methods, and you can probably already figure out what each one of them does. The important thing to keep in mind that _in addition_ to accepting IDs to target, each of these methods can also accept numbers, just like in terminal. This is implemented ON TOP of the API protocol, which doesn't support this natively. This is in fact Util's main super power compared to a plain Client.

Let's look at some examples...
## add()
The add() method accepts an array of properties to assign to a new entry in the current menu. You can also supply multiple arguments, each of which will be treated as an entry to be added.

Here's the second example from the last tutorial, rewritten with Util in mind:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/ip arp');
$util->add(
    array(
        'address', '192.168.0.100',
        'mac-address', '00:00:00:00:00:01'
    ),
    array(
        'address', '192.168.0.101',
        'mac-address', '00:00:00:00:00:02'
    )
);
```

Note that add() returns the IDs of the new entries, so if you're interested in later targeting those entries, you may want to store these.
