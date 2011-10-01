# Getting started
## Introduction
RouterOS is the flag product of the company [MikroTik](http://mikrotik.com) and is a powerful router software. One of its many abilities is to allow control over it via an API. This package provides a client for [that API](http://wiki.mikrotik.com/wiki/Manual:API), in turn allowing you to use PHP to control RouterOS hosts.
## Installation
### Installation with [Pyrus/PEAR2](http://pear2.php.net/)
Assuming you have [installed Pyrus](http://pear.php.net/manual/en/installationpyrus.introduction.php), you can install PEAR2_Net_RouterOS with

`pyrus install PEAR2_Net_RouterOS-alpha`

You might notice that the version number of PEAR2_Net_RouterOS suggests it's a beta, and yet we use "-alpha" in the command above. Well, yes, PEAR2_Net_RouterOS is a beta, but it has a dependency to another package - [PEAR2_Net_Transmitter](https://github.com/boenrobot/PEAR2_Net_Transmitter) - which is an alpha. To avoid getting errors, you need to use "-alpha" until that package reaches a beta.

### Manual installation
The archive includes a version of PEAR2_Net_Transmitter, so if you've downloaded the archive, you can just extract the contents of the "src" folder wherever you like. To emulate the PEAR2 installer, you can place the files in a folder that's within your include_path.

If you're installing from source, you'll have to also [install PEAR2_Net_Transmitter](https://github.com/boenrobot/PEAR2_Net_Transmitter/wiki/Getting-started) in one way or another before you can use this package.