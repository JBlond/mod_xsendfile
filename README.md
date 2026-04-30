# mod_xsendfile for Apache 2

## Overview

mod_xsendfile is a small Apache 2 module that processes X-SENDFILE headers registered by the original output handler.

If it encounters the presence of such header it will discard all output and send the file specified by that header instead using Apache internals including all optimizations like caching-headers and sendfile or mmap if configured.

It is useful for processing script-output of e.g. php, perl or any cgi.

## Useful?

Yep, it is useful.

- Some applications require checking for special privileges.
- Other have to lookup values first (e.g.. from a DB) in order to correctly process a download request.
- Or store values (download-counters come into mind).
- etc.

### Benefits

- Uses apache internals
- Optimal delivery through sendfile and mmap (if available).
- Sets correct cache headers such as Etag and If-Modified-Since as if the file was statically served.
- Processes cache headers such as If-None-Match or If-Modified-Since.
- Support for ranges.

## Installation

1. Grab the source.
2. Compile and install `apxs -cia mod_xsendfile.c`
3. Restart apache
4. That's all.

## Configuration

### XSendFile

| Description| Enables or disables header processing             |
|------------|---------------------------------------------------|
| Syntax     | XSendFile on\|off                                 |
| Default    | XSendFile off                                     |
| Context    | server config, virtual host, directory, .htaccess |

Setting `XSendFile on` will enable processing.

The file specified in X-SENDFILE header will be sent instead of the handler output.

If the response lacks the X-SENDFILE header nothing is done.

### XSendFileIgnoreEtag

| Description| Ignore script provided Etag headers               |
|------------|---------------------------------------------------|
| Syntax     | XSendFileIgnoreEtag on\|off                       |
| Default    | XSendFileIgnoreEtag off                           |
| Context    | server config, virtual host, directory, .htaccess |

Setting `XSendFileIgnoreEtag on` will ignore all ETag headers the original output handler may have set.
This is helpful for applications that will generate such headers even for empty content.

### XSendFileIgnoreLastModified

| Description| Ignore script provided LastModified headers       |
|------------|---------------------------------------------------|
| Syntax     | XSendFileIgnoreLastModified on\|off               |
| Default    | XSendFileIgnoreLastModified off                   |
| Context    | server config, virtual host, directory, .htaccess |

Setting `XSendFileIgnoreLastModified on` will ignore all Last-Modified headers the original output handler may have set.
This is helpful for applications that will generate such headers even for empty content.

### XSendFileUnsetContentEncoding

| Description| Unsets the Content-Encoding header                |
|------------|---------------------------------------------------|
| Syntax     | XSendFileUnsetContentEncoding on\|off             |
| Default    | XSendFileIgnoreLastModified on                    |
| Context    | server config, virtual host, directory, .htaccess |

Setting `XSendFileUnsetContentEncoding on` Unsets the Content-Encoding header.
The Content-Encoding header - if present - will be dropped, as the module cannot know if it was set by intention of the programmer or the handler. E.g. php with output compression enabled will set this header, but the replacement file send via mod_xsendfile is most likely not compressed.​

### XSendFilePath

| Description| White-list more paths                  |
|------------|----------------------------------------|
| Syntax     | XSendFilePath absolute path            |
| Default    | None                                   |
| Context    | server config, virtual host, directory |

`XSendFilePath` allow you to add additional paths to some kind of white list. All files within these paths are allowed to get served through mod_xsendfile.

Provide an absolute path as Parameter to this directive.

You may provide more than one path.

#### Remarks - Relative paths

The current working directory (if it can be determined) will be always checked first.

If you provide relative paths via the X-SendFile header, then all whitelist items will be checked until a seamingly valid combination is found, i.e. the result is within the bounds of the whitelist item; it isn't checked at this point if the path in question actually exists.
Considering you whitelisted `/tmp/pool` and `/tmp/pool2` and your script working directory is `/var/www`.

##### X-SendFile: file

`/var/www/file` - Within bounds of `/var/www`, OK

##### X-SendFile: ../pool2/file

`/var/www/../pool2/file` = `/var/pool2/file` - Not within bounds of `/var/www`
`/tmp/pool/../pool2/file` = `/tmp/pool2/file` - Not within bounds of `/tmp/pool`
`/tmp/pool2/../pool2/file` = `/tmp/pool2/file` - Within bounds of `/tmp/pool2`, OK

You still can only access paths that are whitelisted. However you have might expect a different behavior here, hence the documentation.

#### Remarks - Inheritance

The white list "inherits" entries from higher level configuration.

```xml
XSendFilePath /tmp
<VirtualHost *>
    ServerName someserver
    XSendFilePath /home/userxyz
</VirtualHost>
<VirtualHost *>
    ServerName anotherserver
    XSendFilePath /var/www/somesite/
    <Directory /var/www/somesite/fastcgis>
        XSendFilePath /var/www/shared
    </Directory>
</VirtualHost>
```

Above example will give:

```text
    *
        /tmp
    someserver
        /tmp
        /home/userxyz
    another
        /tmp
        /var/www/somesite
        /var/www/shared (for scripts* located in /var/www/somesite/fastcgis)
```

*) Scripts, in this context, mean the actual script-starters. E.g. PHP as a handler will use the .php itself, while in CGI mode refers to the starter.

Windows users should take care to include the drive letter to those paths as well. Tests show that it has to be in upper-case.

### Example

.htaccess

```htaccess
<Files out.php>
    XSendFile on
</Files>
```

out.php

```php
<?php
...
if ($user->isLoggedIn())
{
    header("X-Sendfile: $path_to_somefile");
    header("Content-Type: application/octet-stream");
    header("Content-Disposition: attachment; filename=\"$somefile\"");
    exit;
}
?>
<h1>Permission denied</h1>
<p>Login first!</p>
```

## Limitations / Issues / Security considerations

- The Content-Encoding header - if present - will be dropped, as the module cannot know if it was set by intention of the programmer or the handler. E.g. php with output compression enabled will set this header, but the replacement file send via mod_xsendfile is most likely not compressed.
- The header key (X-SENDFILE) is not case-sensitive.
- X-Sendfile will also send files that are otherwise protected (e.g. Deny from all). This includes .htaccess and such!
... as long as one has regular file access permissions defined in the file system.
But, on the other hand, how is this different from the usual scripting?

## Credits

The idea comes from [lighttpd](https://www.lighttpd.net/) - A fast web server with minimal memory footprint.

The module itself was inspired by many other Apache2 modules such as mod_rewrite, mod_headers and obviously core.c.

## License

Copyright 2006-2010 by Nils Maier

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License.

You may obtain a copy of the License at

<http://www.apache.org/licenses/LICENSE-2.0>

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.

See the License for the specific language governing permissions and limitations under the License.

## Changes

[Changelog](CHANGELOG.md)
