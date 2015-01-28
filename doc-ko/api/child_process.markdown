# Child Process

    Stability: 3 - Stable

Node는 `child_process` 모듈로 세 방향의 `popen(3)` 기능을
제공한다.

완전한 넌블락킹 방법으로 자식 프로세스의 `stdin`, `stdout`, `stderr`에
데이터를 스트리밍 하는 것이 가능하다. (일부 프로그램은 내부적으로 라인 버퍼링 I/O
(line-buffered I/O)를 사용한다. 이는 node.js에는 영향을 주지 않지만 자식
프로세스에 보낸 데이터가 즉시 소비되지 않는다는 것을 의미한다.)

자식 프로세스를 생성하려면 `require('child_process').spawn()`나
`require('child_process').fork()`를 사용해라. 각각의 의미는
약간 다른데 아래에서 설명한다.

## Class: ChildProcess

`ChildProcess`는 [EventEmitter][]이다.

자식 프로세스들은 자신들과 연관된 세 가지 스트림 `child.stdin`, `child.stdout`,
`child.stderr`를 항상 가진다. 이 세 스트림은 부모 프로세스의 stdio 스트림을
공유하거나 파이프로 연결될 수 있는 스트림을 구분할 것이다.

ChildProcess 클래스는 직접 사용하도록 만들어진 것이 아니다. 자식 프로세스의
인스턴스를 생성하려면 `spawn()`나 `fork()`를 사용해라.

### Event:  'error'

* `err` {Error Object} 오류.

다음과 같은 상황에 발생한다.

1. 프로세스를 생성(spawn)할 수 없거나
2. 프로세스를 죽일 수 없거나
3. 어떤 이유로든지 자식 프로세스에 메시지 전송이 실패한 경우

오류가 발생한 후에 `exit`-이벤트가 발생할 수도 있고 발생하지 않을 수도 있다는 점을 주의해라. 함수를
실행하려고 두 이벤트를 리스닝하고 있다면 함수가 두 번 호출되지 않도록 해야 한다.

[`ChildProcess#kill()`](#child_process_child_kill_signal)와
[`ChildProcess#send()`](#child_process_child_send_message_sendhandle)도
참고해라.

### Event:  'exit'

* `code` {숫자} 정상적으로 종료되는 경우의 종료코드.
* `signal` {String} 부모가 자식프로세스를 죽일 때 자식 프로세스에 전달하는 신호.

이 이벤트는 자식프로세스가 종료된 후에 발생한다. 프로세스가 정상적으로 종료된다면
`code`는 프로세스의 최종 종료코드이고 정상적으로 종료되지 않았다면 `null`이다.
프로세스가 신호를 받아서 종료되었다면 `signal`는 문자열로 된 신호의 이름이고
신호를 받아서 종료되지 않았다면 `null`이다.

자식 프로세스의 stdio 스트림은 여전히 열려있을 것이다.

노드가 `'SIGINT'`와 `'SIGTERM'`에 대한 신호 핸들러를 만들었으므로 이러한 신호를 받아서는
종료되지(terminate) 않고 빠져나갈(exit) 것이다.

`waitpid(2)`를 봐라.

### Event: 'close'

* `code` {Number} 정상적으로 종료되었다면 종료 코드다.
* `signal` {String} 부모가 자식 프로세스를 종료했다면 자시 프로세스에 전달된 신호이다.

이 이벤트는 자식 프로세스의 stdio 스트림이 모두 종료되었을 때 발생한다. 이 이벤트는
다중 프로세스가 같은 stdio 스트림을 공유할 수도 있으므로 'exit'와는 다르다.

### Event: 'disconnect'

이 이벤트는 부모나 자식의 `.disconnect()` 메서드를 사용한 수에 발생한다. 연결이 끊긴
후에는 더는 메시지를 보낼 수 없고 `.connected`가 false가 된다.

### Event: 'message'

* `message` {Object} 파싱된 JSON 객체나 프리미티브 값
* `sendHandle` {Handle object} Socket이나 Server 객체

`.send(message, [sendHandle])`로 보낸 메시지는 `message` 이벤트로 받는다.

### child.stdin

* {Stream 객체}

자식 프로세스의 `stdin`를 나타내는 `Writable Stream`이다. 자식 프로세스가
이 입력을 읽으려고 기다리고 있다면 `end()`로 해당 스트림을 닫을 때까지 계속되지
않을 것이다.

자식 프로세스의 `stdio[0]`가 `'pipe'`로 설정해서 생성되지 않았다면 이 값은
설정되지 않을 것이다.

`child.stdin`는 `child.stdio[0]`의 단축프로퍼티다. 두 프로퍼티는 같은 객체를
참조하거나 null을 참조할 것이다.

### child.stdout

* {Stream 객체}

자식 프로세스의 `stdout`를 나타내는 `Readable Stream`이다.

자식 프로세스의 `stdio[1]`가 `'pipe'`로 설정해서 생성되지 않았다면 이 값은
설정되지 않을 것이다.

`child.stdout`는 `child.stdio[1]`의 단축프로퍼티다. 두 프로퍼티는 같은 객체를
참조하거나 null을 참조할 것이다.

### child.stderr

* {Stream 객체}

자식 프로세스의 `stderr`를 나타내는 `Readable Stream`이다.

자식 프로세스의 `stdio[2]`가 `'pipe'`로 설정해서 생성되지 않았다면 이 값은
설정되지 않을 것이다.

`child.stderr`는 `child.stdio[2]`의 단축프로퍼티다. 두 프로퍼티는 같은 객체를
참조하거나 null을 참조할 것이다.

### child.stdio

* {Array}

자식 프로세스로의 파이프의 빈약한 배열로 [stdio](#child_process_options_stdio)
옵션에서 `'pipe'`로 설정된
[spawn](#child_process_child_process_spawn_command_args_options)의 위치에
대응된다.
streams 0-2도 각각 ChildProcess.stdin, ChildProcess.stdout,
ChildProcess.stderr로 사용할 수 있다.

아래 예제는 자식의 fd `1`만 파이프로 설정했으므로 부모의 `child.stdio[1]`만 스트림이고
배열의 다른 값은 모두 `null`이다.

    child = child_process.spawn("ls", {
        stdio: [
          0, // 자식에 부모의 stdin을 사용한다
          'pipe', // 자식의 stdout를 부모로 파이프 연결한다
          fs.openSync("err.out", "w") // 자식의 stderr는 파일로 직접 연결한다
        ]
    });

    assert.equal(child.stdio[0], null);
    assert.equal(child.stdio[0], child.stdin);

    assert(child.stdout);
    assert.equal(child.stdio[1], child.stdout);

    assert.equal(child.stdio[2], null);
    assert.equal(child.stdio[2], child.stderr);

### child.pid

* {정수}

자식 프로세스의 PID.

예제:

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    console.log('Spawned child pid: ' + grep.pid);
    grep.stdin.end();

### child.connected

* {Boolean} `.disconnect'가 호출되면 false로 설정한다

`.connected`가 false이면 더는 메시지를 보낼 수 없다.

### child.kill([signal])

* `signal` {문자열}

자식 프로세스에 신호를 보낸다. 아규먼트를 전달하지 않으면 프로세스는 `'SIGTERM'`를 보낼
것이다. 사용할 수 있는 신호 목록은 `signal(7)`를 참고해라.

    var spawn = require('child_process').spawn,
        grep  = spawn('grep', ['ssh']);

    grep.on('close', function (code, signal) {
      console.log('child process terminated due to receipt of signal '+signal);
    });

    // 프로세스에 SIGHUP를 보낸다
    grep.kill('SIGHUP');

신호를 전송할 수 없을 때는 `'error'`를 발생시킬 것이다. 종료된 자식 프로세스에 신호를 보내는 것은
오류가 아니지만 의도치 않은 결과가 될 수 있다. PID(프로세스 ID)가 다른 프로세스에 재할당되었다면
신호는 해당 프로세스로 전달될 것이고 무슨 일이 일어날지 아무도 짐작할 수 없다.

함수의 이름이 `kill`이기는 하지만 자식 프로세스에 전달된 신호가 실제로 자식 프로세스를
죽이지는 않을 것이다. `kill`은 프로세스에 단지 신호를 보낼 뿐이다.

`kill(2)`를 참고해라.

### child.send(message, [sendHandle])

* `message` {객체}
* `sendHandle` {Handle 객체}

`child_process.fork()`를 사용했을 때 `child.send(message, [sendHandle])`를
사용해서 자식에 작성할 수 있고 자식에서는 `'message'` 이벤트로 메시지를 받는다.

예를 들어:

    var cp = require('child_process');

    var n = cp.fork(__dirname + '/sub.js');

    n.on('message', function(m) {
      console.log('PARENT got message:', m);
    });

    n.send({ hello: 'world' });

그리고 자식 스크립트인 `'sub.js'`는 다음과 같을 것이다.

    process.on('message', function(m) {
      console.log('CHILD got message:', m);
    });

    process.send({ foo: 'bar' });

자식에서 `process` 객체는 `send()` 메시지를 가질 것이고 `process`는 채널에서
메시지를 받을 때마다 객체를 발생시킬 것이다.

부모와 자식 모두에서 `send()` 메서드는 동기라는 점을 유념해라. 청크가 큰 데이터를 보내는 것은
권장하지 않는다.(대신 파이프를 사용할 수 있다.
[`child_process.spawn`](#child_process_child_process_spawn_command_args_options)
참고.)

`{cmd: 'NODE_foo'}` 메시지를 보냈을 때 특별한 경우가 있다.
`cmd` 프로퍼티에 `NODE_` 접두사가 있는 모든 메시지는 node 코어에서 사용되는 내부 메시지이므로
`message` 이벤트에서 발생하지 않을 것이다. 접두사가 있는 메시지들은 `internalMessage` 이벤트를
발생시킨다. 이는 별도의 공지 없이 변경되므로 이 기능을 사용하지 말아야 한다는 것을 의미한다.

`child.send()`의 `sendHandle` 옵션은 TCP 서버나 소켓 객체를 다른 프로세스에 보내는
용도이다. 자식 프로세스는 `message` 이벤트의 두 번째 아규먼트로 이 객체를 받을 것이다.

메시지를 보낼 수 없다면 `'error'` 이벤트를 발생시킨다. 예를 들어 자식프로세스가 이미 종료되었다면
보낼 수 없다.

#### Example: sending server object

다음은 서버를 전송하는 예제다.

    var child = require('child_process').fork('child.js');

    // 서버객체를 열고 handle을 전송한다.
    var server = require('net').createServer();
    server.on('connection', function (socket) {
      socket.end('부모가 처리한다');
    });
    server.listen(1337, function() {
      child.send('server', server);
    });

자식프로세스는 다음과 같이 서버 객체를 받는다.

    process.on('message', function(m, server) {
      if (m === 'server') {
        server.on('connection', function (socket) {
          socket.end('자식이 처리한다');
        });
      }
    });

서버는 이제 부모와 자식 사이에서 공유된다. 이는 연결들이 부모와 자식에서 모두 다룰 수
있다는 의미이다.

`dgram` 서버에서 워크플로우는 완전히 같다. 여기서 `connection` 대신 `message` 이벤트를
리스닝하고 `server.listen` 대신 `server.bind`를 사용한다. (현재는 UNIX 플랫폼에서만
지원한다.)

#### Example: sending socket object

다음은 소켓을 전송하는 예제다. 이 예제는 두 자식 프로세스를 생성하고 "특별한" 자식 프로세스에
소켓을 전송해서 VIP인 원격주소 `74.125.127.100`의 연결을 다룬다. 다른 소켓들은 "보통의"
프로세스로 갈 것이다.

    var normal = require('child_process').fork('child.js', ['normal']);
    var special = require('child_process').fork('child.js', ['special']);

    // 서버를 열고 자식 프로세스에 소켓을 전송한다
    var server = require('net').createServer();
    server.on('connection', function (socket) {

      // VIP 이라면
      if (socket.remoteAddress === '74.125.127.100') {
        special.send('socket', socket);
        return;
      }
      // 그냥 일반적인 소켓들
      normal.send('socket', socket);
    });
    server.listen(1337);

`child.js`는 다음과 같다.

    process.on('message', function(m, socket) {
      if (m === 'socket') {
        socket.end('You were handled as a ' + process.argv[2] + ' person');
      }
    });

일단 하나의 소켓을 자식 프로세스에 보내면 부모 프로세서는 소켓이 소멸한 것을 더는
추적할 수 없다. 소켓이 소멸한 것을 나타내기 위해 `.connections` 프로퍼티가
`null`이 된다.
이 경우에 `.maxConnections`를 사용하지 않기를 권장한다.

### child.disconnect()

부모와 자식 간의 IPC 연결을 닫으면서 자식 프로세스에 더는 유지되는 연결이 없도록 안전하게 자식
프로세스를 종료한다. 이 메서드를 호출하면 부모와 자식 모두에서 `.connected` 플래그가
`false`로 설정될 것이고 더는 메시지를 보낼 수 없다.

메시지를 받는 프로세스에 더는 메시지가 없을 때 'disconnect' 이벤트가 발생할 것이고
대부분을 즉각적으로 발생한다.

자식 프로세스가 부모와 연결된 IPC 채널을 하나라도 열고 있다면(예시 `fork()`)
자식 프로세스에서 `process.disconnect()`를 호출할 수도 있다.

## child_process.spawn(command, [args], [options])

* `command` {문자열} 실행할 명령어
* `args` {배열} 문자열 아규먼트의 리스트
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉토리
  * `env` {객체} 환경변수 키-밸류 쌍
  * `stdio` {배열|문자열} 자식의 stdio 설정.
    ([하단]((#child_process_options_stdio))을 참고)
  * `customFds` {배열} **폐기됨** stdio에 사용할 자식 프로세스의
    파일 디스크립터 ([하단]((#child_process_options_customFds))을 참고)
  * `detached` {불리언} 자식이 프로세스 그룹의 리더가 될 것이다.
    ([하단]((#child_process_options_detached))을 참고)
  * `uid` {숫자} 프로세스의 사용자 id를 설정한다. (setuid(2) 참고.)
  * `gid` {숫자} 프로세스의 그룹 id를 설정한다. (setgid(2) 참고.)
* return: {ChildProcess 객체}

명령행 아규먼트 `args`와 함께 주어진 `command`로 새로운 프로세스를 실행한다. 생략할 경우
`args`의 기본값은 비어있는 배열이다.

세 번째 아규먼트는 선택적으로 옵션을 지정하기 위해서 사용하면 기본값은 다음과 같다.

    { cwd: undefined,
      env: process.env
    }

프로세스가 생성되는 워킹 디렉터리를 지정할 때 `cwd`를 사용한다.
지정하지 않으면 기본값으로 현재 워킹디렉터리를 사용한다.

새로운 프로세스에서 사용할 수 있는 환경변수를 지정할 때 `env`를 사용하고
기본적으로 `process.env`이다.

다음은 `ls -lh /usr`를 실행하고 `stdout`, `stderr`와 종료 코드를 잡는 예제다.

    var spawn = require('child_process').spawn,
        ls    = spawn('ls', ['-lh', '/usr']);

    ls.stdout.on('data', function (data) {
      console.log('stdout: ' + data);
    });

    ls.stderr.on('data', function (data) {
      console.log('stderr: ' + data);
    });

    ls.on('close', function (code) {
      console.log('child process exited with code ' + code);
    });

예제: 'ps ax | grep ssh'를 실행하는 아주 정교한 방법.

    var spawn = require('child_process').spawn,
        ps    = spawn('ps', ['ax']),
        grep  = spawn('grep', ['ssh']);

    ps.stdout.on('data', function (data) {
      grep.stdin.write(data);
    });

    ps.stderr.on('data', function (data) {
      console.log('ps stderr: ' + data);
    });

    ps.on('close', function (code) {
      if (code !== 0) {
        console.log('ps process exited with code ' + code);
      }
      grep.stdin.end();
    });

    grep.stdout.on('data', function (data) {
      console.log('' + data);
    });

    grep.stderr.on('data', function (data) {
      console.log('grep stderr: ' + data);
    });

    grep.on('close', function (code) {
      if (code !== 0) {
        console.log('grep process exited with code ' + code);
      }
    });


### options.stdio

간단하게 `stdio` 인자도 다음 문자열 중 하나가 될 수 있다.

* `'pipe'` - `['pipe', 'pipe', 'pipe']`, 이는 기본값이다
* `'ignore'` - `['ignore', 'ignore', 'ignore']`
* `'inherit'` - `[process.stdin, process.stdout, process.stderr]`나 `[0,1,2]`

그렇지 않으면 `child_process.spawn()`의 'stdio' 옵션은 자식 프로세스의 fd에 대응하는
각 인덱스 배열이다. 값은 다음 중 하나이다.

1. `'pipe'` - 자식프로세스와 부모프로세스 간의 파이프를 생성한다.
   파이프의 부모 쪽은 `ChildProcess.stdio[fd]`처럼 `child_process` 객체의 프로퍼티로
   부모에게 노출된다. fd 0 - 2에 대해 생성된 파이프들은 각각 ChildProcess.stdin,
   ChildProcess.stdout, ChildProcess.stderr로 사용할 수도 있다.
2. `'ipc'` - 부모와 자식 간에 메시지/파일 디스크립터를 전달하는 IPC 채널을 생성한다.
   ChildProcess는 많아야 *하나의* IPC stdio 파일 디스크립터를 가진다. 이 옵션을 설정하면
   ChildProcess.send() 메서드가 활성화된다. 자식 프로세스가 이 파일 디스크립터에 JSON
   메시지를 작성한다면 파일디스크립터는 ChildProcess.on('message')를 실행한다. 자식
   프로세스가 Node.js 프로그램이면서 IPC 채널이 있으면 process.send()와
   process.on('message') 를 활성화한다.
3. `'ignore'` - 자식 프로세스에서 이 파일디스크립터를 설정하지 마라. Node는 생성한
   프로세스에 대해 항상 fd 0 - 2를 연다. 이 중에 무시되는 것이 있다면 node는
   `/dev/null`를 열고 자식의 fd에 붙일 것이다.
4. `Stream` 객체 - tty, 파일, 소켓, 자식프로세스에 연결된 파이프를 참조하는 읽거나
   쓰기가 가능한 스트림을 공유한다. 스트림이 의존하는 파일 디스크립터는 자식 프로세스에서
   `stdio` 비열의 인덱스에 대응하는 fd로 복제된다. 스트림은 반드시 의존하는 디스크립터를
   가져야 한다.(`'open'` 이벤트가 발생할 때까지 파일 스트림을 하지 않는다.)
5. 양의 정수 - 이 정숫값은 부모 프로세스에서 현재 열려있는 파일 디스크립터로 변환된다.
   `Stream` 객체가 공유되는 방법과 유사하게 자식 프로세스와 공유된다.
6. `null`, `undefined` - 기본값을 사용한다. stdio fd 0, 1, 2(즉 stdin, stdout,
   stderr)에 대한 파이프를 생성한다. fd 3 이상에 대한 기본값은 `'ignore'`이다.

예제:

    var spawn = require('child_process').spawn;

    // 자식은 부모의 stdio를 사용할 것이다
    spawn('prg', [], { stdio: 'inherit' });

    // stderr만 공유하는 자식프로세스를 생성한다
    spawn('prg', [], { stdio: ['pipe', 'pipe', process.stderr] });

    // startd 방식의 인터페이스를 제공하는 프로그램과 상호작용하기 위해
    // 여분의 fd=4를 연다
    spawn('prg', [], { stdio: ['pipe', null, null, null, 'pipe'] });

### options.detached

`detached` 옵션을 설정하면 자식 프로세스가 새로운 프로세스 그룹의 리더가 될 것이다.
이는 부모가 종료된 후에도 자식 프로세스가 계속해서 동작할 수 있게 한다.

기본적으로 부모는 분리된(detached) 자식프로세스가 종료되길 기다릴 것이다. 부모가
`child`을 기다리지 않게 하려면 `child.unref()` 메서드를 사용해라. 부모의 이벤트 루프는
참조 수에 자식을 포함하지 않을 것이다.

오랫동안 실행되는 프로세스를 분리하고 출력을 파일로 보내는 예제.

     var fs = require('fs'),
         spawn = require('child_process').spawn,
         out = fs.openSync('./out.log', 'a'),
         err = fs.openSync('./out.log', 'a');

     var child = spawn('prg', [], {
       detached: true,
       stdio: [ 'ignore', out, err ]
     });

     child.unref();

오랫동안 동작하는 프로세스를 시작하려고 `detached` 옵션을 사용해도 `stdio` 설정을 부모에
접속하지 않도록 하지 않으면 프로세스는 백그라운드에서 동작하지 않을 것이다. 부모의
`stdio`를 상속받았다면 자식은 제어하는 터미널에 연결된 채로 유지될 것이다.

### options.customFds

자식 프로세스의 stdio에 특정 파일 디스크립터를 지정하는 폐기된 옵션인 `customFds`가
있다. 이 API는 모든 플랫폼에서 사용할 수 있는 것이 아니라서 삭제되었다.
`customFds`를 사용해서 새로운 프로세스의 `[stdin, stdout, stderr]`를 존재하는
스트림으로 후킹할 수 있다. `-1`은 새로운 스트림이 생성되어야 한다는 것을 의미한다.
사용할 때는 위험을 감수해라.

`child_process.exec()`와 `child_process.fork()`도 참고해라.

## child_process.exec(command, [options], callback)

* `command` {문자열} 실행할 명령어로 전달할 아규먼트는 공백으로 구분한다
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉터리
  * `env` {객체} 환경변수 키-밸류 쌍
  * `encoding` {문자열} (기본값: 'utf8')
  * `timeout` {숫자} (기본값: 0)
  * `maxBuffer` {숫자} (기본값: `200*1024`)
  * `killSignal` {문자열} (기본값: 'SIGTERM')
* `callback` {함수} 프로세스가 종료되었을 때 출력과 함께 호출된다
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* Return: ChildProcess 객체

쉘에서 명령어를 실행하고 출력을 버퍼에 넣는다.

    var exec = require('child_process').exec,
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
된다. 오류가 있으면 `error`는 `Error`의 인스턴스가 되고 `error.code`는 자식 프로세스의
종료코드가 되고 `error.signal`은 프로세스를 종료하는 신호로 설정될 것이다.

두 번째 선택적인 아규먼트는 여러 가지 옵션을 지정한다. 기본 옵션은 다음과 같다.

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

## child_process.execFile(file, [args], [options], [callback])

* `file` {문자열} 실행할 프로그램의 파일명
* `args` {배열} 문자열 아규먼트의 목록
* `options` {객체}
  * `cwd` {문자열} 자식 프로세스의 현재 워킹 디렉터리
  * `env` {객체} 환경변수의 키-밸류 쌍
  * `encoding` {문자열} (기본값: 'utf8')
  * `timeout` {숫자} (기본값: 0)
  * `maxBuffer` {숫자} (기본값: 200\*1024)
  * `killSignal` {문자열} (기본값: 'SIGTERM')
* `callback` {함수} 프로세스가 종료되었을 때 출력과 함께 호출된다.
  * `error` {Error}
  * `stdout` {Buffer}
  * `stderr` {Buffer}
* 반환값: ChildProcess 객체

하위 쉘을 실행하는 대신에 지정한 파일을 직접 실행한다는 점을 제외하면
`child_process.exec()`와 유사하다. `child_process.exec`보다
약간 의존적이게 만든다. 이는 같은 옵션을 가진다.


## child_process.fork(modulePath, [args], [options])

* `modulePath` {문자열} 자식프로세스에서 실행될 모듈
* `args` {배열} 문자열 아규먼트의 목록
* `options` {객체}
  * `cwd` {문자열} 자식프로세스의 현재 워킹 디렉토리
  * `env` {객체} 환경변수의 키-밸류 쌍
  * `execPath` {String} 자식 프로세스를 생성하는 데 사용하는 실행 가능한 경로
  * `execArgv` {Array} 실행파일에 전달하는 문자열 인자 목록
    (기본값: `process.execArgv`)
  * `silent` {Boolean} true이면 자식의 stdin, stdout, stderr가 부모로
    파이프로 연결될 것이다. false이면 stdin, stdout, stderr를 부모에서
    상속받는다. 자세한 내용은 `spawn()`의 `stdio`에서 "pipe"와 "inherit" 옵션을 봐라.
    (기본값은 false)
* 반환 값: ChildProcess 객체

이는 Node 프로세스를 생성하기(spawn) 위해 `spawn()` 기능의 특별한 경우이다. 게다가
보통의 ChildProcess 인스턴스에서 모든 메서드를 가지려고 반환된 객체는 내장된
통신 채널을 가진다. 자세한 내용은 `child.send(message, [sendHandle])`를 참고해라.

이러한 자식 노드들도 V8의 완전한 새 인스턴스이다. 새로운 노드마다 최소한 30ms의
구동시간과 10mb의 메모리를 가정해보자. 즉, 수천 개의 노드를 생성할 수 없다.

`options`객체의 `execPath` 프로퍼티는 현재 실행된 `node` 대신 자식을 위한 프로세스를 생성할
수 있게 한다. 이는 조심히 수행되어야 하고 기본적으로 자식프로세스의 `NODE_CHANNEL_FD` 환경변수를
나타내는 fd에 얘기할 것이다. 이 fd의 입력과 출력은 라인으로 구분된 JSON 객체일 것이다.

[EventEmitter]: events.html#events_class_events_eventemitter
