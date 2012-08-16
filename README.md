# VFS Local

[![Build Status](https://secure.travis-ci.org/c9/vfs-local.png?branch=master)](http://travis-ci.org/c9/vfs-local)

VFS is an abstract interface for working with systems.  This module is the reference implementation and provides a vfs interface to the local system using node apis.  This is also often used in conjuction with `vfs-socket` to provide the vfs interface to a remote system over any kind of network socket.

## setup(fsOptions) -> vfs

This is the main exports of the module.  It's a function that returns a vfs-local instance.

The `fsOptions` argument is an object that can contain the following properties:

 - `root`: Root path to mount the vfs.  All fs operations are done relative to this root.  Access outside this root is not allowed.
 - `checkSymlinks`: Normally paths are resolved using pure string manipulation.  This options will aditionally realpath any symlinks to get the absolute path on the filesystem.  Pratically it prevents you from using symlinks that point outside the `root`.
 - `umask`: Default umask for creating files (defaults to 0750)
 - `defaultEnv`: A shallow hash of env values to inject into child processes.

## vfs.resolve(path, options, callback(err, meta))

This takes a virtual path as `path` and returns the resolved path in the real filesystem as `meta.path` in the callback.

This function has one option `options.alreadyRooted` that tells resolve to not prefix the path with the vfs root.

## vfs.stat(path, options, callback(err, stat))

Loads the stat information for a single path entity as `stat` in the callback.  This is a javascript object with the following fields:

 - `name`: The basename of the file path (eg: file.txt).
 - `size`: The size of the entity in bytes.
 - `mtime`: The mtime of the file in ms since epoch.
 - `mime`: The mime type of the entity.  Folders will have a mime that matches `/(directory|folder)$/`.  This vfs implementation will give `inode/directory` for directories.
 - `link`: If the file is a symlink, this property will contain the link data as a string.
 - `linkStat`: The stat information for what the link points to.
   - `fullPath`: The link stat object will have an additional property that's the resolved path relative to the vfs root.

## vfs.readfile(path, options, callback(err, meta))

Open a file as a readable stream.

Options can include:

 - `head`: Set to any truthy value to skip creating the file stream.
 - `etag`: Make a conditional request.  If the provided etag value matches the entity, the `meta` response will skip the stream and instead have a truthy `notModified` property.
 - `range`: Read only a part of a file.  The range value is an object with `start`, `end`, and/or `etag`.  Start and end are byte offsets and inclusive.  Etag means to only get the partial if the etag matches.  The `meta` object in the response will have a `partialContent` property containing an object with `start`, `end`, and `size` (total size).  The `meta.size` property will be only the size of the bytes in the partial response.  If the `options.range` object is invalid `meta` will contain `rangeNotSatisfiable`.
 - Any other options that node's `fs.createReadStream` accepts (like `encoding`)

Meta may include:

 - `stream`: A readable node stream containing the contents.
 - `size`: The length of the stream in bytes.
 - `mime`: The mime type of the file.
 - `etag`: The etag for the file.
 - `rangeNotSatisfiable`: The range was bad.
 - `partialContent`: The range was successful, this has the range offsets.
 - `notModified`: The conditional etag matched.

## vfs.readdir(path, options, callback(err, meta))

Read the contents of a directory as a stream of stat objects.  The stream in `meta.stream` emits vfs stat objects as documented above for the `data` events.  This is not a byte stream.

Options can include:

 - `head`: Set to any truthy value to skip creating the event stream.
 - `etag`: Provide an etag and the function will set `meta.partialContent` to true and skip the stream if it matches.

Meta may include:

 - `stream`: The node stream that emits stat objects.
 - `etag`: The weak etag for the directory listing.
 - `notModified`: The conditional etag matched.

## vfs.mkfile(path, options, callback)

Create or overwrite a file.  This has two modes. In one mode, the caller provides a readable stream which is then piped to the filesystem.  In the other mode, the callback meta contains a writable stream to the filesystem.

Options can include:

 - `stream`: A readable streaem to be written to the filesystem.
 - Any other options are passed through to node's `fs.createWriteStream`.

Meta may include:

 - `stream`: If a stream wasn't provided in the options, a writable stream is returned in the meta.  This stream will emit "done" when it's done being written to.  Any errors will also be emitted on this stream object since the callback will have already fired.

## vfs.mkdir(path, options, callback)

Create a directory at `path`.  Will error with `EEXIST` if something is already at the path.

## vfs.rmfile(path, options, callback)

Delete a file at `path`.

If `options.recursive` is truthy, it will instead shell out to `rm -rf` after resolving the path.

## vfs.rmdir(path, options, callback)

Delete a directory at `path`

## vfs.rename(path, options, callback)

Rename/move a file or directory.  There are two modes depending on the option passed in.

Options can include:

 - `to`: This property is the path to the target, the `path` option will be read as the source.
 - `from`: This property is the path of the source, the `path` option will be read as the target.

## vfs.copy(path, options, callback)

Copy a file.  There are two modes depending on the option passed in.

Options can include:

 - `to`: This property is the path to the new file, the `path` option will be read as the source.
 - `from`: This property is the path of the source, the `path` option will be read as the new file.

## vfs.symlink(path, options, callback)

Create a special symlink file. `path` is the file to create and `options.target` is the symlink data.  No translation of the link data is done.  It's taken literally.

## vfs.watch(path, options, callback)

Wrapper around node's `fs.watch` and `fs.watchFile`.

If `options.file` is truthy, then `fs.watchFile` is used.  Otherwise `fs.watch` is used.  The watcher will be at `meta.watcher`.  For consistency, `watchFile` objects will have a `close()` method added that internally calls `fs.unwatchFile` for the original path.

## vfs.connect(port, options, callback)

Make a TCP connection and return the duplex stream as `meta.stream`.

Options can include:
 
 - `retries`: Number of times to retry connecting.  Defaults to 5.
 - `retryDelay`: The delay between retrieson ms.  Defaults to 50.

## vfs.spawn(executablePath, options, callback)

TODO: document this

## vfs.execFile(executablePath, options, callback)

TODO: document this

## vfs.on(event, handler, callback)

TODO: document this

## vfs.off(event, handler, callback)

TODO: document this

## vfs.emit(event, value, callback)

TODO: document this

## vfs.extend(name, options, callback)

TODO: document this

## vfs.unextend(name, options, callback)

TODO: document this

## vfs.use(name, options, callback)

TODO: document this
