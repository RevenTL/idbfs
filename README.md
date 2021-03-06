IDBFS is provides a POSIX-like file system interface for browser-based JavaScript.

* [idbfs.js](https://raw.github.com/js-platform/idbfs/develop/dist/idbfs.js)
* [idbfs.min.js](https://raw.github.com/js-platform/idbfs/develop/dist/idbfs.min.js)

### Getting Started

IDBFS is partly based on the `fs` module from node.js. The API is asynchronous and most methods require the caller to provide a callback function. Errors are passed to callbacks through the first parameter.

To create a new file system or open an existing one, create a new `FileSystem` instance and pass the name of the file system. A new IndexedDB database is created for each file system.

#### Example

````
<script>
  var fs = new IDBFS.FileSystem('local');
  fs.open('/myfile', 'w+', function(err, fd) {
    if (err) throw err;
    fs.close(fd, function(err) {
      if (err) throw err;
      fs.stat('/myfile', function(err, stats) {
        if (err) throw err;
        console.log('stats: ' + JSON.stringify(stats));
      });
    });
  });
</script>
````

As with node.js, there is no guarantee that file system operations will be executed in the order they are invoked. Ensure proper ordering by chaining operations in callbacks.

### API Reference

Callbacks for methods that accept them are non-optional. The first callback parameter is reserved for passing errors. It will be `undefined` if no errors occurred and should always be checked.

#### IDBFS.FileSystem(name, flags)

File system constructor, invoked to open an existing file system or create a new one. Accepts a name and optional flags. Use `'FORMAT'` to force IDBFS for format the file system.

#### fs.stat(path, callback)

Asynchronous stat(2). Callback gets `(error, stats)`, where `stats` is an object like

        {
          node: <string> // internal node id (unique)
          dev: <string> // file system name
          size: <number> // file size in bytes
          nlinks: <number> // number of links
          atime: <number> // last access time
          mtime: <number> // last modified time
          ctime: <number> // creation time
          type: <string> // file type (FILE, DIRECTORY, ...)
        }

#### fs.fstat(fd, callback)

Asynchronous stat(2). Callback gets `(error, stats)`. See `fs.stat`.

#### fs.link(oldpath, newpath, callback)

Asynchronous link(2). Callback gets no additional agruments.

#### fs.unlink(path, callback)

Asynchronous unlink(2). Callback gets no additional agruments.

#### fs.rmdir(path, callback)

Asynchronous rmdir(2). Callback gets no additional agruments.

#### fs.mkdir(path, callback)

Asynchronous mkdir(2). Callback gets no additional agruments.

#### fs.close(fd, callback)

Asynchronous close(2). Callback gets no additional agruments.

#### fs.open(path, flags, callback)

Asynchronous open(2). Flags can be

  * `'r'`: Open file for reading. An exception occurs if the file does not exist.
  * `'r+'`: Open file for reading and writing. An exception occurs if the file does not exist.
  * `'w'`: Open file for writing. The file is created (if it does not exist) or truncated (if it exists).
  * `'w+'`: Open file for reading and writing. The file is created (if it does not exist) or truncated (if it exists).
  * `'a'`: Open file for appending. The file is created if it does not exist.
  * `'a+'`: Open file for reading and appending. The file is created if it does not exist.

Callback gets `(error, fd)`, where `fd` is the file descriptor.

Unlike node.js, IDBFS does not accept the optional `mode` parameter since it doesn't yet implement file permissions.

#### fs.write(fd, buffer, offset, length, position, callback)

Write bytes from `buffer` to the file specified by `fd`, where `offset` and `length` describe the part of the buffer to be written. The `position` refers to the offset from the beginning of the file where this data should be written. If `position` is `null`, the data will be written at the current position. See pwrite(2).

The callback gets `(error, nbytes)`, where `nbytes` is the number of bytes written.

#### fs.read(fd, buffer, offset, length, position, callback)

Read bytes from the file specified by `fd` into `buffer`, where `offset` and `length` describe the part of the buffer to be used. The `position` refers to the offset from the beginning of the file where this data should be read. If `position` is `null`, the data will be written at the current position. See pread(2).

The callback gets `(error, nbytes)`, where `nbytes` is the number of bytes read.

#### fs.lseek(fd, offset, whence, callback)

Asynchronous lseek(2), where `whence` can be `SET`, `CUR`, or `END`. Callback gets `(error, pos)`, where `pos` is the resulting offset, in bytes, from the beginning of the file.

#### fs.readdir(path, callback)

Asynchronous readdir(3). Reads the contents of a directory. Callback gets `(error, files)`, where `files` is an array containing the names of each file in the directory, excluding `.` and `..`.