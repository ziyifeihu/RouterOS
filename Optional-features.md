# Optional features

## Automatic charset conversion
Let's say you're not a native English speaker, and that your native language doesn't use just the latin alphabet (e.g. Portuguese) or doesn't use it at all (e.g. Bulgarian, Georgian, etc.).

You were probably tempted to write in your native language from within Winbox. You'll see you can successfully do that, and read everything from within Winbox later. But if you do some more extensive testing, you'll see that your non-latin text is only readable from computers with the same regional settings. Furthermore, if you were to view the text at the router itself or with the API, you'll see the non-latin text as gibberish.

All of this is because the charsets are different in all of these environments - Winbox uses your regional settings' charset, the terminal shows non ANSI characters with their code points, and the API gets the raw data (without any charset directions).

PEAR2_Net_RouterOS allows you to tell it the charset your content is stored in, and the charset that your web server content is in. After you specify those, the conversion between those two is done automatically.

But what charset pair to use? Unfortunately, that may be difficult to answer, making this the hardest obstacle for this feature.

On UNIX, you can use ```nl_langinfo(CODESET);``` to get the charset of your current locale settings, but you can't do the same on Windows, and there's no easily accessible place in the Windows GUI where you can view your current charset either.

Your best bet on Windows is to create an empty HTML file, without any meta tags, and run it locally, without an HTTP server, using IE. Right click, and go to "Encoding", and you should see your regional settings' charset being the one selected with a radio button. You may not see the exact charset though, but you'll at least see its family (ISO, Windows, KOIR, etc.), which should narrow down your search to one of the [common character encodings](http://en.wikipedia.org/wiki/Character_encoding#Common_character_encodings).

Let's say that we find our charset pair. In the example below, we'll assume that what the investigation (using the above instructions) revealed was "windows-1251", which is what we would specify as the charset our content is stored in. If you're experienced enough developer, you're probably writing your application using "UTF-8", so that's what we'll use as our web server's charset. This is done like so:

```php
<?php
namespace PEAR2\Net\RouterOS;
require_once 'PEAR2/Autoload.php';

//Ensuring our text on screen in UTF-8 too.
//Note that for this to work, the PHP file itself must also be encoded with UTF-8.
//If you're using Notepad, you can ensure that from the "Encoding" drop down at the "Save As.." menu.
header('Content-Type: text/html;charset=UTF-8');

$client = new Client('192.168.0.1', 'admin');

//Here's where we specify the charset pair.
$client->setCharset(
    array(
        Communicator::CHARSET_REMOTE => 'windows-1251',
        Communicator::CHARSET_LOCAL  => 'UTF-8'
    )
);

$client->sendSync(new Request('/queue/simple/add name=Йес'));
//"Йес" should appear in the exact same way in Winbox now,
//assuming your Windows' regional settings use this charset.

//Let's assume you already have another queue list entry with the name "ягода"
echo $client->sendSync(
    new Request('/queue/simple/print', Query::where('name', 'ягода'))
)->getArgument('name');
//Shoud output "ягода" in the exact same fashion as you see it here, and in Winbox.
```

## Persistent connections
PEAR2_Net_RouterOS offers persistent connections, which in essence provides the exact same thing as the [MySQLi equivalent](http://php.net/manual/en/mysqli.persistconns.php) - all of your PHP requests will share the same TCP connection to the router.

This may reduce the required bandwidth between your web server and your router, and should also take off some of the load from the router.

It's important to note that unlike the MySQLi equivalent, persistent connections come with a penalty of their own. A penalty on the web server. Because multiple requests can come in at the same time, and they're all "meshed", PEAR2_Net_RouterOS needs to marshal the different requests and responses to each Client instance, which is done measurably slower in PHP than in C.

If you have a powerful enough web server (as in, you're happy with the web server's performance of non-persistent connections), and a not-so-powerful router (as is typically the case), this trade off is probably well worth it.

You can create a persistent connection by specifying ```true``` at the 5th argument of the Client constructor (the 4th being the API port).

__There is no difference in terms of "features" between persistent and non persistent connections. If you find a difference, you've found a bug. The only difference is in performance.__