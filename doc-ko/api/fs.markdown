# File System

    Stability: 3 - Stable

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

비동기 rename(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.renameSync(path1, path2)

동기 rename(2).

## fs.truncate(fd, len, [callback])

동기 ftruncate(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.truncateSync(fd, len)

동기 ftruncate(2).

## fs.chown(path, uid, gid, [callback])

비동기 chown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.chownSync(path, uid, gid)

동기 chown(2).

## fs.fchown(fd, uid, gid, [callback])

비동기 fchown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fchownSync(fd, uid, gid)

동기 fchown(2).

## fs.lchown(path, uid, gid, [callback])

비동기 lchown(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.lchownSync(path, uid, gid)

동기 lchown(2).

## fs.chmod(path, mode, [callback])

비동기 chmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.chmodSync(path, mode)

동기 chmod(2).

## fs.fchmod(fd, mode, [callback])

비동기 fchmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fchmodSync(fd, mode)

동기 fchmod(2).

## fs.lchmod(path, mode, [callback])

비동기 lchmod(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.lchmodSync(path, mode)

동기 lchmod(2).

## fs.stat(path, [callback])

비동기 stat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 
[fs.Stats](#fs_class_fs_stats) 객체이다. 더 자세한 내용은 아래의 
[fs.Stats](#fs_class_fs_stats)부분을 봐라.

## fs.lstat(path, [callback])

동기 lstat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 `fs.Stats` 객체다.
`lstat()`은 `path`가 심볼릭 링크일 경우 참조하는 파일이 아닌 심볼릭 링크 자체의 상태라는 
점만 빼면 `stat()`와 같다.

## fs.fstat(fd, [callback])

비동기 fstat(2). 콜백은 두 아규먼트 `(err, stats)`를 받고 `stats`은 `fs.Stats` 객체다.
`fstat()`은 상태를 확인하는 파일이 파일 디스크립터 `fd`가 지정한 파일이라는 점만 빼면 
`stat()`와 같다.

## fs.statSync(path)

동기 stat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.lstatSync(path)

동기 lstat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.fstatSync(fd)

동기 fstat(2). `fs.Stats` 인스턴스를 반환한다.

## fs.link(srcpath, dstpath, [callback])

비동기 link(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.linkSync(srcpath, dstpath)

동기 link(2).

## fs.symlink(linkdata, path, [type], [callback])

비동기 symlink(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.
`type` 아규먼트는 `'dir'`이나 `'file'`가 가능하다.(기본값은 `'file'`이다) 이 옵션은 
윈도우에서만 사용된다.(다른 플랫폼에서는 무시한다.)

## fs.symlinkSync(linkdata, path, [type])

동기 symlink(2).

## fs.readlink(path, [callback])

비동기 readlink(2). 콜백은 두 아규먼트 `(err, linkString)`를 받는다.

## fs.readlinkSync(path)

동기 readlink(2). 심볼릭 링크의 문자열 값을 반환한다.

## fs.realpath(path, [callback])

비동기 realpath(2). 콜백은 두 아규먼트 `(err, resolvedPath)`를 받는다.
상대경로를 처리하려면 `process.cwd`를 사용해야 할 것이다.

## fs.realpathSync(path)

동기 realpath(2). 처리된 경로를 반환한다.

## fs.unlink(path, [callback])

비동기 unlink(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.unlinkSync(path)

동기 unlink(2).

## fs.rmdir(path, [callback])

비동기 rmdir(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.rmdirSync(path)

동기 rmdir(2).

## fs.mkdir(path, [mode], [callback])

비동기 mkdir(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.
`mode`의 기본값은 `0777`이다.

## fs.mkdirSync(path, [mode])

동기 mkdir(2).

## fs.readdir(path, [callback])

비동기 readdir(3). 디렉토리의 내용을 읽는다.
콜백은 두 아규먼트 `(err, files)`를 받고 `files`는 디렉토리에서 `'.'`와 `'..'`를 
제외한 파일명들의 배열이다.

## fs.readdirSync(path)

동기 readdir(3). `'.'`와 `'..'`를 제외한 파일명들의 배열을 반환한다.

## fs.close(fd, [callback])

비동기 close(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.closeSync(fd)

동기 close(2).

## fs.open(path, flags, [mode], [callback])

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

동기 open(2).

## fs.utimes(path, atime, mtime, [callback])
## fs.utimesSync(path, atime, mtime)

전달한 경로가 참조하는 파일의 타임스탬프를 변경한다.

## fs.futimes(fd, atime, mtime, [callback])
## fs.futimesSync(fd, atime, mtime)

전달한 파일 디스크립터가 참조하는 파일의 타임스탬프를 변경한다.

## fs.fsync(fd, [callback])

비동기 fsync(2). 전달한 완료콜백에는 예외 아규먼트 외에 다른 아규먼트는 없다.

## fs.fsyncSync(fd)

동기 fsync(2).

## fs.write(fd, buffer, offset, length, position, [callback])

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

버퍼기반 `fs.write()`의 동기 버전. 작성한 바이트 수를 반환한다.

## fs.writeSync(fd, str, position, [encoding])

문자열기반 `fs.write()`의 동기 버전. `encoding`의 기본값은 `'utf8'`이다. 
작성한 _바이트_ 수를 반환한다.

## fs.read(fd, buffer, offset, length, position, [callback])

`fd`가 지정한 파일에서 데이터를 읽는다.

`buffer`는 데이터가 작성될 버퍼이다.

`offset`은 버퍼내에서 작성이 시작될 오프셋이다.

`length`는 읽어들일 바이트 수를 지정하는 정수이다.

`position`은 파일에서 읽어들이기 시작하는 위치를 지정하는 정수이다.
`position`가 `null`이면 데이터는 파일의 현재 위치에서 읽을 것이다. 

콜백은 세 아규먼트 `(err, bytesRead, buffer)`를 받는다.

## fs.readSync(fd, buffer, offset, length, position)

버퍼기반 `fs.read`의 동기 버전이다. `bytesRead`의 수를 반환한다.

## fs.readSync(fd, length, position, encoding)

문자열기반 `fs.read`의 동기버전이다. `bytesRead`의 수를 반환한다.

## fs.readFile(filename, [encoding], [callback])

파일의 전체 내용을 비동기로 읽는다. 예제:

    fs.readFile('/etc/passwd', function (err, data) {
      if (err) throw err;
      console.log(data);
    });

콜백에는 두 아규먼트 `(err, data)`를 전달하고 `data`는 파일의 내용이다.

인코딩을 지정하지 않으면 로우(raw) 버퍼를 반환한다.


## fs.readFileSync(filename, [encoding])

`fs.readFile`의 동기버전이다. `filename`의 내용을 반환한다.

`encoding`을 지정하면 이 함수는 문자열을 반환하고 `encoding`을 지정하지 않으면 
버퍼를 반환한다.


## fs.writeFile(filename, data, [encoding], [callback])

비동기로 파일에 데이터를 작성하고 파일이 이미 존재하는 경우에는 파일을 대체한다.
`data`는 문자열이나 버퍼가 될 수 있다. `data`가 버퍼일 경우 `encoding`  아규먼트는 
무시된다. `encoding`의 기본값은 `'utf8'`이다. 

예제:

    fs.writeFile('message.txt', 'Hello Node', function (err) {
      if (err) throw err;
      console.log('It\'s saved!');
    });

## fs.writeFileSync(filename, data, [encoding])

`fs.writeFile`의 동기버전이다.

## fs.watchFile(filename, [options], listener)

    Stability: 2 - Unstable.  가능하다면 대신 fs.watch를 사용해라.

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

    Stability: 2 - Unstable.  가능하다면 대신 fs.watch를 사용해라.

`filename`의 변경사항을 감시하는 것을 멈춘다.

## fs.watch(filename, [options], listener)

    Stability: 2 - Unstable.  모든 플랫폼에서 사용할 수 있는 것은 아니다.

`filename`의 변경사항을 감시하고 `filename`은 파일이나 디렉토리가 될 수 있다. 
반환객체는 [fs.FSWatcher](#fs_class_fs_fswatcher)이다.

두번째 아규먼트는 선택사항이다. `options`을 전달한다면 `options`은 불리언값의 멤버변수 
`persistent`를 담고 있는 객체여야 한다. `persistent`는 파일을 감사하는 동안 계속해서 
프로세스가 실행되어야 하는지를 나타낸다. 기본값은 `{ persistent: true }`이다.

리스너 콜백은 두 아규먼트 `(event, filename)`를 받는다. `event`는 'rename'나 'change'이고 
`filename`은 이벤트를 발생시킨 파일의 이름이다.

### Caveats

<!--type=misc-->

`fs.watch` API는 모든 플랫폼에서 100% 일치하지 않고 몇몇 상황에서는 
사용할 수 없다.

#### Availability

<!--type=misc-->

이 기능은 의존 운영체제가 제공하는 파일시스템의 변경사항을 알리는 방법에 따라 다르다.

* Linux 시스템에서 이 기능은 `inotify`를 사용한다.
* BSD 시스템에서 (OS X 포함) 이 기능은 `kqueue`를 사용한다.
* SunOS 시스템에서 (Solaris와 SmartOS 포함) 이 기능은 `event ports`를 사용한다.
* Windows 시스템에서 이 기능은 `ReadDirectoryChangesW`에 달려있다.

몇가지 이유로 의존하는 기능을 사용할 수 없다면 `fs.watch`를 사용할 수 없을 것이다.
stat 폴링(polling)을 사용하지만 더 느리고 덜 신뢰적인 `fs.watchFile`는 여전히 
사용할 수 있다.

#### Filename Argument

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

`ReadStream`는 [Readable Stream](stream.html#stream_readable_stream)이다.

### Event: 'open'

* ReadStream는 `fd` {Integer} 파일 디스크립터를 사용한다.

ReadStream의 파일이 열렸을 때 발생한다.


## fs.createWriteStream(path, [options])

새로운 WriteStream 객체를 반환한다. (`Writable Stream`를 봐라.)

`options`는 다음의 기본값을 갖는 객체다.

    { flags: 'w',
      encoding: null,
      mode: 0666 }

`options`도 파일의 시작위치가 아닌 다른 위치에 데이터를 작성하도록 `start` 옵션을 
포함할 수도 있다. 파일을 교체하는 대신에 파일을 수정하려면 `flags` 모드를 기본값인 
`w` 대신에 `r+`를 사용해야 한다.

## fs.WriteStream

`WriteStream`는 [Writable Stream](stream.html#stream_writable_stream)이다.

### Event: 'open'

* `fd` {Integer} ReadStream이 사용하는 파일 디스크립터.

WriteStream의 파일이 열렸을 때 발생한다.

### file.bytesWritten

지금까지 작성된 바이트의 수. 작성하기 위해 아직 큐에 있는 데이터는 포함하지 않는다.

## Class: fs.FSWatcher

`fs.watch()`가 반환하는 객체가 이 타입이다.

### watcher.close()

주어진 `fs.FSWatcher`에서 변경사항을 감시하는 것을 멈춘다.

### Event: 'change'

* `event` {String} fs 변경사항의 타입
* `filename` {String} 변경된 파일명 (적절하거나 사용가능하다면)

감시하는 디렉토리나 파일명에서 어떤 변경이 생겼을 때 발생한다.
더 자세한 내용은 [fs.watch](#fs_fs_watch_filename_options_listener)를 봐라.

### Event: 'error'

* `error` {Error object}

오류가 생겼을 때 발생한다.
