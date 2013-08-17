# exec()
## Basic run
The Util class allows you to run a RouterOS script right from within PHP using the exec() method. Note that the script is run relative to the menu you're at, so you can move from one to the other with ease.

This is particularly useful when you need to execute commands that are either unavailable from the API protocol, or are otherwise buggy.

For example:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/ip arp');

$util->exec('
add address=192.168.0.100 mac-address=00:00:00:00:00:01
add address=192.168.0.101 mac-address=00:00:00:00:00:02
/tool
fetch url="http://example.com/?name=customer_1"
fetch url="http://example.com/?name=customer_2"
');
```

## Supplying arguments
Running a RouterOS script from PHP with a literal source wouldn't be much useful if you didn't also have some variables in there that change based on some sort of user input. On the other hand, simply writing them out as part of the source can lead to code injection, which can be as catastrophic for RouterOS as an SQL injection is for an SQL database.

The exec() method also accepts a second argument where you can supply an associative array of values that will be made available to the script as local variables. The array key will be the name of the variable, and the array's value will be sanitized so as to not cause any sort of code injection.

The previous example could be written as:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/ip arp');

$source = '
add address="192.168.0.$ip" mac-address="00:00:00:00:00:$mac" comment=$name
/tool
fetch url="http://example.com/?name=$name"
';
$util->exec(
    $source,
    array(
        'ip' => 100,
        'mac' => '01',
        'name' => 'customer_1'
    )
);
$util->exec(
    $source,
    array(
        'ip' => 101,
        'mac' => '02',
        'name' => 'customer_2'
    )
);
```

Note also that PEAR2_Net_RouterOS is smart enough to convert native PHP types into equivalent RouterOS types. While this may not sound much interesting and/or useful for scalar values (string, boolean, (int/double)/number), consider fancier types like arrays, which are processed recursively for all of their values, and made available in the script as an array you can then ":pick" apart. Even more interestingly - PHP's DateInterval objects are converted into a RouterOS "time" typed value, and a PHP DateTime object is converted into a "time" object relative to the UNIX epoch time.

**IMPORANT NOTE: Watch out for PHP's double quotes and HEREDOC notation. They both accept PHP variables, and since both they and RouterOS variables are accessed with "$" in front of the name, you may end up writing a literal value when you meant to address a supplied variable. To be certain, make sure you're using either single quotes or NEWDOC notation.**

## Restricting script access
OK, so let's say that you use the above approach, and yet you're still somewhat paranoid about the values being escaped properly, or maybe your script is depending on untrusted data that's not supplied by PHP itself (e.g. a fetch of a script that's then being "/import"-ed). How do you make sure the script doesn't do anything too damaging? Lucky, MikroTik already have the solution, and PEAR2_Net_RouterOS supports it - you can simply set a policy on the script, which would contain the minimum permissions needed for it to work properly. In the case of a code injection or otherwise "normal" execution, the script will fail as soon as it tried to violate its granted permissions.

You can set permissions as the third argument of exec(). Without this argument, full permissions are assumed. You can see the acceptable values by typing
```sh
/system script add policy=?
```
from a terminal.

Here's one example:
```php
<?php
use PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

$util = new RouterOS\Util($client = new RouterOS\Client('192.168.0.1', 'admin'));
$util->changeMenu('/tool');

$source = '
fetch url="http://example.com/$db" keep-result=yes dst-path=$db
# Give the script time to be written onto disk
:delay 2
/import file=$db
';
$util->exec(
    $source,
    array(
        'db' => 'geoip.rsc'
    ),
    'read,write'
);
```

With the policy being specified as "read,write", the script could do lots of damage, but at least it won't be able to read/write sensitive information like passwords for hotspots, wifi, and RouterOS (because that requires the "sensitive" permission)... and it also won't be able to forcefully reboot your router (because that requires the "reboot" permission), which combined with the write permission can prove fatal if a startup script is made to again reboot the router... And it can't sniff the rest of your network (which requires the "sniff" permission). With all of those restrictions, you'll be able to easily recover form any damage that the remote script could possibly do, assuming you have backup of course, and no one else would know.