#Console
The PHAR file is also an executable, that is a console. Running the PHAR file with any arguments will run the console. Also, installing PEAR2_Net_RouterOS with Pyrus, PEAR or Composer will give you an executable in your bin folder, called "roscon", which is that same console.

You can use the console to quickly test for connectivity, login and API protocol related issues, making sure that certain errors are ultimately due to RouterOS and/or configuration, and not due to a bug in PEAR2_Net_RouterOS or your code.

(Strictly speaking, it's more of a [REPL](http://en.wikipedia.org/wiki/Read%E2%80%93eval%E2%80%93print_loop) shell)

## Arguments
You can get a list of all arguments by either running the "roscon" executable without arguments, or running the PHAR with the "--help" argument. You should see the following:

```
RouterOS API console.

Usage:
  roscon.php [options] hostname [username] [password]

Options:
  -p portNum, --port=portNum  Port to connect to. Default is either 8728 or
                              8729, depending on whether an encryption is
                              specified.
  --cTimeout=conTime          Time in seconds to wait for the connection to
                              be estabilished. If "--timeout" is specified,
                              its value will be used when this option is
                              not specified.
                              Defaults to PHP's default_socket_timeout ini
                              option.
  --enc=crypto                Encryption to use, if at all. Currently,
                              RouterOS supports only "TLS".
                              (Default: "")
  --ca=caPath                 Optional path to a file or folder to use for
                              certification authority, when using
                              encryption. Ignored when not using encryption
                              or using ADH cipher.
  -t time, --timeout=time     Time in seconds to wait when receiving. If
                              this time passes without data awaiting,
                              control is passed back for further input.
                              (Default: 3)
  -v, --verbose               Turn on verbose output.
  --colors=isColored          Choose whether to color output (requires
                              PEAR2_Console_Color). Possible values:
                              "auto" - color is always enabled, except on
                              Windows, where ANSICON must be installed
                              (detected via the ANSICON_VER environment
                              variable).
                              "yes"  - force colored output.
                              "no"   - force no coloring of output.
                              (Default: "auto")
  -w size, --width=size       Width of console screen. Used in verbose mode
                              to wrap output in this length.
                              (Default: 80)
  --command-mode=commandMode  Mode to send commands in. Can be one of:
                              "w" - send every word as soon as it is
                              entered
                              "s" - wait for a sentence to be formed, and
                              send all its words then
                              "e" - wait for an empty sentence, and send
                              all previous sentences then. You can send an
                              empty sentence by sending two consecutive
                              empty words.
                              (Default: "s")
  --reply-mode=replyMode      Mode to get replies in. Can be one of:
                              "w" - after every send, try to get a word
                              "s" - after every send, try to get a sentence
                              "e" - after every send, try to get all
                              sentences until a timeout.
                              (Default: "s")
  -m, --multiline             Turn on multiline mode. Without this mode,
                              every line of input is considered a word.
                              With it, every line is a line within the
                              word, and the end of the word is marked
                              instead by an "end of text" character as the
                              only character on a line. To write out such a
                              character, you can use ALT+Numpad3. If you
                              want to write this character as part of the
                              word, you can write two such characters on a
                              line.
  -h, --help                  show this help message and exit
  --version                   show the program version and exit

Arguments:
  hostname  Hostname of the RouterOS to connect to.
  username  Username to log in with. If left empty, no login will be
            performed.
  password  Password to log in with.
```

## Example command lines
1. Connecting to RouterOS at 192.168.0.1 without credentials over the default API port (8728):

    ```sh
    roscon "192.168.0.1"
    ```
    this mode allows you to diagnose issues during the login procedure itself or with commands that may otherwise be issued anonymously.

1. Same as above, but with username "admin" and no password (as all other examples in this wiki):

    ```sh
    roscon "192.168.0.1" "admin"
    ```
2. Connecting to RouterOS at 192.168.0.1 with the username "admin" and the password "password" over the default API port (8728):

    ```sh
    roscon "192.168.0.1" "admin" "password"
    ```
3. Same as above, but on port 80 instead of port 8728:

    ```sh
    roscon -p 80 "192.168.0.1" "admin" "password"
    ```
## Flow
As mentioned in the beginning, the console is a REPL shell, and as such, it follows that flow. In other words, after a connection is established (and perhaps you're logged in):

1. You write a word.
2. At some point (defined by the --command-mode option; By default, as soon as you form an API sentence, i.e. when you write an empty word), no more reading is done, and all collected input is sent to RouterOS. The console goes back to step 1 if that point is not yet reached.
3. The console waits for data up to the time defined by --timeout, and once a word is received or the time is up, it prints on screen what's been received.
4. Up to a certain point (defined by the --reply-mode option), the console keeps repeating step 3.
5. After reading is done, if the connection is still alive, things go back to step 1. Otherwise, connection is terminated, and the console exits.