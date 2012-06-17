# File System

    Stability: 3 - Stable

<!--name=fs-->

File I/O is provided by simple wrappers around standard POSIX functions.  To
use this module do `require('fs')`. All the methods have asynchronous and
synchronous forms.

The asynchronous form always take a completion callback as its last argument.
The arguments passed to the completion callback depend on the method, but the
first argument is always reserved for an exception. If the operation was
completed successfully, then the first argument will be `null` or `undefined`.

When using the synchronous form any exceptions are immediately thrown.
You can use try/catch to handle exceptions or allow them to bubble up.

Here is an example of the asynchronous version:

    var fs = require('fs');

    fs.unlink('/tmp/hello', function (err) {
      if (err) throw err;
      console.log('successfully deleted /tmp/hello');
    });

Here is the synchronous version:

    var fs = require('fs');

    fs.unlinkSync('/tmp/hello')
    console.log('successfully deleted /tmp/hello');

With the asynchronous methods there is no guaranteed ordering. So the
following is prone to error:

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      console.log('renamed complete');
    });
    fs.stat('/tmp/world', function (err, stats) {
      if (err) throw err;
      console.log('stats: ' + JSON.stringify(stats));
    });

It could be that `fs.stat` is executed before `fs.rename`.
The correct way to do this is to chain the callbacks.

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      fs.stat('/tmp/world', function (err, stats) {
        if (err) throw err;
        console.log('stats: ' + JSON.stringify(stats));
      });
    });

In busy processes, the programmer is _strongly encouraged_ to use the
asynchronous versions of these calls. The synchronous versions will block
the entire process until they complete--halting all connections.

Relative path to filename can be used, remember however that this path will be relative
to `process.cwd()`.

    안정성: 3 - Stable

<!--name=fs-->

파일 I/O는 표준 POSIX 함수의 랩퍼(wrapper)로 제공된다. 이 모듈을 사용하려면 
`require('fs')`를 사용해라. 모든 함수는 비동기 방식과 동기방식이 있다.

비동기 방식은 항상 마지막 파라미터로 완료 콜백함수를 받는다. 완료 콜백에 전달되는 함수는 
메서드에 따라 다르지만 첫 아규먼트는 항상 예외로 사용한다. 작업이 성공적으로 완료되면
첫 아규먼트는 `null`이나 `undefined`가 될 것이다.

동기형식을 사용할 때는 모든 예외가 즉시 던져진다. try/catch를 사용해서 예외를 
다루거나 위로 버블링할 수 있다.

다음은 비동기 버전의 예제다.

    var fs = require('fs');

    fs.unlink('/tmp/hello', function (err) {
      if (err) throw err;
      console.log('successfully deleted /tmp/hello');
    });

다음은 동기버전이다.

    var fs = require('fs');

    fs.unlinkSync('/tmp/hello')
    console.log('successfully deleted /tmp/hello');

비동기 메서드를 사용할 때는 순서대로 실행된다는 보장을 하지 않는다. 그래서 다음 
예제는 오류가 날 수 있다.

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      console.log('renamed complete');
    });
    fs.stat('/tmp/world', function (err, stats) {
      if (err) throw err;
      console.log('stats: ' + JSON.stringify(stats));
    });

`fs.stat가 `fs.rename`보다 먼저 실행될 수 있다.
이에 대한 올바른 방법은 콜백 체인으로 연결하는 것이다.

    fs.rename('/tmp/hello', '/tmp/world', function (err) {
      if (err) throw err;
      fs.stat('/tmp/world', function (err, stats) {
        if (err) throw err;
        console.log('stats: ' + JSON.stringify(stats));
      });
    });

프로세스가 바쁜 경우 프로그래머는 이러한 호출을 비동기 방식으로 사용하기를 
_강력히 추천한다_. 동기방식은 모든 연결을 멈추고 작업이 완료될 때까지
전체 프로세스를 블락킹할 것이다.

파일명에 상대경로를 사용할 수 있지만 이 상대 경로는 `process.cwd()`이 대한 
상대경로가 될 것이다.

## fs.rename(path1, path2, [callback])

Asynchronous rename(2). No arguments other than a possible exception are given
to the completion callback.

비동기 rename(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.renameSync(path1, path2)

Synchronous rename(2).

동기 rename(2).

## fs.truncate(fd, len, [callback])

Asynchronous ftruncate(2). No arguments other than a possible exception are
given to the completion callback.

동기 ftruncate(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.truncateSync(fd, len)

Synchronous ftruncate(2).

동기 ftruncate(2).

## fs.chown(path, uid, gid, [callback])

Asynchronous chown(2). No arguments other than a possible exception are given
to the completion callback.

비동기 chown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.chownSync(path, uid, gid)

Synchronous chown(2).

동기 chown(2).

## fs.fchown(fd, uid, gid, [callback])

Asynchronous fchown(2). No arguments other than a possible exception are given
to the completion callback.

비동기 fchown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fchownSync(fd, uid, gid)

Synchronous fchown(2).

동기 fchown(2).

## fs.lchown(path, uid, gid, [callback])

Asynchronous lchown(2). No arguments other than a possible exception are given
to the completion callback.

비동기 lchown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.lchownSync(path, uid, gid)

Synchronous lchown(2).

동기 lchown(2).

## fs.chmod(path, mode, [callback])

Asynchronous chmod(2). No arguments other than a possible exception are given
to the completion callback.

비동기 chmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.chmodSync(path, mode)

Synchronous chmod(2).

동기 chmod(2).

## fs.fchmod(fd, mode, [callback])

Asynchronous fchmod(2). No arguments other than a possible exception
are given to the completion callback.

비동기 fchmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fchmodSync(fd, mode)

Synchronous fchmod(2).

동기 fchmod(2).

## fs.lchmod(path, mode, [callback])

Asynchronous lchmod(2). No arguments other than a possible exception
are given to the completion callback.

비동기 lchmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.lchmodSync(path, mode)

Synchronous lchmod(2).

동기 lchmod(2).

## fs.stat(path, [callback])

Asynchronous stat(2). The callback gets two arguments `(err, stats)` where
`stats` is a [fs.Stats](#fs_class_fs_stats) object.  See the [fs.Stats](#fs_class_fs_stats)
section below for more information.

비동기 stat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 
[fs.Stats](#fs_class_fs_stats) 객체이다. 더 자세한 내용은 아래의 
[fs.Stats](#fs_class_fs_stats)부분을 봐라.

## fs.lstat(path, [callback])

Asynchronous lstat(2). The callback gets two arguments `(err, stats)` where
`stats` is a `fs.Stats` object. `lstat()` is identical to `stat()`, except that if
`path` is a symbolic link, then the link itself is stat-ed, not the file that it
refers to.

동기 lstat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 `fs.Stats` 객체다.
`lstat()`은 `path`가 심볼릭 링크일 경우 참조하는 파일이 아닌 심볼릭 링크 자체의 상태라는 
점만 빼면 `stat()`와 같다.

## fs.fstat(fd, [callback])

Asynchronous fstat(2). The callback gets two arguments `(err, stats)` where
`stats` is a `fs.Stats` object. `fstat()` is identical to `stat()`, except that
the file to be stat-ed is specified by the file descriptor `fd`.

비동기 fstat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 `fs.Stats` 객체다.
`fstat()`은 상태를 확인하는 파일이 파일 디스크립터 `fd`가 지정한 파일이라는 점만 빼면 
`stat()`와 같다.

## fs.statSync(path)

Synchronous stat(2). Returns an instance of `fs.Stats`.

동기 stat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.lstatSync(path)

Synchronous lstat(2). Returns an instance of `fs.Stats`.

동기 lstat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.fstatSync(fd)

Synchronous fstat(2). Returns an instance of `fs.Stats`.

동기 fstat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.link(srcpath, dstpath, [callback])

Asynchronous link(2). No arguments other than a possible exception are given to
the completion callback.

비동기 link(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.linkSync(srcpath, dstpath)

Synchronous link(2).

동기 link(2).

## fs.symlink(linkdata, path, [type], [callback])

Asynchronous symlink(2). No arguments other than a possible exception are given
to the completion callback.
`type` argument can be either `'dir'` or `'file'` (default is `'file'`).  It is only 
used on Windows (ignored on other platforms).

비동기 symlink(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.
`type` 아규먼트는 `'dir'`이나 `'file'`가 가능하다.(기본값은 `'file'`이다) 이 옵션은 
윈도우에서만 사용된다.(다른 플랫폼에서는 무시한다.)

## fs.symlinkSync(linkdata, path, [type])

Synchronous symlink(2).

동기 symlink(2).

## fs.readlink(path, [callback])

Asynchronous readlink(2). The callback gets two arguments `(err,
linkString)`.

비동기 readlink(2). 콜백은 두 아규먼트 `(err, linkString)`를 받는다.

## fs.readlinkSync(path)

Synchronous readlink(2). Returns the symbolic link's string value.

동기 readlink(2). 심볼릭 링크의 문자열 값을 반환한다.

## fs.realpath(path, [callback])

Asynchronous realpath(2).  The callback gets two arguments `(err,
resolvedPath)`.  May use `process.cwd` to resolve relative paths.

비동기 realpath(2). 콜백은 두 아규먼트 `(err, resolvedPath)`를 받는다.
상대경로를 처리하려면 `process.cwd`를 사용해야 할 것이다.

## fs.realpathSync(path)

Synchronous realpath(2). Returns the resolved path.

동기 realpath(2). 처리된 경로를 반환한다.

## fs.unlink(path, [callback])

Asynchronous unlink(2). No arguments other than a possible exception are given
to the completion callback.

비동기 unlink(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.unlinkSync(path)

Synchronous unlink(2).

동기 unlink(2).

## fs.rmdir(path, [callback])

Asynchronous rmdir(2). No arguments other than a possible exception are given
to the completion callback.

비동기 rmdir(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.rmdirSync(path)

Synchronous rmdir(2).

동기 rmdir(2).

## fs.mkdir(path, [mode], [callback])

Asynchronous mkdir(2). No arguments other than a possible exception are given
to the completion callback. `mode` defaults to `0777`.

비동기 mkdir(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.
`mode`의 기본값은 `0777`이다.

## fs.mkdirSync(path, [mode])

Synchronous mkdir(2).

동기 mkdir(2).

## fs.readdir(path, [callback])

Asynchronous readdir(3).  Reads the contents of a directory.
The callback gets two arguments `(err, files)` where `files` is an array of
the names of the files in the directory excluding `'.'` and `'..'`.

비동기 readdir(3). 디렉토리의 내용을 읽는다.
콜백은 두 아규먼트 `(err, files)`를 받고 `files`는 디렉토리에서 `'.'`와 `'..'`를 
제외한 파일명들의 배열이다.

## fs.readdirSync(path)

Synchronous readdir(3). Returns an array of filenames excluding `'.'` and
`'..'`.

동기 readdir(3). `'.'`와 `'..'`를 제외한 파일명들의 배열을 반환한다.

## fs.close(fd, [callback])

Asynchronous close(2).  No arguments other than a possible exception are given
to the completion callback.

비동기 close(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.closeSync(fd)

Synchronous close(2).

동기 close(2).

## fs.open(path, flags, [mode], [callback])

Asynchronous file open. See open(2). `flags` can be:

* `'r'` - Open file for reading.
An exception occurs if the file does not exist.

* `'r+'` - Open file for reading and writing.
An exception occurs if the file does not exist.

* `'w'` - Open file for writing.
The file is created (if it does not exist) or truncated (if it exists).

* `'w+'` - Open file for reading and writing.
The file is created (if it does not exist) or truncated (if it exists).

* `'a'` - Open file for appending.
The file is created if it does not exist.

* `'a+'` - Open file for reading and appending.
The file is created if it does not exist.

`mode` defaults to `0666`. The callback gets two arguments `(err, fd)`.

비동기 파일 열기. open(2).를 봐라. `flags`는 다음의 값이 될 수 있다.

* `'r'` - 읽기모드로 파일을 연다.
파일이 존재하지 않으면 예외가 발생한다.

* `'r+'` - 읽기와 쓰기모드로 파일을 연다.
(파일이 존재하지 않으면) 파일을 생성하거나 (파일이 존재하면) 새로 쓴다.

* `'w'` - 쓰기모드로 파일을 연다.
(파일이 존재하지 않으면) 파일을 생성하거나 (파일이 존재하면) 새로 쓴다.

* `'w+'` - 읽기와 쓰기모드로 파일을 연다.
(파일이 존재하지 않으면) 파일을 생성하거나 (파일이 존재하면) 새로 쓴다.

* `'a'` - 추가모드로 파일을 연다.
파일이 존재하지 않으면 예외가 발생한다.

* `'a+'` - 읽기와 추가모드로 파일을 연다.
파일이 존재하지 않으면 예외가 발생한다.

`mode`와 기본값은 `0666`이다. 콜백은 두 아규먼트 `(err, fd)`를 받는다.

## fs.openSync(path, flags, [mode])

Synchronous open(2).

동기 open(2).

## fs.utimes(path, atime, mtime, [callback])
## fs.utimesSync(path, atime, mtime)

Change file timestamps of the file referenced by the supplied path.

전달한 경로가 참조하는 파일의 타임스탬프를 변경한다.

## fs.futimes(fd, atime, mtime, [callback])
## fs.futimesSync(fd, atime, mtime)

Change the file timestamps of a file referenced by the supplied file
descriptor.

전달한 파일 디스크립터가 참조하는 파일의 타임스탬프를 변경한다.

## fs.fsync(fd, [callback])

Asynchronous fsync(2). No arguments other than a possible exception are given
to the completion callback.

비동기 fsync(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fsyncSync(fd)

Synchronous fsync(2).

동기 fsync(2).

## fs.write(fd, buffer, offset, length, position, [callback])

Write `buffer` to the file specified by `fd`.

`offset` and `length` determine the part of the buffer to be written.

`position` refers to the offset from the beginning of the file where this data
should be written. If `position` is `null`, the data will be written at the
current position.
See pwrite(2).

The callback will be given three arguments `(err, written, buffer)` where `written`
specifies how many _bytes_ were written from `buffer`.

Note that it is unsafe to use `fs.write` multiple times on the same file
without waiting for the callback. For this scenario,
`fs.createWriteStream` is strongly recommended.

`fd`가 지정한 파일에 `buffer`를 작성한다.

`offset`과 `length`는 작성할 버퍼의 부분을 결정한다.

`position`은 이 데이터를 작성해야할 파일의 시작 위치부터의 오프셋을 참조한다. `position`이 
`null`이면 데이터는 현재 위치에 작성할 것이다.
pwrite(2)를 봐라.

콜백은 세 아규먼트 `(err, written, buffer)`를 받고 `written`는 `buffer`에서 
얼마나 많은 _바이트_가 작성되었는 지를 가리킨다.

콜백을 기다리지 않고 같은 파일에 여러번 `fs.write`를 사용하는 것은 안전하지 않다. 이 경우에 
`fs.createWriteStream`를 사용하기를 강력하게 추천한다.

## fs.writeSync(fd, buffer, offset, length, position)

Synchronous version of buffer-based `fs.write()`. Returns the number of bytes
written.

버퍼기반 `fs.write()`의 동기 버전. 작성한 바이트 수를 반환한다.

## fs.writeSync(fd, str, position, [encoding])

Synchronous version of string-based `fs.write()`. `encoding` defaults to
`'utf8'`. Returns the number of _bytes_ written.

문자열기반 `fs.write()`의 동기 버전. `encoding`의 기본값은 `'utf8'`이다. 
작성한 _바이트_ 수를 반환한다.

## fs.read(fd, buffer, offset, length, position, [callback])

Read data from the file specified by `fd`.

`buffer` is the buffer that the data will be written to.

`offset` is offset within the buffer where writing will start.

`length` is an integer specifying the number of bytes to read.

`position` is an integer specifying where to begin reading from in the file.
If `position` is `null`, data will be read from the current file position.

The callback is given the three arguments, `(err, bytesRead, buffer)`.

`fd`가 지정한 파일에서 데이터를 읽는다.

`buffer`는 데이터가 작성될 버퍼이다.

`offset`은 버퍼내에서 작성이 시작될 오프셋이다.

`length`는 읽어들일 바이트 수를 지정하는 정수이다.

`position`은 파일에서 읽어들이기 시작하는 위치를 지정하는 정수이다.
`position`가 `null`이면 데이터는 파일의 현재 위치에서 읽을 것이다. 

콜백은 세 아규먼트 `(err, bytesRead, buffer)`를 받는다.

## fs.readSync(fd, buffer, offset, length, position)

Synchronous version of buffer-based `fs.read`. Returns the number of
`bytesRead`.

버퍼기반 `fs.read`의 동기 버전이다. `bytesRead`의 수를 반환한다.

## fs.readSync(fd, length, position, encoding)

Synchronous version of string-based `fs.read`. Returns the number of
`bytesRead`.

문자열기반 `fs.read`의 동기버전이다. `bytesRead`의 수를 반환한다.

## fs.readFile(filename, [encoding], [callback])

Asynchronously reads the entire contents of a file. Example:

    fs.readFile('/etc/passwd', function (err, data) {
      if (err) throw err;
      console.log(data);
    });

The callback is passed two arguments `(err, data)`, where `data` is the
contents of the file.

If no encoding is specified, then the raw buffer is returned.

파일의 전체 내용을 비동기로 읽는다. 예제:

    fs.readFile('/etc/passwd', function (err, data) {
      if (err) throw err;
      console.log(data);
    });

콜백에는 두 아규먼트 `(err, data)`를 전달하고 `data`는 파일의 내용이다.


## fs.readFileSync(filename, [encoding])

Synchronous version of `fs.readFile`. Returns the contents of the `filename`.

If `encoding` is specified then this function returns a string. Otherwise it
returns a buffer.

`fs.readFile`의 동기버전이다. `filename`의 내용을 반환한다.

`encoding`을 지정하면 이 함수는 문자열을 반환하고 `encoding`을 지정하지 않으면 
버퍼를 반환한다.


## fs.writeFile(filename, data, [encoding], [callback])

Asynchronously writes data to a file, replacing the file if it already exists.
`data` can be a string or a buffer. The `encoding` argument is ignored if
`data` is a buffer. It defaults to `'utf8'`.

Example:

    fs.writeFile('message.txt', 'Hello Node', function (err) {
      if (err) throw err;
      console.log('It\'s saved!');
    });

비동기로 파일에 데이터를 작성하고 파일이 이미 존재하는 경우에는 파일을 대체한다.
`data`는 문자열이나 버퍼가 될 수 있다. `data`가 버퍼일 경우 `encoding`  아규먼트는 
무시된다. `encoding`의 기본값은 `'utf8'`이다. 

예제:

    fs.writeFile('message.txt', 'Hello Node', function (err) {
      if (err) throw err;
      console.log('It\'s saved!');
    });

## fs.writeFileSync(filename, data, [encoding])

The synchronous version of `fs.writeFile`.

`fs.writeFile`의 동기버전이다.

## fs.watchFile(filename, [options], listener)

    Stability: 2 - Unstable.  Use fs.watch instead, if available.

Watch for changes on `filename`. The callback `listener` will be called each
time the file is accessed.

The second argument is optional. The `options` if provided should be an object
containing two members a boolean, `persistent`, and `interval`. `persistent`
indicates whether the process should continue to run as long as files are
being watched. `interval` indicates how often the target should be polled,
in milliseconds. (On Linux systems with inotify, `interval` is ignored.) The
default is `{ persistent: true, interval: 0 }`.

The `listener` gets two arguments the current stat object and the previous
stat object:

    fs.watchFile('message.text', function (curr, prev) {
      console.log('the current mtime is: ' + curr.mtime);
      console.log('the previous mtime was: ' + prev.mtime);
    });

These stat objects are instances of `fs.Stat`.

If you want to be notified when the file was modified, not just accessed
you need to compare `curr.mtime` and `prev.mtime`.

    안정성: 2 - Unstable.  가능하다면 대신 fs.watch를 사용해라.

`filename`의 변경사항을 감시한다. 콜백 `listener`는 파일이 접근될 때마다 호출될 
것이다.

두번째 아규먼트는 선택사항이다. `options`을 전달하는 경우 `options`은 두 불리언값의 
멤버변수 `persistent`와 `interval`을 담고 있는 객체가 될 것이다. `persistent`는 
파일을 감사하는 동안 계속해서 프로세스가 실행되어야 하는지를 나타낸다. `interval`은 
얼마나 자주 대상을 확인해야 하는지를 밀리초로 나타낸다. (inotify를 사용하는 Linux 
시스템에서 `interval`는 무시한다.) 기본값은 `{ persistent: true, interval: 0 }`이다.

`listener`는 두 아규먼트 현재의 stat 객체와 이전의 stat 객체를 받는다.

    fs.watchFile('message.text', function (curr, prev) {
      console.log('the current mtime is: ' + curr.mtime);
      console.log('the previous mtime was: ' + prev.mtime);
    });

이 stat 객체들은 `fs.Stat` 인스턴스다.

그냥 파일에 접근만 했을 때가 아니라 파일이 수정되었을 때 알림을 받고 싶다면 
`curr.mtime`와 `prev.mtime`를 비교해야 한다.


## fs.unwatchFile(filename)

    Stability: 2 - Unstable.  Use fs.watch instead, if available.

Stop watching for changes on `filename`.

    안정성: 2 - Unstable.  가능하다면 대신 fs.watch를 사용해라.

`filename`의 변경사항을 감시하는 것을 멈춘다.

## fs.watch(filename, [options], listener)

    Stability: 2 - Unstable.  Not available on all platforms.

Watch for changes on `filename`, where `filename` is either a file or a
directory.  The returned object is a [fs.FSWatcher](#fs_class_fs_fswatcher).

The second argument is optional. The `options` if provided should be an object
containing a boolean member `persistent`, which indicates whether the process
should continue to run as long as files are being watched. The default is
`{ persistent: true }`.

The listener callback gets two arguments `(event, filename)`.  `event` is either
'rename' or 'change', and `filename` is the name of the file which triggered
the event.

    안정성: 2 - Unstable.  모든 플랫폼에서 사용할 수 있는 것은 아니다.

`filename`의 변경사항을 감시하고 `filename`은 파일이나 디렉토리가 될 수 있다. 
반환객체는 [fs.FSWatcher](#fs_class_fs_fswatcher)이다.

두번째 아규먼트는 선택사항이다. `options`을 전달한다면 `options`은 불리언값의 멤버변수 
`persistent`를 담고 있는 객체여야 한다. `persistent`는 파일을 감사하는 동안 계속해서 
프로세스가 실행되어야 하는지를 나타낸다. 기본값은 `{ persistent: true }`이다.

리스너 콜백은 두 아규먼트 `(event, filename)`를 받는다. `event`는 'rename'나 'change'이고 
`filename`은 이벤트를 발생시킨 파일의 이름이다.

### 경고(Caveats)

<!--type=misc-->

The `fs.watch` API is not 100% consistent across platforms, and is
unavailable in some situations.

<!--type=misc-->

`fs.watch` API는 모든 플랫폼에서 100% 일치하지 않고 몇몇 상황에서는 
사용할 수 없다.

#### 가용성(Availability)

<!--type=misc-->

This feature depends on the underlying operating system providing a way
to be notified of filesystem changes.

* On Linux systems, this uses `inotify`.
* On BSD systems (including OS X), this uses `kqueue`.
* On SunOS systems (including Solaris and SmartOS), this uses `event ports`.
* On Windows systems, this feature depends on `ReadDirectoryChangesW`.

If the underlying functionality is not available for some reason, then
`fs.watch` will not be able to function.  You can still use
`fs.watchFile`, which uses stat polling, but it is slower and less
reliable.

<!--type=misc-->

이 기능은 의존 운영체제가 제공하는 파일시스템의 변경사항을 알리는 방법에 따라 다르다.

* Linux 시스템에서 이 기능은 `inotify`를 사용한다.
* BSD 시스템에서 (OS X 포함) 이 기능은 `kqueue`를 사용한다.
* SunOS 시스템에서 (Solaris와 SmartOS 포함) 이 기능은 `event ports`를 사용한다.
* Windows 시스템에서 이 기능은 `ReadDirectoryChangesW`에 달려있다.

몇가지 이유로 의존하는 기능을 사용할 수 없다면 `fs.watch`를 사용할 수 없을 것이다.
stat 폴링(polling)을 사용하지만 더 느리고 덜 신뢰적인 `fs.watchFile`는 여전히 
사용할 수 있다.

#### 파일명 아규먼트(Filename Argument)

<!--type=misc-->

Providing `filename` argument in the callback is not supported
on every platform (currently it's only supported on Linux and Windows).  Even
on supported platforms `filename` is not always guaranteed to be provided.
Therefore, don't assume that `filename` argument is always provided in the
callback, and have some fallback logic if it is null.

    fs.watch('somedir', function (event, filename) {
      console.log('event is: ' + event);
      if (filename) {
        console.log('filename provided: ' + filename);
      } else {
        console.log('filename not provided');
      }
    });

<!--type=misc-->

모든 클랫폼에서 `filename` 아규먼트를 콜백에 전달하는 것은 아니다. (현재는 Linux와 
Windows에서만 지원한다.) 이를 지원하는 플랫폼에서 조차도 `filename`을 항상 제공한다고 
보장하는 것은 아니다. 그러므로 콜백에 `filename` 아규먼트가 항상 전달된다고 가정하지 말고 
null 일 경우를 위한 대체(fallback) 로직을 가지고 있어야 한다.

    fs.watch('somedir', function (event, filename) {
      console.log('event is: ' + event);
      if (filename) {
        console.log('filename provided: ' + filename);
      } else {
        console.log('filename not provided');
      }
    });

## Class: fs.Stats

Objects returned from `fs.stat()`, `fs.lstat()` and `fs.fstat()` and their
synchronous counterparts are of this type.

 - `stats.isFile()`
 - `stats.isDirectory()`
 - `stats.isBlockDevice()`
 - `stats.isCharacterDevice()`
 - `stats.isSymbolicLink()` (only valid with  `fs.lstat()`)
 - `stats.isFIFO()`
 - `stats.isSocket()`

For a regular file `util.inspect(stats)` would return a string very
similar to this:

    { dev: 2114,
      ino: 48064969,
      mode: 33188,
      nlink: 1,
      uid: 85,
      gid: 100,
      rdev: 0,
      size: 527,
      blksize: 4096,
      blocks: 8,
      atime: Mon, 10 Oct 2011 23:24:11 GMT,
      mtime: Mon, 10 Oct 2011 23:24:11 GMT,
      ctime: Mon, 10 Oct 2011 23:24:11 GMT }

Please note that `atime`, `mtime` and `ctime` are instances
of [Date][MDN-Date] object and to compare the values of
these objects you should use appropriate methods. For most
general uses [getTime()][MDN-Date-getTime] will return
the number of milliseconds elapsed since _1 January 1970
00:00:00 UTC_ and this integer should be sufficient for
any comparison, however there additional methods which can
be used for displaying fuzzy information. More details can
be found in the [MDN JavaScript Reference][MDN-Date] page.

`fs.stat()`, `fs.lstat()`, `fs.fstat()`가 리턴하는 객체고 이 함수들의 동기함수들도 
이 타입을 리턴한다.

 - `stats.isFile()`
 - `stats.isDirectory()`
 - `stats.isBlockDevice()`
 - `stats.isCharacterDevice()`
 - `stats.isSymbolicLink()` (only valid with  `fs.lstat()`)
 - `stats.isFIFO()`
 - `stats.isSocket()`

정규 파일에 대한 `util.inspect(stats)`는 다음과 유사한 문자열을 리턴할 
것이다.

    { dev: 2114,
      ino: 48064969,
      mode: 33188,
      nlink: 1,
      uid: 85,
      gid: 100,
      rdev: 0,
      size: 527,
      blksize: 4096,
      blocks: 8,
      atime: Mon, 10 Oct 2011 23:24:11 GMT,
      mtime: Mon, 10 Oct 2011 23:24:11 GMT,
      ctime: Mon, 10 Oct 2011 23:24:11 GMT }

`atime`, `mtime`, `ctime`는 [Date][MDN-Date] 객체의 인스턴스이고 
이 객체의 값들을 비교하려면 적절한 메서드를 사용해야한다. 대부분의 경우 
[getTime()][MDN-Date-getTime]는 1980년 1월 _1일부터 경과된 
밀리초를 반환할 것이고 이 정수값은 비교하기에 충분하다. 하지만 명확하지 
않은 정보를 보여주는데 사용할 수 있는 추가적인 메서드들이 있다. 더 자세한 
내용은 [MDN JavaScript Reference][MDN-Date] 페이지에 있다.

[MDN-Date]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date
[MDN-Date-getTime]: https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Date/getTime


## fs.createReadStream(path, [options])

Returns a new ReadStream object (See `Readable Stream`).

`options` is an object with the following defaults:

    { flags: 'r',
      encoding: null,
      fd: null,
      mode: 0666,
      bufferSize: 64 * 1024
    }

`options` can include `start` and `end` values to read a range of bytes from
the file instead of the entire file.  Both `start` and `end` are inclusive and
start at 0.

An example to read the last 10 bytes of a file which is 100 bytes long:

    fs.createReadStream('sample.txt', {start: 90, end: 99});

새로운 ReadStream 객체를 반환한다. (`Readable Stream`를 봐라)

`options`는 다음의 기본값을 가진 객체다.

    { flags: 'r',
      encoding: null,
      fd: null,
      mode: 0666,
      bufferSize: 64 * 1024
    }

`options`는 전체 파일대신 읽어드릴 파일의 범위인 `start`와 `end`를 포함할 수 있다.
`start`와 `end` 둘 다 포함하고 0 부터 시작한다.

100 바이트 길이인 파일의 마지막 10 바이트를 읽는 예제.

    fs.createReadStream('sample.txt', {start: 90, end: 99});


## Class: fs.ReadStream

`ReadStream` is a [Readable Stream](stream.html#stream_readable_stream).

`ReadStream`는 [Readable Stream](stream.html#stream_readable_stream)이다.

### Event: 'open'

* `fd` {Integer} file descriptor used by the ReadStream.

Emitted when the ReadStream's file is opened.

* ReadStream는 `fd` {Integer} 파일 디스크립터를 사용한다.

ReadStream의 파일이 열렸을 때 발생한다.


## fs.createWriteStream(path, [options])

Returns a new WriteStream object (See `Writable Stream`).

`options` is an object with the following defaults:

    { flags: 'w',
      encoding: null,
      mode: 0666 }

`options` may also include a `start` option to allow writing data at
some position past the beginning of the file.  Modifying a file rather
than replacing it may require a `flags` mode of `r+` rather than the
default mode `w`.

새로운 WriteStream 객체를 반환한다. (`Writable Stream`를 봐라.)

`options`는 다음의 기본값을 갖는 객체다.

    { flags: 'w',
      encoding: null,
      mode: 0666 }

`options`도 파일의 시작위치가 아닌 다른 위치에 데이터를 작성하도록 `start` 옵션을 
포함할 수도 있다. 파일을 교체하는 대신에 파일을 수정하려면 `flags` 모드를 기본값인 
`w` 대신에 `r+`를 사용해야 한다.

## fs.WriteStream

`WriteStream` is a [Writable Stream](stream.html#stream_writable_stream).

`WriteStream`는 [Writable Stream](stream.html#stream_writable_stream)이다.

### Event: 'open'

* `fd` {Integer} file descriptor used by the ReadStream.

Emitted when the WriteStream's file is opened.

* ReadStream는 `fd` {Integer} 파일 디스크립터를 사용한다.

WriteStream의 파일이 열렸을 때 발생한다.

### file.bytesWritten

The number of bytes written so far. Does not include data that is still queued
for writing.

지금까지 작성된 바이트의 수. 작성하기 위해 아직 큐에 있는 데이터는 포함하지 않는다.

## Class: fs.FSWatcher

Objects returned from `fs.watch()` are of this type.

`fs.watch()`가 반환하는 객체가 이 타입이다.

### watcher.close()

Stop watching for changes on the given `fs.FSWatcher`.

주어진 `fs.FSWatcher`에서 변경사항을 감시하는 것을 멈춘다.

### Event: 'change'

* `event` {String} The type of fs change
* `filename` {String} The filename that changed (if relevant/available)

Emitted when something changes in a watched directory or file.
See more details in [fs.watch](#fs_fs_watch_filename_options_listener).

* `event` {String} fs 변경사항의 타입
* `filename` {String} 변경된 파일명 (적절하거나 사용가능하다면)

감시하는 디렉토리나 파일명에서 어떤 변경이 생겼을 때 발생한다.
더 자세한 내용은 [fs.watch](#fs_fs_watch_filename_options_listener)를 봐라.

### Event: 'error'

* `error` {Error object}

Emitted when an error occurs.

* `error` {Error object}

오류가 생겼을 때 발생한다.
