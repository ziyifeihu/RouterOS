# Getting started
## Introduction
RouterOS is the flag product of the company [MikroTik](http://mikrotik.com) and is a powerful router software. One of its many abilities is to allow control over it via an API. This package provides a client for [that API](http://wiki.mikrotik.com/wiki/Manual:API), in turn allowing you to use PHP to control RouterOS hosts.
## Requirements
The requirements to watch out for are:
* PHP 5.3.0 or later. 
* A host with RouterOS v3 or later. 
* Enabled API service on the RouterOS host.

Other requirements are not a problem in most scenarios. For reference, they are:
* The PCRE and SPL extensions (compiled into PHP by default)
* [PEAR2_Net_Transmitter](http://pear2.github.com/Net_Transmitter/) (bundled in the archive; installed automatically by Pyrus)
* [optional] The iconv extension (compiled into PHP by default; required only if you want to use automatic charset convertion)
* [optional] [PEAR2_Cache_SHM](http://pear2.github.com/Cache_SHM/) (bundled in the archive; needed only if you use persistent connections; installed by Pyrus if you pass the "-o" flag at installation)
* [optional] A PSR-0 compliant autoloader (highly recommended; [PEAR2_Autoload](http://pear2.php.net/PEAR2_Autoload) is one such autoloader that is bundled in the archive and installed by Pyrus if you pass the "-o" flag at installation)

The API service in RouterOS is disabled by default. To enable it, you need to execute 
```sh
/ip service set numbers="api" address="0.0.0.0/0" disabled="no"
```
from a RouterOS terminal. The "address" argument in the command above allows you to limit access to this service only to certain IP addresses. For security's sake, it's better that you limit connections only to the IP address(es) with which PHP will access RouterOS.
## Installation
### Installation with [Pyrus/PEAR2](http://pear2.php.net/) (recommended)
Assuming you have [installed Pyrus](http://pear.php.net/manual/en/installationpyrus.introduction.php), you can install PEAR2_Net_RouterOS from the PEAR2 channel with just

```sh
php pyrus.phar install PEAR2_Net_RouterOS-alpha
```

or, if you want to also get the optional dependencies, you can use

```sh
php pyrus.phar install -o PEAR2_Net_RouterOS-alpha
```

You might notice that the version number of PEAR2_Net_RouterOS suggests it's a beta, and yet we use "-alpha" in the command above. Well, yes, PEAR2_Net_RouterOS is a beta, but it has a dependency to another package - PEAR2_Net_Transmitter - which is an alpha. To avoid getting errors, you need to use "-alpha" until that package reaches a beta.

__Note also that this package is a "beta" according to the [PEAR2 version standard](https://wiki.php.net/pear/rfc/pear2_versioning_standard_revision), which essentially means that it works, it's documented, has large code coverage (100% in this case), but future versions of it are still a potential subject to breaking API changes (e.g. the next release could rename methods, add/remove/rename classes, swap arguments, etc.) as long as those changes are still working, documented and covered by tests. In other words, the "beta" does not refer to error prone-ness.__

If you've decided to not use the PEAR2 channel, but instead install directly from the archive distributed at the project page, you can use

```sh
php pyrus.phar install /path/to/downloaded/archive.tgz
```

If you haven't installed PEAR_Net_Transmitter previously, Pyrus will install the one at the PEAR2 channel (not the bundled version, although the two are equivalent at the time of this writing).

### Installation with [PEAR](http://pear.php.net/)
Like most PEAR2 packages, PEAR2_Net_RouterOS is compatible with the [PEAR installer](http://pear.php.net/manual/en/installation.getting.php). However, you have to first discover the PEAR2 channel with

```sh
pear channel-discover pear2.php.net
```

and only then install PEAR2_Net_RouterOS with

```sh
pear install pear2/PEAR2_Net_RouterOS-alpha
```

or

```sh
pear install -a pear2/PEAR2_Net_RouterOS-alpha
```

to install optional dependencies as well.

### Direct PHAR usage
If you download the ".phar" archive, instead of using the PEAR(2) installer, you can just include the archive, and be ready to go, like for example:

```php
<?php
require_once 'PEAR2_Net_RouterOS-1.0.0b3.phar';
//Use any PEAR2_Net_RouterOS class
```

### Manual installation
Instead of using the PEAR(2) installer, you can just download an archive, and extract the contents of the "src" folder wherever you like. To emulate the PEAR(2) installer, you can place the files in a folder that's within your include_path. The packaged archive includes a version of PEAR2_Net_Transmitter, so there's nothing to worry about beyond extracting the archive.

### Installation from the repository (with [Git](http://git-scm.com/) and [Composer](http://getcomposer.org/))
If you want to get the "cutting edge", unpackaged version of PEAR2_Net_RouterOS, you'll need to have Git and Composer. Once you have them, create a folder to place the package in, navigate to it from the command line, and execute the following commands:

```sh
git clone git://github.com/pear2/Net_RouterOS
php composer.phar install
```

PEAR2_Net_Transmitter and PEAR2_Cache_SHM will be placed into a subfolder named "vendor. After that, you can use the autoloader generated by Composer in "vendor/autoload.php" to load classes from the package.

## Usage
If you use the PHAR archive, every class you attempt to use will automatically be auto loaded with the bundled PEAR2_Autoload. In virtually all places of this documentation, the line

```php
include_once 'PEAR2/Autoload.php';
```

can be replaced with

```php
include_once 'PEAR2_Net_RouterOS-1.0.0b3.phar';
```

and then everything should work the same.

With any other method, you need to include any PSR-0 compatible autoloader (the bundled PEAR2_Autoload being just one option), and if necessary, register the folder where the PHP files are located. With PEAR2_Autoload, that is only needed when the files are outside the parent folder of Autoload.php's folder, so the code MAY look like:

```php
<?php
include_once 'PEAR2/Autoload.php';
\PEAR2\Autoload::register('/path/to/your/PEAR2/files');
//Use any PEAR2_Net_RouterOS class
```

Like every other PEAR2 package, PEAR2_Net_RouterOS uses namespaces - a feature introduced in PHP 5.3 - for its organization. Among other things, this means that instead of you having to write long class names, you can just declare at the top that you'll be using this namespace, and then just write shorter class names. The possible approaches are as follows:

* Using a fully qualified class name

```php
<?php
include_once 'PEAR2/Autoload.php';
$client = new \PEAR2\Net\RouterOS\Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring the PEAR2\Net\RouterOS as your default namespace
 
```php
<?php
namespace PEAR2\Net\RouterOS;
include_once 'PEAR2/Autoload.php';
$client = new Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring the PEAR2\Net\RouterOS as an aliased namespace
 
```php
<?php
use PEAR2\Net\RouterOS as Ros;
include_once 'PEAR2/Autoload.php';
$client = new Ros\Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring an alias of each class you intend to use directly.

```php
<?php
use PEAR2\Net\RouterOS\Client as Ros;
include_once 'PEAR2/Autoload.php';
$client = new Ros('example.com', 'admin');
// Use the client here
?>
```

Note that namespace declarations must appear before any includes. They must in fact be the first thing in a PHP file. The rest of the examples in this documentation will be setting PEAR2\Net\RouterOS as the default namespace.

## Troubleshooting
If the package doesn't work, you can download the "phar" file (maybe rename it from ".phar" to ".php"), and run it in your browser or command line.

When you do that, you should see the version of the package, along with some messages indicating if you're missing any of the requirements. If all requirements are present, you'll see suggestions as to what may go wrong when connecting to a RouterOS host, and suggestions on how to fix it.

## Further information
The [rest of this documentation](../wiki) contains more tutorials and examples on how to use this package. If you have trouble doing a certain thing with the RouterOS API, the best place to ask for help is [MikroTik's forum on scripting](http://forum.mikrotik.com/viewforum.php?f=9). If you believe you've found a bug in this package or miss a certain feature, don't hesitate to [submit an issue](../issues/new) for it.