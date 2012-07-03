# Child Process

    Stability: 3 - Stable

Node provides a tri-directional `popen(3)` facility through the
`child_process` module.

It is possible to stream data through a child's `stdin`, `stdout`, and
`stderr` in a fully non-blocking way.

To create a child process use `require('child_process').spawn()` or
`require('child_process').fork()`.  The semantics of each are slightly
different, and explained below.

    안정성: 3 - Stable

Node는 `child_process` 모듈로 세 가지 방향의 `popen(3)` 기능을 
제공한다.

완전한 넌블락킹 방ㅃ으로 자식 프로세스의 `stdin`, `stdout`, `stderr`에 
데이터를 스트리밍하는 것이 가능하다.

자식 프로세스를 생성하려면 `require('child_process').spawn()`나 
`require('child_process').fork()`를 사용해라. 각각의 의미는 
약간 다른데 아래에서 설명한다.

## Class: ChildProcess

`ChildProcess` is an `EventEmitter`.

Child processes always have three streams associated with them. `child.stdin`,
`child.stdout`, and `child.stderr`.  These may be shared with the stdio
streams of the parent process, or they may be separate stream objects
which can be piped to and from.

The ChildProcess class is not intended to be used directly.  Use the
`spawn()` or `fork()` methods to create a Child Process instance.

`ChildProcess`는 `EventEmitter`이다..

자식 프로세스들은 자신들과 연관된 세 가지 스트림 `child.stdin`, `child.stdout`, 
`child.stderr`를 항상 가진다. 이 세 스트림은 부모 프로세스의 stdio 스트림을 
공유하거나 파이프로 연결될 수 있는 스크림을 구분할 것이다.

ChildProcess 클래스는 직접 사용하도록 만들어 진 것이 아니다. 자식 프로세스의 
인스턴스를 생성하려면 `spawn()`나 `fork()`를 사용해라.

### Event:  'exit'

* `code` {Number} the exit code, if it exited normally.
* `signal` {String} the signal passed to kill the child process, if it
  was killed by the parent.

This event is emitted after the child process ends. If the process terminated
normally, `code` is the final exit code of the process, otherwise `null`. If
the process terminated due to receipt of a signal, `signal` is the string name
of the signal, otherwise `null`.

See `waitpid(2)`.

* `code` {숫자} 정상적으로 종료되는 경우의 종료코드.
* `signal` {String} 부모가 자식프로세스를 죽일 때 자식 프로세스에 전달하는 신호.

이 이벤트는 자식프로세스가 종료된 후에 발생한다. 프로세스가 정상적으로 종료된다면 
`code`는 프로세스의 최종 종료코드이고 정상적으로 종료되지 않았다면 `null`이다. 
프로세스가 신호를 받아서 종료되었다면 `signal`는 문자열로 된 신호의 이름이고 
신호를 받아서 종료되지 않았다면 `null`이다.

`waitpid(2)`를 봐라.

### child.stdin

* {Stream object}

A `Writable Stream` that represents the child process's `stdin`.
Closing this stream via `end()` often causes the child process to terminate.

If the child stdio streams are shared with the parent, then this will
not be set.

* {Stream 객체}

자식 프로세스의 `stdin`를 나타내는 `Writable Stream`이다. `end()`로 
이 스트림을 닫으면 종종 자식 프로세스가 종료되기도 한다.

자식 프로세스의 stdio 스트림이 부모 프로세스와 공유한다면 이 값은 설정되지 
않을 것이다.

### child.stdout

* {Stream object}

A `Readable Stream` that represents the child process's `stdout`.

If the child stdio streams are shared with the parent, then this will
not be set.

* {Stream 객체}

자식 프로세스의 `stdout`를 나타내는 `Readable Stream`이다. 

자식 프로세스의 stdio 스트림이 부모 프로세스와 공유한다면 이 값은 설정되지 
않을 것이다.

### child.stderr

* {Stream object}

A `Readable Stream` that represents the child process's `stderr`.

If the child stdio streams are shared with the parent, then this will
not be set.

* {Stream 객체}

자식 프로세스의 `stderr`를 나타내는 `Readable Stream`이다. 

자식 프로세스의 stdio 스트림이 부모 프로세스와 공유한다면 이 값은 설정되지 
않을 것이다.

### child.pid

* {Integer}

The PID of the child process.

Example:

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Spawned child pid: ' + grep.pid);
    grep.stdin.end();

* {정수}

자식 프로세스의 PID.

예제:

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Spawned child pid: ' + grep.pid);
    grep.stdin.end();

### child.kill([signal])

* `signal` {String}

Send a signal to the child process. If no argument is given, the process will
be sent `'SIGTERM'`. See `signal(7)` for a list of available signals.

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('exit', function (code, signal) {
      console.log('child process terminated due to receipt of signal '+signal);
    });

    // send SIGHUP to process
    grep.kill('SIGHUP');

Note that while the function is called `kill`, the signal delivered to the child
process may not actually kill it.  `kill` really just sends a signal to a process.

See `kill(2)`

* `signal` {문자열}

자식 프로세스에 신호를 보낸다. 아규먼트를 전달하지 않으면 프로세스는 `'SIGTERM'`를 보낼 
것이다. 사용할 수 있는 신호 목록은 `signal(7)`를 참고해라.

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('exit', function (code, signal) {
      console.log('child process terminated due to receipt of signal '+signal);
    });

    // 프로세스에 SIGHUP를 보낸다
    grep.kill('SIGHUP');

함수의 이름이 `kill`이기는 하지만 자식 프로세스에 전달된 신호가 실제로 자식 프로세스를 
죽이지는 않을 것이다. `kill`은 프로세스에 단지 신호를 보낼 뿐이다.

`kill(2)`를 참고해라.

### child.send(message, [sendHandle])

* `message` {Object}
* `sendHandle` {Handle object}

Send a message (and, optionally, a handle object) to a child process.

See `child_process.fork()` for details.

* `message` {객체}
* `sendHandle` {Handle 객체}

자식 프로세스에 메시지(선택적으로 핸들 객체와 함께)를 보낸다.

자세한 내용은 `child_process.fork()`를 참고해라..

## child_process.spawn(command, [args], [options])

* `command` {String} The command to run
* `args` {Array} List of string arguments
* `options` {Object}
  * `cwd` {String} Current working directory of the child process
  * `customFds` {Array} **Deprecated** File descriptors for the child to use
    for stdio.  (See below)
  * `env` {Object} Environment key-value pairs
* return: {ChildProcess object}

Launches a new process with the given `command`, with  command line arguments in `args`.
If omitted, `args` defaults to an empty Array.

The third argument is used to specify additional options, which defaults to:

    { cwd: undefined,
      env: process.env
    }

`cwd` allows you to specify the working directory from which the process is spawned.
Use `env` to specify environment variables that will be visible to the new process.

Example of running `ls -lh /usr`, capturing `stdout`, `stderr`, and the exit code:

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
    });

    ls.on('exit', function (code) {
      console.log('child process exited with code ' + code);
    });


Example: A very elaborate way to run 'ps ax | grep ssh'

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      console.log('ps stderr: ' + data);
    });

    ps.on('exit', function (code) {
      if (code !== 0) {
        console.log('ps process exited with code ' + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      console.log(data);
    });

    grep.stderr.on('data', function (data) {
      console.log('grep stderr: ' + data);
    });

    grep.on('exit', function (code) {
      if (code !== 0) {
        console.log('grep process exited with code ' + code);
      }
    });


Example of checking for failed exec:

    var spawn = require('child_process').spawn,
        child = spawn('bad_command');

    child.stderr.setEncoding('utf8');
    child.stderr.on('data', function (data) {
      if (/^execvp\(\)/.test(data)) {
        console.log('Failed to start child process.');
      }
    });

Note that if spawn receives an empty options object, it will result in
spawning the process with an empty environment rather than using
`process.env`. This due to backwards compatibility issues with a deprecated
API.

There is a deprecated option called `customFds` which allows one to specify
specific file descriptors for the stdio of the child process. This API was
not portable to all platforms and therefore removed.
With `customFds` it was possible to hook up the new process' `[stdin, stdout,
stderr]` to existing streams; `-1` meant that a new stream should be created.
Use at your own risk.

There are several internal options. In particular `stdinStream`,
`stdoutStream`, `stderrStream`. They are for INTERNAL USE ONLY. As with all
undocumented APIs in Node, they should not be used.

See also: `child_process.exec()` and `child_process.fork()`

---

* `command` {문자열} 실행할 명령어
* `args` {배열} 문자열 아규먼트의 리스트
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉토리
  * `customFds` {배열} **폐기됨** stdio에 사용할 자식 프로세스의 
    파일 디스크립터 (하단을 참고)
  * `env` {객체} 환경변수 키-밸류 쌍
* return: {ChildProcess 객체}

명령행 아규먼트 `args`와 함께 주어진 `command`로 새로운 프로세스를 실행한다. 생략할 경우 
`args`의 기본값은 비어있는 배열이다.

세 번째 아규먼트는 선택적으로 옵션을 지정하기 위해서 사용하면 기본값은 다음과 같다.

    { cwd: undefined,
      env: process.env
    }

`cwd`로 프로세스가 생성되는(spawn)가 생성되는 워킹 디렉토리를 지정할 수 있다.
`env`를 사용해서 새로운 프로세스에서 볼 수 있는 환경 변수를 지정한다.

다음은 `ls -lh /usr`를 실행하고 `stdout`, `stderr`와 종료 코드를 잡는 예제다.

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
    });

    ls.on('exit', function (code) {
      console.log('child process exited with code ' + code);
    });

예제: 'ps ax | grep ssh'를 실행하는 아주 정교한 방법.

    var util  = require('util'),
        spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      console.log('ps stderr: ' + data);
    });

    ps.on('exit', function (code) {
      if (code !== 0) {
        console.log('ps process exited with code ' + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      console.log(data);
    });

    grep.stderr.on('data', function (data) {
      console.log('grep stderr: ' + data);
    });

    grep.on('exit', function (code) {
      if (code !== 0) {
        console.log('grep process exited with code ' + code);
      }
    });


실대한 실행을 확인하는 예제.

    var spawn = require('child_process').spawn,
        child = spawn('bad_command');

    child.stderr.setEncoding('utf8');
    child.stderr.on('data', function (data) {
      if (/^execvp\(\)/.test(data)) {
        console.log('Failed to start child process.');
      }
    });

spawn이 비어 있는 옵션 객체를 받으면 `process.env`를 사용하는 대신 비어있는 
환경변수를 가진 프로세스를 생성(spawn)할 것이다. 이는 폐기된 API와 관련된 하위 
호환성때문에 존재하는 것이다.

자식 프로세스의 stdio에 특정 파일 디스크립터를 지정하는 폐기된 옵션인 `customFds`가 
있다. 이 API는 모든 플랫폼에서 사용할 수 있는 것이 아니라서 삭제되었다.
`customFds`를 사용해서 새로운 프로세스의 `[stdin, stdout, stderr]`를 존재하는 
스트림으로 후킹할 수 있다. `-1`은 새로운 스트림이 생성되어야 한다는 것을 의미한다.
사용할 때는 위험을 감수해라.

여러가지 내부 옵션이 존재한다. 특히 `stdinStream`, `stdoutStream`, 
`stderrStream`가 있다. 이 옵션들은 내부적으로만 사용된다. Node에서 문서화되지 않은 
모든 API는 사용하지 말아야 한다.

`child_process.exec()`와 `child_process.fork()`도 참고해라.

## child_process.exec(command, [options], callback)

* `command` {String} The command to run, with space-separated arguments
* `options` {Object}
  * `cwd` {String} Current working directory of the child process
  * `customFds` {Array} **Deprecated** File descriptors for the child to use
    for stdio.  (See below)
  * `env` {Object} Environment key-value pairs
  * `encoding` {String} (Default: 'utf8')
  * `timeout` {Number} (Default: 0)
  * `maxBuffer` {Number} (Default: 200*1024)
  * `killSignal` {String} (Default: 'SIGTERM')
* `callback` {Function} called with the output when process terminates
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* Return: ChildProcess object

Runs a command in a shell and buffers the output.

    var util = require('util'),
        exec = require('child_process').exec,
        child;

    child = exec('cat *.js bad_file | wc -l',
      function (error, stdout, stderr) {
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
        if (error !== null) {
          console.log('exec error: ' + error);
        }
    });

The callback gets the arguments `(error, stdout, stderr)`. On success, `error`
will be `null`.  On error, `error` will be an instance of `Error` and `err.code`
will be the exit code of the child process, and `err.signal` will be set to the
signal that terminated the process.

There is a second optional argument to specify several options. The
default options are

    { encoding: 'utf8',
      timeout: 0,
      maxBuffer: 200*1024,
      killSignal: 'SIGTERM',
      cwd: null,
      env: null }

If `timeout` is greater than 0, then it will kill the child process
if it runs longer than `timeout` milliseconds. The child process is killed with
`killSignal` (default: `'SIGTERM'`). `maxBuffer` specifies the largest
amount of data allowed on stdout or stderr - if this value is exceeded then
the child process is killed.

* `command` {문자열} 실행할 명령어로 전달할 아규먼트는 공백으로 구분한다
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉토리
  * `customFds` {배열} **폐기됨** stdio에 사용할 자식 프로세스의 
    파일 디스크립터 (하단을 참고)
  * `env` {객체} 환경변수 키-밸류 쌍
  * `encoding` {문자열} (기본값: 'utf8')
  * `timeout` {숫자} (기본값: 0)
  * `maxBuffer` {숫자} (기본값: 200*1024)
  * `killSignal` {문자열} (기본값: 'SIGTERM')
* `callback` {함수} 프로세스가 종료되었을 대 출력과 함께 호출된다
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* Return: ChildProcess 객체

쉘에서 명령어를 실행하고 출력을 버퍼에 넣는다.

    var util = require('util'),
        exec = require('child_process').exec,
        child;

    child = exec('cat *.js bad_file | wc -l',
      function (error, stdout, stderr) {
        console.log('stdout: ' + stdout);
        console.log('stderr: ' + stderr);
        if (error !== null) {
          console.log('exec error: ' + error);
        }
    });

콜백은 `(error, stdout, stderr)` 아규먼트를 받는다. 성공했을 때 `error`는 `null`이 
된다. 오류가 있을 경우 `error`는 `Error`의 인스턴스가 되고 `err.code`는 자식 프로세스의 
종료코드가 되고 `err.signal`은 프로세스를 종료하는 신호로 설정될 것이다. 

두번째 선택적인 아규먼트는 여러가지 옵션을 지정한다. 기본 옵션은 다음과 같다.

    { encoding: 'utf8',
      timeout: 0,
      maxBuffer: 200*1024,
      killSignal: 'SIGTERM',
      cwd: null,
      env: null }

`timeout`이 0보다 크면 `timeout` 밀리초보다 오래 실행되는 경우 자식 프로세스를 
죽일 것이다. 자식 프로세스는 `killSignal` (기본값: `'SIGTERM'`) 신호로 죽는다. 
`maxBuffer`는 stdout이나 stderr에서 사용할 수 있는 가장 큰 데이터의 양을 
지정한다. `maxBuffer`갑을 초과하면 자식 프로세스는 죽을 것이다.

## child_process.execFile(file, args, options, callback)

* `file` {String} The filename of the program to run
* `args` {Array} List of string arguments
* `options` {Object}
  * `cwd` {String} Current working directory of the child process
  * `customFds` {Array} **Deprecated** File descriptors for the child to use
    for stdio.  (See below)
  * `env` {Object} Environment key-value pairs
  * `encoding` {String} (Default: 'utf8')
  * `timeout` {Number} (Default: 0)
  * `maxBuffer` {Number} (Default: 200*1024)
  * `killSignal` {String} (Default: 'SIGTERM')
* `callback` {Function} called with the output when process terminates
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* Return: ChildProcess object

This is similar to `child_process.exec()` except it does not execute a
subshell but rather the specified file directly. This makes it slightly
leaner than `child_process.exec`. It has the same options.

* `file` {문자열} 실행할 프로그램의 파일명
* `args` {배열} 문자열 아규먼트의 목록
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉토리
  * `customFds` {배열} **폐기됨** stdio에 사용할 자식 프로세스의 
    파일 디스크립터 (하단을 참고)
  * `env` {객체} 환경변수의 키-밸류 쌍
  * `encoding` {문자열} (기본값: 'utf8')
  * `timeout` {숫자} (기본값: 0)
  * `maxBuffer` {숫자} (기본값: 200*1024)
  * `killSignal` {문자열} (기본값: 'SIGTERM')
* `callback` {함수} 프로세스가 종료되었을 때 출력과 함께 호출된다.
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* Return: ChildProcess 객체

하위 쉘을 실행하는 대신에 지정한 파일을 직접 실행한다는 점을 제외하면 
`child_process.exec()`와 유사하다. `child_process.exec`보다 
약간 의존적이게 만든다. 이는 같은 옵션을 가진다.


## child_process.fork(modulePath, [args], [options])

* `modulePath` {String} The module to run in the child
* `args` {Array} List of string arguments
* `options` {Object}
  * `cwd` {String} Current working directory of the child process
  * `customFds` {Array} **Deprecated** File descriptors for the child to use
    for stdio.  (See below)
  * `env` {Object} Environment key-value pairs
  * `encoding` {String} (Default: 'utf8')
  * `timeout` {Number} (Default: 0)
* Return: ChildProcess object

This is a special case of the `spawn()` functionality for spawning Node
processes. In addition to having all the methods in a normal ChildProcess
instance, the returned object has a communication channel built-in. The
channel is written to with `child.send(message, [sendHandle])` and messages
are received by a `'message'` event on the child.

For example:

    var cp = require('child_process');

    var n = cp.fork(__dirname + '/sub.js');

    n.on('message', function(m) {
      console.log('PARENT got message:', m);
    });

    n.send({ hello: 'world' });

And then the child script, `'sub.js'` might look like this:

    process.on('message', function(m) {
      console.log('CHILD got message:', m);
    });

    process.send({ foo: 'bar' });

In the child the `process` object will have a `send()` method, and `process`
will emit objects each time it receives a message on its channel.

By default the spawned Node process will have the stdin, stdout, stderr
associated with the parent's.

These child Nodes are still whole new instances of V8. Assume at least 30ms
startup and 10mb memory for each new Node. That is, you cannot create many
thousands of them.

The `sendHandle` option to `child.send()` is for sending a handle object to
another process. Child will receive the handle as as second argument to the
`message` event. Here is an example of sending a handle:

    var server = require('net').createServer();
    var child = require('child_process').fork(__dirname + '/child.js');
    // Open up the server object and send the handle.
    server.listen(1337, function() {
      child.send({ server: true }, server._handle);
    });

Here is an example of receiving the server handle and sharing it between
processes:

    process.on('message', function(m, serverHandle) {
      if (serverHandle) {
        var server = require('net').createServer();
        server.listen(serverHandle);
      }
    });

* `modulePath` {문자열} 자식프로세스에서 실행될 모듈
* `args` {배열} 문자열 아규먼트의 목록
* `options` {객체}
  * `cwd` {문자열} 자식프로세스의 현재 워킹 디렉토리
  * `customFds` {배열} **폐기됨** stdio에 사용할 자식 프로세스의 
    파일 디스크립터 (하단을 참고)
  * `env` {객체} 환경변수의 키-밸류 쌍
  * `encoding` {문자열} (기본값: 'utf8')
  * `timeout` {숫자} (기본값: 0)
* Return: ChildProcess 객체

이는 Node 프로세스를 생성하기(spawn) 위해 `spawn()` 기능의 틀별한 경우이다. 게다가 
보통의 ChildProcess 인스턴스에서 모든 메서드를 가지려고 반환된 객체는 내장된 
통신 채널을 가진다. 채널은 `child.send(message, [sendHandle])`로 작성되고 
메시지는 자식 프로세스에서 `'message'` 이벤트로 받는다.

예를 들어:

    var cp = require('child_process');

    var n = cp.fork(__dirname + '/sub.js');

    n.on('message', function(m) {
      console.log('PARENT got message:', m);
    });

    n.send({ hello: 'world' });

그리고 자식 스크립트 `'sub.js'`는 다음과 같을 것이다.

    process.on('message', function(m) {
      console.log('CHILD got message:', m);
    });

    process.send({ foo: 'bar' });

자식 프로세스에서 `process` 객체는 `send()` 메서드를 가질 것이고 `process`는 
채널에서 메시지를 받을 때마다 객체를 발생시킬 것이다.

기본적으로 생성된(spawn) Node 프로세스는 부모와 연결된 stdin, stdout, stderr를 
가질 것이다.

이러한 자식 노드들도 V8의 완전한 새 인스턴스이다. 새로운 각 노드마다 최소한 30ms의 
구동시간과 10mb의 메모리를 가정해보자. 즉, 수천 개의 노드를 생성할 수 없다.

`child.send()`의 `sendHandle` 옵션은 다른 프로세스에 핸들 객체를 전달하기 위함이다. 
자식 프로세스는 `message` 이벤트의 두번째 아규먼트로 핸들을 받을 것이다. 다음은 
핸들을 보내는 예제이다.

    var server = require('net').createServer();
    var child = require('child_process').fork(__dirname + '/child.js');
    // 서버 객체를 열고 핸들을 보낸다.
    server.listen(1337, function() {
      child.send({ server: true }, server._handle);
    });

다음은 서버 핸들을 받고 프로세스들 사이에서 핸들을 공유하는 예제다.

    process.on('message', function(m, serverHandle) {
      if (serverHandle) {
        var server = require('net').createServer();
        server.listen(serverHandle);
      }
    });



