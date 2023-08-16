enable Debugging in XAMPP `php.ini`:

```ini
zend_extension=php_xdebug.dll
xdebug.mode = debug
xdebug.start_with_request = yes
```

## PHP Debug Adapter for Visual Studio Code
## Installation

Install the extension: Press `F1`, type `ext install php-debug`.

This extension is a debug adapter between VS Code and [Xdebug](https://xdebug.org/) by Derick Rethans. Xdebug is a PHP extension (a `.so` file on Linux and a `.dll` on Windows) that needs to be installed on your server.

1. [Install Xdebug](https://xdebug.org/docs/install) **_I highly recommend you make a simple `test.php` file, put a `phpinfo();` statement in there, then copy the output and paste it into the [Xdebug installation wizard](https://xdebug.org/wizard.php). It will analyze it and give you tailored installation instructions for your environment._** In short:
    
    - On Windows: [Download](https://xdebug.org/download.php) the appropriate precompiled DLL for your PHP version, architecture (64/32 Bit), thread safety (TS/NTS) and Visual Studio compiler version and place it in your PHP extension folder.
    - On Linux: Either download the source code as a tarball or [clone it with git](https://xdebug.org/docs/install#source), then [compile it](https://xdebug.org/docs/install#compile). Or see if your distribution already offers prebuilt packages.
2. [Configure PHP to use Xdebug](https://xdebug.org/docs/install#configure-php) by adding `zend_extension=path/to/xdebug` to your php.ini. The path of your php.ini is shown in your `phpinfo()` output under "Loaded Configuration File".
    
3. Enable remote debugging in your `php.ini`:
    
```ini
zend_extension=php_xdebug.dll
xdebug.mode = debug
xdebug.start_with_request = yes
```

4. If you are doing web development, don't forget to restart your webserver to reload the settings.
    
5. Verify your installation by checking your `phpinfo()` output for an Xdebug section.
    

### VS Code Configuration

In your project, go to the debugger and hit the little gear icon and choose _PHP_. A new launch configuration will be created for you with three configurations:

- **Listen for Xdebug** This setting will simply start listening on the specified port (by default 9003) for Xdebug. If you configured Xdebug like recommended above, every time you make a request with a browser to your webserver or launch a CLI script Xdebug will connect and you can stop on breakpoints, exceptions etc.
- **Launch currently open script** This setting is an example of CLI debugging. It will launch the currently opened script as a CLI, show all stdout/stderr output in the debug console and end the debug session once the script exits.
- **Launch Built-in web server** This configuration starts the PHP built-in web server on a random port and opens the browser with the `serverReadyAction` directive. The port is random (localhost:0) but can be changed to a desired fixed port (ex: localhost:8080). If a router script is needed, add it with `program` directive. Additional PHP/Xdebug directives trigger debugging on every page load.

## Handle Attachements in Modules
modulspezifische Handler sind in `lib.php` vereinbart.

```php
//filelib/file_pluginfile
$filefunction: "mod_glossary_pluginfile"
$filefunctionold: "glossary_pluginfile"
```


### Install guide for macos
[Installing and using Xdebug with Homebrew, Valet, and VS Code in 2021 | Blog (daronspence.com)](https://www.daronspence.com/2021/06/02/installing-and-using-xdebug-with-homebrew-valet-and-vs-code-in-2021/)
