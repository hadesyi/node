# process

<!-- type=global -->

The `process` object is a global object and can be accessed from anywhere.
It is an instance of `EventEmitter`.

`process` 객체는 전역객체이고 어디서나 접근할 수 있다.
이 객체는 `EventEmitter` 인스턴스이다.


## 이벤트: 'exit'

Emitted when the process is about to exit.  This is a good hook to perform
constant time checks of the module's state (like for unit tests).  The main
event loop will no longer be run after the 'exit' callback finishes, so
timers may not be scheduled.

프로세스가 종료될 때 발생한다. 이 이벤트는 모듈의 상태를 상수시간으로 확인하는데 좋은 
훅(hook)이다.(유닛테스트에서처럼) 메인 이벤트루프는 'exit' 콜백이 종료된 후에는 더이상
실행되지 않으므로 타이머도 스케쥴링되지 않을 것이다.

Example of listening for `exit`:

`exit`이벤트 예제:

    process.on('exit', function () {
      process.nextTick(function () {
       console.log('This will not run');
      });
      console.log('About to exit.');
    });

## 이벤트: 'uncaughtException'

Emitted when an exception bubbles all the way back to the event loop. If a
listener is added for this exception, the default action (which is to print
a stack trace and exit) will not occur.

예외가 이벤트루프까지 버블링되었을 때 발생한다. 이 예외에 대한 리스너를 추가하면
기본 동작(스택트레이스를 출력하고 종료한다)은 수행되지 않을 것이다.

Example of listening for `uncaughtException`:

`uncaughtException`이벤트 예제:

    process.on('uncaughtException', function (err) {
      console.log('Caught exception: ' + err);
    });

    setTimeout(function () {
      console.log('This will still run.');
    }, 500);

    // Intentionally cause an exception, but don't catch it.
    nonexistentFunc();
    console.log('This will not run.');

Note that `uncaughtException` is a very crude mechanism for exception
handling.  Using try / catch in your program will give you more control over
your program's flow.  Especially for server programs that are designed to
stay running forever, `uncaughtException` can be a useful safety mechanism.

`uncaughtException`는 세련되지 않은 예외처리 방법이다. 프로그램에서 try / catch를
사용하는 것이 프로그램의 흐름을 더 잘 제어할 수 있을 것이다. 특히 계속해서 실행되어야 하는
서버 프로그램에서 `uncaughtException`는 유용하고 안전한 방법이 될 것이다.


## 신호 이벤트(Signal Events)

<!--type=event-->
<!--name=SIGINT, SIGUSR1, etc.-->

Emitted when the processes receives a Signal. See sigaction(2) for a list of
standard POSIX signal names such as SIGINT, SIGUSR1, etc.

프로세스가 Signal을 받았을 때 밸상한다. SIGINT, SIGUSR1와 같은 표준 POSIX 신호이름의
목록은 sigaction(2)를 봐라.

Example of listening for `SIGINT`:

`SIGINT` 예제:

    // Start reading from stdin so we don't exit.
    process.stdin.resume();

    process.on('SIGINT', function () {
      console.log('Got SIGINT.  Press Control-D to exit.');
    });

An easy way to send the `SIGINT` signal is with `Control-C` in most terminal
programs.

`SIGINT` 신호를 보내는 쉬운 방법은 대부분의 터미털 프로그램에서 `Control-C`를 입력하는 
것이다.


## process.stdout

A `Writable Stream` to `stdout`.

`stdout`에 대한 `Writable Stream`이다.

Example: the definition of `console.log`

예제: `console.log`의 정의

    console.log = function (d) {
      process.stdout.write(d + '\n');
    };

`process.stderr` and `process.stdout` are unlike other streams in Node in
that writes to them are usually blocking.  They are blocking in the case
that they refer to regular files or TTY file descriptors. In the case they
refer to pipes, they are non-blocking like other streams.

`process.stderr`와 `process.stdout`을 쓰기를 할 때 보통 블락킹된다는 점에서 Node의 
다른 스크림과는 다르다. 이 둘은 보통의 파일이나 TTY 파일 디스크립터를 참조할 때
블락킹된다. 파이프를 참조하는 경우에는 다른 스크림처럼 넌블락킹이다.


## process.stderr

A writable stream to stderr.

stderr에 대한 writable stream이다.

`process.stderr` and `process.stdout` are unlike other streams in Node in
that writes to them are usually blocking.  They are blocking in the case
that they refer to regular files or TTY file descriptors. In the case they
refer to pipes, they are non-blocking like other streams.

`process.stderr`와 `process.stdout`는 쓰기를 할 때 보통 블락킹된다는 점에서 Node의 
다른 스크림과는 다르다. 이 둘은 보통의 파일이나 TTY 파일 디스크립터를 참조할 때
블락킹된다. 파이프를 참조하는 경우에는 다른 스크림처럼 넌블락킹이다.


## process.stdin

A `Readable Stream` for stdin. The stdin stream is paused by default, so one
must call `process.stdin.resume()` to read from it.

stdin에 대한 `Readable Stream`이다. stdin 스트림은 기본적으로 멈추기 때문에 stdin에서
읽으려면 `process.stdin.resume()`를 호출해야 한다.


Example of opening standard input and listening for both events:

표준 입력을 열고 두 이벤트를 리스닝하는 예제:

    process.stdin.resume();
    process.stdin.setEncoding('utf8');

    process.stdin.on('data', function (chunk) {
      process.stdout.write('data: ' + chunk);
    });

    process.stdin.on('end', function () {
      process.stdout.write('end');
    });


## process.argv

An array containing the command line arguments.  The first element will be
'node', the second element will be the name of the JavaScript file.  The
next elements will be any additional command line arguments.

커맨드라인 아규먼트를 담고 있는 배열이다. 첫 엘리먼트는 'node'일 것이고 두 번째 
엘리먼트는 자바스트립트 파일명이 될 것이다. 다음 엘리먼트들은 추가적인 커맨드라인
아규먼트일 것이다.

    // print process.argv
    process.argv.forEach(function (val, index, array) {
      console.log(index + ': ' + val);
    });

This will generate:

다음과 같이 출력된다:

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


## process.execPath

This is the absolute pathname of the executable that started the process.

프로세스가 시작되는 실행가능한 절대경로명이다.

Example:

예제:

    /usr/local/bin/node


## process.chdir(directory)

Changes the current working directory of the process or throws an exception if that fails.

프로세스의 현재 워킹디렉토리를 바꾸거나 바꾸는데 실패할 경우 예외를 던진다.

    console.log('Starting directory: ' + process.cwd());
    try {
      process.chdir('/tmp');
      console.log('New directory: ' + process.cwd());
    }
    catch (err) {
      console.log('chdir: ' + err);
    }



## process.cwd()

Returns the current working directory of the process.

프로세스의 현재 워킹디렉토리를 리턴한다.

    console.log('Current directory: ' + process.cwd());


## process.env

An object containing the user environment. See environ(7).

사용자의 환경변수를 담고 있는 객체다. environ(7)를 봐라.


## process.exit([code])

Ends the process with the specified `code`.  If omitted, exit uses the
'success' code `0`.

지정한 `code`로 프로세스를 종료한다. `code`를 생략하면 'success' 코드 `0`을
사용해서 종료한다.

To exit with a 'failure' code:

'failure' 코드로 종료하려면:

    process.exit(1);

The shell that executed node should see the exit code as 1.

node를 실행한 쉘은 1을 종료코드로 간주할 것이다.


## process.getgid()

Gets the group identity of the process. (See getgid(2).)
This is the numerical group id, not the group name.

프로세스의 그룹식별자를 얻는다.(getgid(2)를 봐라.)
이는 그룹 이름이 아니라 숫자로 된 그룹 id 이다.

    console.log('Current gid: ' + process.getgid());


## process.setgid(id)

Sets the group identity of the process. (See setgid(2).)  This accepts either
a numerical ID or a groupname string. If a groupname is specified, this method
blocks while resolving it to a numerical ID.

프로세스의 그룹 식별자를 설정한다.(setgid(2)를 봐라.) 이 함수는 숫자로 된 ID나 문자열로 된
그룹명을 모두 받아들인다. 그룹명을 지정하면 이 메서드가 그룹명을 숫자로된 ID로 처리할 때까지
블락킹한다.

    console.log('Current gid: ' + process.getgid());
    try {
      process.setgid(501);
      console.log('New gid: ' + process.getgid());
    }
    catch (err) {
      console.log('Failed to set gid: ' + err);
    }


## process.getuid()

Gets the user identity of the process. (See getuid(2).)
This is the numerical userid, not the username.

프로세스의 사용자 식별자를 얻는다.(getuid(2)를 봐라.)
이는 사용자명이 아니라 숫자로된 userid이다.

    console.log('Current uid: ' + process.getuid());


## process.setuid(id)

Sets the user identity of the process. (See setuid(2).)  This accepts either
a numerical ID or a username string.  If a username is specified, this method
blocks while resolving it to a numerical ID.

프로세스의 사용자 식별자를 설정한다. (setuid(2)를 봐라.) 이는 숫자로된 ID와 문자열로 된
사용자명을 모두 받아들인다. 사용자명을 지정하면 이 메서드가 사용자명을 숫자로 된 ID로 
처리할 때까지 블락킹한다.

    console.log('Current uid: ' + process.getuid());
    try {
      process.setuid(501);
      console.log('New uid: ' + process.getuid());
    }
    catch (err) {
      console.log('Failed to set uid: ' + err);
    }


## process.version

A compiled-in property that exposes `NODE_VERSION`.

`NODE_VERSION`으로 노출된 컴파일된 프로퍼티이다.

    console.log('Version: ' + process.version);

## process.versions

A property exposing version strings of node and its dependencies.

node와 의존성에 대한 버전 문자열을 노출하는 프로퍼티이다.

    console.log(process.versions);

Will output:

다음과 같이 출력될 것이다:

    { node: '0.4.12',
      v8: '3.1.8.26',
      ares: '1.7.4',
      ev: '4.4',
      openssl: '1.0.0e-fips' }


## process.installPrefix

A compiled-in property that exposes `NODE_PREFIX`.

`NODE_PREFIX`으로 노출된 컴파일된 프로퍼티이다.

    console.log('Prefix: ' + process.installPrefix);


## process.kill(pid, [signal])

Send a signal to a process. `pid` is the process id and `signal` is the
string describing the signal to send.  Signal names are strings like
'SIGINT' or 'SIGUSR1'.  If omitted, the signal will be 'SIGTERM'.
See kill(2) for more information.

프로세스에 신호를 보낸다. `pid`는 프로세스 id이고 `signal`은 보내려는 신호를
설명하는 문자열이다. 신호이름은 'SIGINT'나 'SIGUSR1'같은 문자열이다. `signal`을
생략하면 'SIGTERM'가 될 것이다. 더 자세한 내용은 kill(2)를 봐라.

Note that just because the name of this function is `process.kill`, it is
really just a signal sender, like the `kill` system call.  The signal sent
may do something other than kill the target process.

이 함수의 이름이 `process.kill`이므로 실제로 `kill` 시스템 호출처럼 단순히 신호 
전송자이다. 보낸 신호는 타겟 신호를 죽이는 일이외에 다른 일을 할 것이다.

Example of sending a signal to yourself:

자신에게 신호를 보내는 예제:

    process.on('SIGHUP', function () {
      console.log('Got SIGHUP signal.');
    });

    setTimeout(function () {
      console.log('Exiting.');
      process.exit(0);
    }, 100);

    process.kill(process.pid, 'SIGHUP');


## process.pid

The PID of the process.

프로세스의 PID.

    console.log('This process is pid ' + process.pid);

## process.title

Getter/setter to set what is displayed in 'ps'.

'ps'에서 표시될 어떻게 표시되는 지에 대한 Getter와 Setter


## process.arch

What processor architecture you're running on: `'arm'`, `'ia32'`, or `'x64'`.

어떤 프로세스 아키텍쳐에서 실행되고 있는지 보여준다.: `'arm'`, `'ia32'`, `'x64'`.

    console.log('This processor architecture is ' + process.arch);


## process.platform

What platform you're running on. `'linux2'`, `'darwin'`, etc.

어떤 플랫폼에서 실행되고 있는지 보여준다. `'linux2'`, `'darwin'`, 등

    console.log('This platform is ' + process.platform);


## process.memoryUsage()

Returns an object describing the memory usage of the Node process
measured in bytes.

Node 프로세스의 메모리 사용량을 바이트로 나타내서 보여주는 객체를 리턴한다.

    var util = require('util');

    console.log(util.inspect(process.memoryUsage()));

This will generate:

다음과 같이 출력될 것이다:

    { rss: 4935680,
      heapTotal: 1826816,
      heapUsed: 650472 }

`heapTotal` and `heapUsed` refer to V8's memory usage.

`heapTotal`와 `heapUsed`는 V8의 메모리 사용량을 참조한다.


## process.nextTick(callback)

On the next loop around the event loop call this callback.
This is *not* a simple alias to `setTimeout(fn, 0)`, it's much more
efficient.

이벤트 루프의 다음 번 루프에서 이 callback을 호출한다.
이는 단순히 `setTimeout(fn, 0)`에 대한 별칭이 *아니라* 훨씬 더 효율적이다.

    process.nextTick(function () {
      console.log('nextTick callback');
    });


## process.umask([mask])

Sets or reads the process's file mode creation mask. Child processes inherit
the mask from the parent process. Returns the old mask if `mask` argument is
given, otherwise returns the current mask.

프로세스의 파일 모드 생성 마스크를 설정하거나 읽는다. 자식 프로세스는 부모 프로세스에서
이 마스크를 상속받는다. `mask`아규먼트를 전달하면 이전의 마스크를 리턴하고 `mask`아규먼트를 
전달하지 않으면 현재 마스크를 리턴한다.

    var oldmask, newmask = 0644;

    oldmask = process.umask(newmask);
    console.log('Changed umask from: ' + oldmask.toString(8) +
                ' to ' + newmask.toString(8));


## process.uptime()

Number of seconds Node has been running.

Node가 실행되고 있는 시간을 초단위로 나타낸다.
