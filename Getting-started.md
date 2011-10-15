# Getting started
## Introduction
RouterOS is the flag product of the company [MikroTik](http://mikrotik.com) and is a powerful router software. One of its many abilities is to allow control over it via an API. This package provides a client for [that API](http://wiki.mikrotik.com/wiki/Manual:API), in turn allowing you to use PHP to control RouterOS hosts.
## Requirements
* PHP 5.3.0 or later. 
* A host with RouterOS v3 or later. 
* Enabled API service on the RouterOS host.
* The PCRE extension (bundled and enabled by default in PHP)
* The iconv extension (optional; bundled and enabled by default in PHP; required only if you want to use automatic charset convertion)
* [PEAR2_Net_Transmitter](http://boenrobot.github.com/PEAR2_Net_Transmitter/) (bundled with the PEAR2_Net_RouterOS archive; installed automatically by Pyrus)

The API service is disabled by default. To enable it, you need to execute 
```sh
/ip service set numbers="api" address="0.0.0.0/0" disabled="no"
```
at a RouterOS terminal. The "address" argument in the command above allows you to limit access to this service only to certain IP addresses. For security's sake, it's better that you limit connections only to the IP address with which PHP will access RouterOS.
## Installation
### Installation with [Pyrus/PEAR2](http://pear2.php.net/)
Assuming you have [installed Pyrus](http://pear.php.net/manual/en/installationpyrus.introduction.php), you can install PEAR2_Net_RouterOS from the pear2.php.net channel with

```sh
pyrus install PEAR2_Net_RouterOS-alpha
```

You might notice that the version number of PEAR2_Net_RouterOS suggests it's a beta, and yet we use "-alpha" in the command above. Well, yes, PEAR2_Net_RouterOS is a beta, but it has a dependency to another package - PEAR2_Net_Transmitter - which is an alpha. To avoid getting errors, you need to use "-alpha" until that package reaches a beta.

If you've decided to not use the pear2.php.net channel, but instead install directly from the archive distributed at the project page, you can use
```sh
pyrus install /path/to/downloaded/archive.tgz
```
If you haven't installed PEAR_Net_Transmitter previously, Pyrus will install the one at pear2.php.net (not the bundled version, although the two are equivalent at the time of this writing).

### Installation with [PEAR](http://pear.php.net/)
Like most PEAR2 packages, PEAR2_Net_RouterOS is compatible with the [PEAR installer](http://pear.php.net/manual/en/installation.getting.php). However, you have to first discover the pear2 channel with
```sh
pear channel-discover pear2.php.net
```
and only then install PEAR2_Net_RouterOS with
```sh
pear install pear2/PEAR2_Net_RouterOS-alpha
```
### Manual installation
The archive includes a version of PEAR2_Net_Transmitter, so if you've downloaded the archive, instead of using the PEAR2 installer, you can just extract the contents of the "src" folder wherever you like. To emulate the PEAR2 installer, you can place the files in a folder that's within your include_path.

If you're installing from the respository, you'll have to also [install PEAR2_Net_Transmitter](/pear2/PEAR2_Net_Transmitter/wiki/Getting-started) in one way or another before you can use this package.
## Usage
To use this package in a PHP file, you could manually include every required class, but to save yourself some hassle, it's a better idea that you just include the file Autoload.php, like:
```php
<?php
include_once 'PEAR2/Net/RouterOS/Autoload.php';
//Use any PEAR2_Net_RouterOS class
?>
```

Like every other PEAR2 package, PEAR2_Net_RouterOS uses namespaces - a feature introduced in PHP 5.3 - for its organization. Among other things, this means that instead of you having to write long class names, you can just declare at the top that you'll be using this namespace, and then just write shorter class names. The possible approaches are as follows:

* Using a fully qualified class name
```php
<?php
include_once 'PEAR2/Net/RouterOS/Autoload.php';
$client = new \PEAR2\Net\RouterOS\Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring the PEAR2\Net\RouterOS as your default namespace 
```php
<?php
namespace PEAR2\Net\RouterOS;
include_once 'PEAR2/Net/RouterOS/Autoload.php';
$client = new Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring the PEAR2\Net\RouterOS as an aliased namespace 
```php
<?php
use PEAR2\Net\RouterOS as Ros;
include_once 'PEAR2/Net/RouterOS/Autoload.php';
$client = new Ros\Client('example.com', 'admin');
// Use the client here
?>
```
* Declaring an alias of each class you intend to use directly. 
```php
<?php
use PEAR2\Net\RouterOS\Client as Ros;
include_once 'PEAR2/Net/RouterOS/Autoload.php';
$client = new Ros('example.com', 'admin');
// Use the client here
?>
```

Note that namespace declarations must appear before any includes. They must in fact be the first thing in a PHP file. The rest of the examples in this documentation will be setting PEAR2\Net\RouterOS as the default namespace.
