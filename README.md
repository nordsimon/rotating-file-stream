# rotating-file-stream

[![Build Status](https://travis-ci.org/iccicci/rotating-file-stream.png)](https://travis-ci.org/iccicci/rotating-file-stream)
[![Code Climate](https://codeclimate.com/github/iccicci/rotating-file-stream/badges/gpa.svg)](https://codeclimate.com/github/iccicci/rotating-file-stream)
[![Test Coverage](https://codeclimate.com/github/iccicci/rotating-file-stream/badges/coverage.svg)](https://codeclimate.com/github/iccicci/rotating-file-stream/coverage)
[![Donate](http://img.shields.io/donate/bitcoin.png?color=blue)](https://blockchain.info/address/12p1p5q7sK75tPyuesZmssiMYr4TKzpSCN)

[![NPM version](https://badge.fury.io/js/rotating-file-stream.svg)](https://www.npmjs.com/package/rotating-file-stream)
[![bitHound Dependencies](https://www.bithound.io/github/iccicci/rotating-file-stream/badges/dependencies.svg)](https://www.bithound.io/github/iccicci/rotating-file-stream/master/dependencies/npm)
[![bitHound Dev Dependencies](https://www.bithound.io/github/iccicci/rotating-file-stream/badges/devDependencies.svg)](https://www.bithound.io/github/iccicci/rotating-file-stream/master/dependencies/npm)

[![NPM](https://nodei.co/npm/rotating-file-stream.png?downloads=true&downloadRank=true&stars=true)](https://nodei.co/npm/rotating-file-stream/)

### Description

Creates a [stream.Writable](https://nodejs.org/api/stream.html#stream_class_stream_writable) to a file which is rotated.
Rotation behaviour can be deeply customized; optionally, classical UNIX __logrotate__ behaviour can be used.

### Usage

```javascript
var rfs    = require('rotating-file-stream');
var stream = rfs('file.log', {
    size:     '10M', // rotate every 10 MegaBytes written
    interval: '1d',  // rotate daily
    compress: 'gzip' // compress rotated files
});
```

### Installation

With [npm](https://www.npmjs.com/package/rotating-file-stream):
```sh
$ npm install --save rotating-file-stream
```

# API

```javascript
require('rotating-file-stream');
```
Returns __RotatingFileStream__ constructor.

## Class: RotatingFileStream
Extends [stream.Writable](https://nodejs.org/api/stream.html#stream_class_stream_writable).

## [new] RotatingFileStream(filename, options)

Returns a new __RotatingFileStream__ to _filename_ as
[fs.createWriteStream](https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options) does.
The file is rotated following _options_ rules.

### filename {String|Function}

The most complex problem about file name is: "how to call the rotated file name?"

The answer to this question may vary in many forms depending on application requirements and/or specifications.
If there are no requirements, a _String_ can be used and _default rotated file name generator_ will be used;
otherwise a _Function_ which returns the _rotated file name_ can be used.

#### function filename(time, index)

* time: {Date} If both rotation by interval is enabled and __options.rotationTime__ [(see below)](#rotationtime) is
__false__, the start time of rotation period, otherwise the time when rotation job started. If __null__, the
_not-rotated file name_ must be returned.
* index {Number} The progressive index of rotation by size in the same rotation period.

An example of a complex _rotated file name generator_ function could be:

```javascript
function pad(num) {
    return (num > 9 ? "" : "0") + num;
}

function generator(time, index) {
    if(! time)
        return "file.log";

    var month  = time.getFullYear() + "" + pad(time.getMonth() + 1);
    var day    = pad(time.getDate());
    var hour   = pad(time.getHours());
    var minute = pad(time.getMinutes());

    return "/storage/" + month + "/" + month +
        day + "-" + hour + minute + "-" + index + "-file.log";
}

var rfs    = require('rotating-file-stream');
var stream = rfs(generator, {
    size:     '10M',
    interval: '30m'
});
```

__Note:__
if both rotation by interval and rotation by time are used, returned _rotated file name_ __must__ be function of both
parameters _time_ and _index_. Alternatively, __rotationTime__ _option_ can be used (to see below).

If classical __logrotate__ behaviour is enabled _rotated file name_ is only a function of _index_.

#### function filename(index)

* index {Number} The progressive index of rotation. If __null__, the _not-rotated file name_ must be returned.

__Note:__
The _not-rotated file name_ __must__ be only the _filename_, to specify a _path_ the appropriate option __must__ be used.
```javascript
rfs('path/to/file.log'); // wrong
rfs('file.log', { path: 'path/to' }); // OK
```

__Note:__
if part of returned destination path does not exists, the rotation job will try to create it.

### options {Object}

* compress: {String|Function|True} (default: null) Specifies compression method of rotated files.
* highWaterMark: {Number} (default: null) Proxied to [new stream.Writable](https://nodejs.org/api/stream.html#stream_constructor_new_stream_writable_options)
* history: {String} (default: null) Specifies the _history filename_.
* initialRotation: {Boolean} (default: null) Eventually makes an initial rotation based on _not-rotated file_ timestamp.
* interval: {String} (default: null) Specifies the time interval to rotate the file.
* maxFiles: {Integer} (default: null) Specifies the maximum number of rotated files to keep.
* maxSize: {String} (default: null) Specifies the maximum size of rotated files to keep.
* mode: {Integer} (default: null) Proxied to [fs.createWriteStream](https://nodejs.org/api/fs.html#fs_fs_createwritestream_path_options)
* path: {String} (default: null) Specifies the base path for files.
* rotate: {Integer} (default: null) Enables the classical UNIX __logrotate__ behaviour.
* rotationTime: {Boolean} (default: null) Makes rotated file name with time of rotation instead of start time of period.
* size: {String} (default: null) Specifies the file size to rotate the file.

#### path

If present, it is prepended to generated file names as well as for history file.

#### size

Accepts a positive integer followed by one of these possible letters:

* __B__: Bites
* __K__: KiloBites
* __M__: MegaBytes
* __G__: GigaBytes

```javascript
  size: '300B', // rotates the file when size exceeds 300 Bytes
                // useful for tests
```

```javascript
  size: '300K', // rotates the file when size exceeds 300 KiloBytes
```

```javascript
  size: '100M', // rotates the file when size exceeds 100 MegaBytes
```

```javascript
  size: '1G', // rotates the file when size exceeds a GigaByte
```

#### interval

Accepts a positive integer followed by one of these possible letters:

* __s__: seconds. Accepts integer divider of 60.
* __m__: minutes. Accepts integer divider of 60.
* __h__: hours. Accepts integer divider of 24.
* __d__: days

```javascript
  interval: '5s', // rotates at seconds 0, 5, 10, 15 and so on
                  // useful for tests
```

```javascript
  interval: '5m', // rotates at minutes 0, 5, 10, 15 and so on
```

```javascript
  interval: '2h', // rotates at midnight, 02:00, 04:00 and so on
```

```javascript
  interval: '1d', // rotates at every midnight
```

#### compress

Due the nature of __Node.js__ compression may be done with an external command (to use other CPUs than the one used
by __Node.js__) or with internal code (to use the CPU used by __Node.js__). This decision is left to you.

Following fixed strings are allowed to compress the files with internal libraries:
* bzip2 (__not implemented yet__)
* gzip

To enable external compression, a _function_ can be used or simply the _boolean_ __true__ value to use default
external compression.
The function should accept _source_ and _dest_ file names and must return the shell command to be executed to
compress the file.
The two following code snippets have exactly the same effect:

```javascript
var rfs    = require('rotating-file-stream');
var stream = rfs('file.log', {
    size:     '10M',
    compress: true
});
```

```javascript
var rfs    = require('rotating-file-stream');
var stream = rfs('file.log', {
    size:     '10M',
    compress: function(source, dest) {
        return "cat " + source + " | gzip -c9 > " + dest;
    }
});
```

__Note:__
the shell command to compress the rotated file should not remove the source file, it will be removed by the package
if rotation job complete with success.

#### rotationTime

As specified above, if rotation by interval is enabled, the parameter _time_ passed to _rotatle name generator_ is the
start time of rotation period. Setting this option to __true__, parameter _time_ passed is time when rotation job
started.

#### initialRotation
When program stops in a rotation period then restarts in a new rotation period, logs of differente rotation period will
go in the next rotated file; in a few words: a rotation job is lost. If this option is set to __true__ an initial check
is performed against the _not-rotated file_ timestamp and, if it falls in a previous rotation period, an initial
rotation job is done as well.

__Note:__
this option is ignored if __rotationTime__ is set to __true__.

#### rotate

If specified, classical UNIX __logrotate__ behaviour is enabled and the value of this option has same effect in
_logrotate.conf_ file.

__Note:__
following options are ignored if __rotate__ option is specified.

#### history

Due to the complexity that _rotated file names_ can have because of the _filename generator function_, if number or
size of rotated files should not exceed a given limit, the package needs a file where to store this information. This
option specifies the name of that file. This option takes effects only if at least one of __maxFiles__ or __maxSize__
is used. If __null__, the _not rotated filename_ with the '.txt' suffix is used.

#### maxFiles

If specified, it's value is the maximum number of _rotated files_ to be kept.

#### maxSize

If specified, it's value must respect same syntax of [size](#size) option and is the maximum size of _rotated files_
to be kept.

## Events

Custom _Events_ are emitted by the stream.

```javascript
var rfs    = require('rotating-file-stream');
var stream = rfs(...);

stream.on('error', function(err) {
    // here are reported blocking errors
    // once this event is emitted, the stream will be closed as well
});

stream.on('open', function() {
    // no rotated file is open (emitted after each rotation as well)
});

stream.on('removed', function(filename, number) {
    // rotation job removed the specified old rotated file
    // number == true, the file was removed to not exceed maxFiles
    // number == false, the file was removed to not exceed maxSize
});

stream.on('rotation', function() {
    // rotation job started
});

stream.on('rotated', function(filename) {
    // rotation job completed with success producing given filename
});

stream.on('warning', function(err) {
    // here are reported non blocking errors
});
```

## Rotation logic

Regardless of when and why rotation happens, the content of a single
[stream.write](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback)
will never be split among two files.

### by size

Once the _not-rotated_ file is opened first time, its size is checked and if it is greater or equal to
size limit, a first rotation happens. After each
[stream.write](https://nodejs.org/api/stream.html#stream_writable_write_chunk_encoding_callback),
the same check is performed.

### by interval

The package sets a [Timeout](https://nodejs.org/api/timers.html#timers_settimeout_callback_delay_args)
to start a rotation job at the right moment.

## Under the hood

Logs should be handled so carefully, so this package tries to never overwrite files.

At stream creation, if the _not-rotated_ log file already exists and its size exceeds the rotation size,
an initial rotation attempt is done.

At each rotation attempt a check is done to verify that destination rotated file does not exists yet;
if this is not the case a new destination _rotated file name_ is generated and the same check is
performed before going on. This is repeated until a not existing destination file name is found or the
package is exhausted. For this reason the _rotated file name generator_ function may be called several
times for each rotation job.

If requested by __maxFiles__ or __maxSize__ options, at the end of a rotation job, a check is performed to ensure that
given limits are respected. This means that __while rotation job is running both the limits could be not respected__,
the same can happend (if __maxFiles__ or __maxSize__ are changed) till the end of first _rotation job_.
The first check performed is the one against __maxFiles__, in case some files are removed, than the check against
__maxSize__ is performed, finally other files can be removed. When __maxFiles__ or __maxSize__ are enabled for first
time, an _history file_ can be created with one _rotated filename_ (as returned by _filename generator function_) at
each line.

Once an __error__ _event_ is emitted, nothing more can be done: the stream is closed as well.

## Compatibility

This package is written following  __Node.js 4.0__ specifications always taking care about backward
compatibility. The package is tested under
[several Node.js versions](https://travis-ci.org/iccicci/rotating-file-stream).

__Required: Node.js 0.11__

## Licence

[MIT Licence](https://github.com/iccicci/rotating-file-stream/blob/master/LICENSE)

## Bugs

Do not hesitate to report any bug or inconsistency [@github](https://github.com/iccicci/rotating-file-stream/issues).

## ChangeLog

[ChangeLog](https://github.com/iccicci/rotating-file-stream/blob/master/CHANGELOG.md)

## Donating

If you find useful this package, please consider the opportunity to donate some satoshis to this bitcoin address:
__12p1p5q7sK75tPyuesZmssiMYr4TKzpSCN__
