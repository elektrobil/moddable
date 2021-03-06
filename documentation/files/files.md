# Files

Copyright 2017-2018 Moddable Tech, Inc.<BR>
Revised: November 6, 2018

**Warning**: These notes are preliminary. Omissions and errors are likely. If you encounter problems, please ask for assistance.

## Table of Contents

* [Notes on SPIFFS file system](#spiffs)
* [File](#file)
* [File Iterator](#file-iterator)
* [File System](#file-system)
* [Zip](#zip)
* [Resource](#resource)
* [Preference](#preference)
* [Flash](#flash)

<a id="spiffs"></a>
## Notes on SPIFFS file system

On embedded systems, the `File` class is implemented using the [SPIFFS](https://github.com/pellepl/spiffs) file system.

SPIFFS is a flat file system, meaning that there are no directories and all files are at the root.

The SPIFFS file system requires some additional memory. Including SPIFFS in the build increase RAM use by about 500 bytes. Using the SPIFFS file system requires about another 3 KB of RAM. To minimize the memory impact, the `File` class only instantiates the SPIFFS file system when necessary -- when a file is open and when a file is deleted. The SPIFFS file system is automatically closed when not in use.

If the SPIFFS file system has not been initialized, it is formatted with the `SPIFFS_format` API when first used. Initialization takes up to one minute.

<a id="file"></a>
## class File

- **Source code:** [file](../../modules/files/file)
- **Relevant Examples:** [files](../../examples/files/files/)

The `File` class provides access to files.

```js
import {File} from "file";
```

### `constructor(path [, write])`

The `File` constructor opens a file for read or write. The optional write argument selects the mode. The default value for write is `false`. When opened, the file position is 0.

If the file does not exist, an exception is thrown when opening in read mode. When opening in write mode, a new file is created if it does not already exist.

```js
let file = new File("preferences.json");
```

***

### `read(type [, count])`

The `read` function reads from the current position. The data is read into a `String` or `ArrayBuffer` based on the value of the `type` argument. The `count` argument is the number of bytes to read. The default value of `count` is the number of bytes between the current `position` and the file `length`.

```js
let file = new File("preferences.json");
let preferences = JSON.parse(file.read(String));
file.close();
```
***

### `write(value [, ...values])`

The `write` function writes one or more values to the file starting at the current `position`. The values may be either a `String` or `ArrayBuffer`.

```js
File.delete("preferences.json");
let file = new File("preferences.json", true);
file.write(JSON.stringify(preferences));
file.close();
```

***

### `length` property

The `length` property is a number indicating the number of bytes in the file. It is read-only.

***

### `position` property

The `position` property is a number indicating the byte offset into the file, for the next read or write operation.

***

### `static delete(path)`

The static `delete` function removes the file at the specified path.

```js
File.delete("test.txt");
```

***

### `static exists(path)`

The static `exists` function returns a boolean indicating whether a file exists at the specified path.

```js
let exists = File.exists("test.txt");
```

***

### `static rename(from, to)`

The static `rename` function renames the file specified by the `from` argument to the name specified by the `to` argument.

```js
File.rename("test.txt", "betterName.txt");
```

***

### Example: Get file size

This example opens a file in read-only mode to retrieve the file's length. If the file does not exist, it is not created and an exception is thrown.

```js
let file = new File("test.txt");
trace(`File length ${file.length}\n`);
file.close();
```

***

### Example: Read file as String

This example retrieves the entire content of a file into a `String`. If there is insufficient memory available to store the string or the file does not exist, an exception is thrown.

```js
let file = new File("test.txt");
trace(file.read(String));
file.close();
```

***

### Example: Read file into ArrayBuffers

This example reads a file into one or more `ArrayBuffer` objects. The final `ArrayBuffer` is smaller than 1024 when the file size is not an integer multiple of 1024.

```js
let file = new File("test.txt");
while (file.position < file.length) {
	let buffer = file.read(ArrayBuffer, 1024);
}
file.close();
```

***

### Example: Write string to file

This example deletes a file, opens it for write (which creates a new empty file), and then writes two `String` values to the file. The script then moves the read/write position to the start of the file, and reads the entire file contents into a single `String`, which is traced to the console.

```js
File.delete("test.txt");

let file = new File("test.txt", true);
file.write("This is a test.\n");
file.write("This is the end of the test.\n");

file.position = 0;
let content = file.read(String);
trace(content);

file.close();
```

***

<a id="file-iterator"></a>
## class File Iterator

- **Source code:** [file](../../modules/files/file)
- **Relevant Examples:** [files](../../examples/files/files/)

The File `Iterator` class enumerates the files and subdirectories in a directory. 

```js
import {Iterator} from "file";
```

> **Note**: Because the SPIFFS file system is a flat file system,  no subdirectories are returned on devices that use it.

### `constructor(path)`

The constructor takes as its sole argument the path of the directory to iterate over.

> **Note**: For the SPIFFS flat file system, always pass "/" for the `path` argument to the constructor. This ensures compatibility with file systems that implement directories.

```js
let root = new Iterator("/");
```

***

### `next()`

The `next` function is called repeatedly, each time retrieving information about one file. When all files have been returned, the `next` function returns `undefined`. For each file and subdirectory, next returns an object. The object always contains a `name` property with the file name. If the object contains a `length` property, it references a file and the `length` property is the size of the file in bytes. If the `length` property is absent, it references a directory.

```js
let item = root.next();
```

***

### Example: List contents of a directory

This example lists all the files and subdirectories in a directory.

```js
let root = new Iterator("/");
let item;
while (item = root.next()) {
	if (undefined === item.length)
		trace(`Directory: ${item.name}\n`);
	else
		trace(`File: ${item.name}, ${item.length} bytes\n`);
}
```

The iterator's `next` function returns an object.  If the object has a `length` property, it is a file; if the `length` property is undefined, it is a directory.

***

<a id="file-system"></a>
## class File System

- **Source code:** [file](../../modules/files/file)

The File `System` class provides information about the file system.

```js
import {System} from "file";
```

### `static config()`

The `config` function returns a dictionary with information about the file system. At this time, the dictionary has a single property, `maxPathLength`, which indicates the length of the longest file path in bytes.

```js
let maxPathLength = System.config().maxPathLength;
```

***

### `static info()`

The `info` function returns a dictionary with information about the free and used space in the file system. The `used` property of the dictionary gives the number of bytes in use and the `total` property indicates the maximum capacity of the file system in bytes.

```js
let info = System.info();
let percentFree = 1 - (info.used / info.total);
```

***

<a id="zip"></a>
## class ZIP

- **Source code:** [zip](../../modules/files/zip)
- **Relevant Examples:** [zip](../../examples/files/zip/)

The `ZIP` class implements read-only file system access to the contents of a ZIP file stored in memory. Typically these are stored in flash memory.

The `ZIP` implementation requires all files in the ZIP file to be uncompressed. The default in ZIP files is to compress files, so special steps are necessary to build a compatible ZIP file.

The [`zip`](https://linux.die.net/man/1/zip) command line tool creates uncompressed ZIP files when a compression level of zero is specified. The following command line creates a ZIP file named `test.zip` with the uncompressed contents of the directory `test`.

	zip -0r test.zip test

### `constructor(buffer)`

The `ZIP` constructor instantiates a `ZIP` object to access the contents of the buffer as a read-only file system. The buffer may be either an `ArrayBuffer` or a Host Buffer.

The constructor validates that the buffer contains a ZIP archive, throwing an exception if it does not.

A ZIP archive is stored in memory. If it is ROM, it will be accessed using a Host Buffer, a variant of an `ArrayBuffer`. The host platform software provides the Host Buffer instance through a platform specific mechanism. This example uses the `Resource` constructor to create the Host Buffer.

```js
let buffer = new Resource("test.zip");
let archive = new ZIP(buffer);
```

***

### `file(path)`

The `file` function instantiates an object to access the content of the specified path within the ZIP archive. The returned instance implements the same API as the `File` class.

```js
let file = archive.file("small.txt");
```

***

### `iterate(path)`

The `iterate` function instantiates an object to access the content of the specified directory path within the ZIP archive. The returned instance implements the same API as the Iterator class. Directory paths end with a slash ("`/`") character and, with the exception of the root path, do not begin with a slash.

```js
let root = archive.iterate("/");
```

***

### `map(path)`

The `map` function returns a Host Buffer that references the bytes of the file at the specified path.

***

### Example: Reading a file from ZIP archive

The `ZIP` instance's `file` function provides an instance used to access a file. Though instantiated differently, the ZIP file instance shares the same API with the `File` class.

```js
let file = archive.file("small.txt");
trace(`File size: ${file.length} bytes\n`);
let string = file.read(String);
trace(string);
file.close();
```

***

### Example: List contents of a ZIP archive's directory

The following example iterates the files and directories at the root of the archive. Often the root contains only a single directory.

```js
let root = archive.iterate("/");
let item;
while (item = root.next()) {
    if (undefined == item.length)
        trace(`Directory: ${item.name}\n`);
    else
        trace(`File: ${item.name}, ${item.length} bytes\n`);
}
```

The ZIP iterator expects directory paths to end with a slash ("`/`"). To iterate the contents of a directory named "test" at the root, use the following code:

```js
let iterator = archive.iterate("test/");
```

***

<a id="resource"></a>
## class Resource

- **Source code:** [resource](../../modules/files/resource)
- **Relevant Examples:** [zip](../../examples/files/zip/), many [Commodetto examples](../../examples/commodetto) including [sprite](../../examples/commodetto/sprite) and [text](../../examples/commodetto/text)

The `Resource` class provides access to assets from an application's resource map.

```js
import Resource from "Resource";
```

### `constructor(path)`

The `Resource` constructor takes a single argument, the resource path, and returns an `ArrayBuffer` or Host Buffer containing the resource data.

```js
let resource = new Resource("logo.bmp");
trace(`resource size is ${resource.byteLength}\n`);
```

***

### `static exists(path)`

The static `exists` function returns a boolean indicating whether a resource exists at the specified path.

```js
let path = "test.zip";
if (Resource.exists(path))
	trace(`File ${path} exists\n`);
```

***

### `slice(begin[, end])`

The `slice` function returns a portion of the resource in an `ArrayBuffer`. The default value of `end` is the resource size.

```js
let resource = new Resource("table.dat");
let buffer1 = resource.slice(5);		// Get a buffer starting from offset 5
let buffer2 = resource.slice(0, 10);		// Get a buffer of the first 10 bytes
```

***

<a id="preference"></a>
## class Preference

- **Source code:** [preference](../../modules/files/preference/)
- **Relevant Examples:** [preference](../../examples/files/preference/), [preferences](../../examples/piu/preferences/)

The `Preference` class provides storage of persistent preference storage. Preferences are appropriate for storing small amounts of data that needs to persist between runs of an application.

```js
import Preference from "preference";
```
	
Preferences are grouped by domain. A domain contains one or more keys. Each domain/key pair holds a single value, which is either a `Boolean`, integer (e.g. `Number` with no fractional part), `String` or `ArrayBuffer`.

```js
const domain = "wifi";
let ssid = Preference.get(domain, "ssid");
let password = Preference.get(domain, "psk");
```

Preference values are limited to 63 bytes. Key and domain names are limited to 32 bytes.

On embedded devices the storage space for preferences is limited. The amount depends on the device, but it can be as little as 4 KB. Consequently, applications should take care to keep their  preferences as small as practical.

> **Note**: On embedded devices, preferences are stored in SPI flash which has a limited number of erase cycles. Applications should minimize the number of write operations (set and delete). In practice, this isn't a significant concern. However, an application that updates preferences once per minute, for example, could eventually exceed the available erase cycles for the preference storage area in SPI flash.

### `static set(domain, key, value)`

The static `set` function sets a preference value.

```js
Preference.set("wifi", "ssid", "linksys");
Preference.set("wifi", "password", "admin");
Preference.set("wifi", "channel", 6);
```

***

### `static get(domain, key)`

The static `get` function reads a preference value. If the preference does not exist, `get` returns `undefined`.

```js
let value = Preference.get("settings", "timezone");
if (value !== undefined)
	trace(`timezone ${value}\n`);
```

***

### `static delete(domain, key)`

The static `delete` function removes a preference. If the preference does not exist, no error is thrown.

```js
Preference.delete("wifi", "password");
```

***

### `static keys(domain)`

Returns an array of all keys under the given domain.
 
```js
let wifiKeys = Preference.keys("wifi");
for (let key of wifiKeys)
	trace(`${key}: ${Preference.get("wifi", key)}\n`);
```
 
***

<a id="flash"></a>
## class Flash

This class is experimental and not yet documented.
