# process

<!-- type=global -->

`process` 객체는 전역객체이고 어디서나 접근할 수 있다.
이 객체는 [EventEmitter][] 인스턴스이다.


## Event: 'exit'

프로세스가 종료될 때 발생한다. 이 이벤트는 모듈의 상태를 상수시간으로 확인하는데 좋은
훅(hook)이다.(유닛테스트에서처럼) 메인 이벤트루프는 'exit' 콜백이 종료된 후에는 더이상
실행되지 않으므로 타이머도 스케쥴링되지 않을 것이다.

`exit`이벤트 예제:

    process.on('exit', function() {
      setTimeout(function() {
        console.log('This will not run');
      }, 0);
      console.log('About to exit.');
    });


## Event: 'uncaughtException'

예외가 이벤트루프까지 버블링되었을 때 발생한다. 이 예외에 대한 리스너를 추가하면
기본 동작(스택트레이스를 출력하고 종료한다)은 수행되지 않을 것이다.

`uncaughtException`이벤트 예제:

    process.on('uncaughtException', function(err) {
      console.log('Caught exception: ' + err);
    });

    setTimeout(function() {
      console.log('This will still run.');
    }, 500);

    // 의도적인 예외로 예외를 처리하지 않는다.
    nonexistentFunc();
    console.log('This will not run.');

`uncaughtException`는 세련되지 않은 예외처리 방법이이고 차후에는 제거될 것이다.

`uncaughtException`를 사용하지 말고 대신 [domains](domain.html)를 사용해라.
`uncaughtException`를 사용한다면 처리하지 않은 예외마다 어플리케이션을 리스타트해라!

`uncaughtException`를 node.js의 `On Error Resume Next`로 사용하지 *마라*.
처리하지 않은 예외는 어플리케이션(그리고 node.js 확장)이 정의되지 않은 상태에 있음을 의미한다.
맹목적으로 복구하면 *무슨 일이든* 발생할 수 있다.

시스템을 업그레이드 하는 중에 파워코드가 빠졌을 때의 복구를 생각해보자.
열의 아홉은 아무일도 발생하지 않을 것이다. 하지만 10번째는 시스템이 깨진다.

주의해라.

## Signal Events

<!--type=event-->
<!--name=SIGINT, SIGUSR1, etc.-->

프로세스가 Signal을 받았을 때 발상한다. SIGINT, SIGUSR1와 같은 표준 POSIX 신호이름의
목록은 sigaction(2)를 봐라.

`SIGINT` 예제:

    // stdin를 읽기 시작하므로 종료되지 않는다.
    process.stdin.resume();

    process.on('SIGINT', function() {
      console.log('Got SIGINT.  Press Control-D to exit.');
    });

`SIGINT` 신호를 보내는 쉬운 방법은 대부분의 터미털 프로그램에서 `Control-C`를 입력하는
것이다.


## process.stdout

`stdout`에 대한 `Writable Stream`이다.

예제: `console.log`의 정의

    console.log = function(d) {
      process.stdout.write(d + '\n');
    };

`process.stderr`와 `process.stdout`을 쓰기를 할 때 보통 블락킹된다는 점에서 Node의
다른 스크림과는 다르다. 이 둘은 보통의 파일이나 TTY 파일 디스크립터를 참조할 때
블락킹된다. 파이프를 참조하는 경우에는 다른 스크림처럼 넌블락킹이다.

Node가 TTY 컨텐스트로 실행되는지 확인하려면 `process.stderr`, `process.stdout`,
`process.stdin`의 `isTTY` 프로퍼티를 읽어라.

    $ node -p "Boolean(process.stdin.isTTY)"
    true
    $ echo "foo" | node -p "Boolean(process.stdin.isTTY)"
    false

    $ node -p "Boolean(process.stdout.isTTY)"
    true
    $ node -p "Boolean(process.stdout.isTTY)" | cat
    false

자세한 내용은 [the tty docs](tty.html#tty_tty)를 참고해라.

## process.stderr

stderr에 대한 writable stream이다.

`process.stderr`와 `process.stdout`는 쓰기를 할 때 보통 블락킹된다는 점에서 Node의
다른 스크림과는 다르다. 이 둘은 보통의 파일이나 TTY 파일 디스크립터를 참조할 때
블락킹된다. 파이프를 참조하는 경우에는 다른 스크림처럼 넌블락킹이다.


## process.stdin

stdin에 대한 `Readable Stream`이다. stdin 스트림은 기본적으로 멈추기 때문에 stdin에서
읽으려면 `process.stdin.resume()`를 호출해야 한다.

표준 입력을 열고 두 이벤트를 리스닝하는 예제:

    process.stdin.resume();
    process.stdin.setEncoding('utf8');

    process.stdin.on('data', function(chunk) {
      process.stdout.write('data: ' + chunk);
    });

    process.stdin.on('end', function() {
      process.stdout.write('end');
    });


## process.argv

커맨드라인 아규먼트를 담고 있는 배열이다. 첫 엘리먼트는 'node'일 것이고 두 번째
엘리먼트는 자바스트립트 파일명이 될 것이다. 다음 엘리먼트들은 추가적인 커맨드라인
아규먼트일 것이다.

    // process.argv 출력
    process.argv.forEach(function(val, index, array) {
      console.log(index + ': ' + val);
    });

다음과 같이 출력된다:

    $ node process-2.js one two=three four
    0: node
    1: /Users/mjr/work/node/process-2.js
    2: one
    3: two=three
    4: four


## process.execPath

프로세스가 시작되는 실행가능한 절대경로명이다.

예제:

    /usr/local/bin/node


## process.abort()

이는 node가 abort를 발생시키게 한다. 즉 node가 종료되고 코어파일을 생성한다.

## process.chdir(directory)

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

프로세스의 현재 워킹디렉토리를 리턴한다.

    console.log('Current directory: ' + process.cwd());


## process.env

사용자의 환경변수를 담고 있는 객체다. environ(7)를 봐라.


## process.exit([code])

지정한 `code`로 프로세스를 종료한다. `code`를 생략하면 'success' 코드 `0`을
사용해서 종료한다.

'failure' 코드로 종료하려면:

    process.exit(1);

node를 실행한 쉘은 1을 종료코드로 간주할 것이다.


## process.getgid()

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(예를 들어 Windows에서는 안된다.)

프로세스의 그룹식별자를 얻는다.(getgid(2)를 봐라.)
이는 그룹 이름이 아니라 숫자로 된 그룹 id 이다.

    if (process.getgid) {
      console.log('Current gid: ' + process.getgid());
    }


## process.setgid(id)

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(예를 들어 Windows에서는 안된다.)

프로세스의 그룹 식별자를 설정한다.(setgid(2)를 봐라.) 이 함수는 숫자로 된 ID나 문자열로 된
그룹명을 모두 받아들인다. 그룹명을 지정하면 이 메서드가 그룹명을 숫자로된 ID로 처리할 때까지
블락킹한다.

    if (process.getgid && process.setgid) {
      console.log('Current gid: ' + process.getgid());
      try {
        process.setgid(501);
        console.log('New gid: ' + process.getgid());
      }
      catch (err) {
        console.log('Failed to set gid: ' + err);
      }
    }


## process.getuid()

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(예를 들어 Windows에서는 안된다.)

프로세스의 사용자 식별자를 얻는다.(getuid(2)를 봐라.)
이는 사용자명이 아니라 숫자로된 userid이다.

    if (process.getuid) {
      console.log('Current uid: ' + process.getuid());
    }


## process.setuid(id)

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(예를 들어 Windows에서는 안된다.)

프로세스의 사용자 식별자를 설정한다. (setuid(2)를 봐라.) 이는 숫자로된 ID와 문자열로 된
사용자명을 모두 받아들인다. 사용자명을 지정하면 이 메서드가 사용자명을 숫자로 된 ID로
처리할 때까지 블락킹한다.

    if (process.getuid && process.setuid) {
      console.log('Current uid: ' + process.getuid());
      try {
        process.setuid(501);
        console.log('New uid: ' + process.getuid());
      }
      catch (err) {
        console.log('Failed to set uid: ' + err);
      }
    }


## process.getgroups()

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(윈도우즈에선 사용할 수 없다.)

추가적인 그룹 ID를 가진 배열을 반환한다. POSIX는 유효한 그룹 ID를 포함하고 있는지
지정하지 않지만 node.js를 이를 항상 보장한다.


## process.setgroups(groups)

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(윈도우즈에선 사용할 수 없다.)

추가적인 그룹 ID를 설정한다. 이는 권한힌 필요한 작업이다. 즉, root이거나 CAP_SETGID 권한을
가져야 한다.

목록은 그룹 ID, 그룹명 또는 둘 다를 포함할 수 있다.


## process.initgroups(user, extra_group)

Note: 이 함수는 POSIX 플랫폼에서만 사용할 수 있다.(윈도우즈에선 사용할 수 없다.)

/etc/group를 읽고 사용자가 멤버로 있는 모든 그룹을 사용해서 그룹 접근 목록을 초기화한다.
이는 권한이 필요한 작업이므로 root이거나 CAP_SETGID 권한을 가져야 한다.

`user`는 사용자명이거나 사용자 ID이다. `extra_group`는 그룹명이나 그룹ID이다.

권한을 버릴때는 조심할 필요가 있다.
예제:

    console.log(process.getgroups());         // [ 0 ]
    process.initgroups('bnoordhuis', 1000);   // switch user
    console.log(process.getgroups());         // [ 27, 30, 46, 1000, 0 ]
    process.setgid(1000);                     // drop root gid
    console.log(process.getgroups());         // [ 27, 30, 46, 1000 ]


## process.version

`NODE_VERSION`으로 노출된 컴파일된 프로퍼티이다.

    console.log('Version: ' + process.version);

## process.versions

node와 의존성에 대한 버전 문자열을 노출하는 프로퍼티이다.

    console.log(process.versions);

다음과 같이 출력될 것이다:

    { http_parser: '1.0',
      node: '0.10.4',
      v8: '3.14.5.8',
      ares: '1.9.0-DEV',
      uv: '0.10.3',
      zlib: '1.2.3',
      modules: '11',
      openssl: '1.0.1e' }

## process.config

현재 실행되는 노드를 컴파일하는데 사용한 설정 옵션의 자바스크립트 표현을 담고 있는 객체다.
이는 `./configure` 스크립트를 실행했을 때 생성되는 "config.gypi" 파일과 같다.

출력은 다음 예제와 같다.

    { target_defaults:
       { cflags: [],
         default_configuration: 'Release',
         defines: [],
         include_dirs: [],
         libraries: [] },
      variables:
       { host_arch: 'x64',
         node_install_npm: 'true',
         node_prefix: '',
         node_shared_cares: 'false',
         node_shared_http_parser: 'false',
         node_shared_libuv: 'false',
         node_shared_v8: 'false',
         node_shared_zlib: 'false',
         node_use_dtrace: 'false',
         node_use_openssl: 'true',
         node_shared_openssl: 'false',
         strict_aliasing: 'true',
         target_arch: 'x64',
         v8_use_snapshot: 'true' } }

## process.kill(pid, [signal])

프로세스에 신호를 보낸다. `pid`는 프로세스 id이고 `signal`은 보내려는 신호를
설명하는 문자열이다. 신호이름은 'SIGINT'나 'SIGUSR1'같은 문자열이다. `signal`을
생략하면 'SIGTERM'가 될 것이다. 더 자세한 내용은 kill(2)를 봐라.

이 함수의 이름이 `process.kill`이므로 실제로 `kill` 시스템 호출처럼 단순히 신호
전송자이다. 보낸 신호는 타겟 신호를 죽이는 일이외에 다른 일을 할 것이다.

자신에게 신호를 보내는 예제:

    process.on('SIGHUP', function() {
      console.log('Got SIGHUP signal.');
    });

    setTimeout(function() {
      console.log('Exiting.');
      process.exit(0);
    }, 100);

    process.kill(process.pid, 'SIGHUP');


## process.pid

프로세스의 PID.

    console.log('This process is pid ' + process.pid);


## process.title

'ps'에서 표시될 어떻게 표시되는 지에 대한 Getter와 Setter

setter로 사용하는 경우 최대 길이는 플랫폼에 의존적이고 아마도 길지 않을 것이다.

리눅스나 OS X에서는 argv 메모리를 덮어쓰기 때문에 바이너리 명의 크기에 명령행 인자의 길이를
더한 크기로 제한될 것이다.

v0.8은 environ 메모리를 덮어써서 프로세스 타이틀 문자열을 더 길게 처리할 수 있지만
몇몇 (명확하지 않은)경우에는 잠재적으로 보안에 취약하거나 혼란스러울 수 있다.


## process.arch

어떤 프로세스 아키텍쳐에서 실행되고 있는지 보여준다.: `'arm'`, `'ia32'`, `'x64'`.

    console.log('This processor architecture is ' + process.arch);


## process.platform

어떤 플랫폼에서 실행되고 있는지 보여준다.
`'darwin'`, `'freebsd'`, `'linux'`, `'sunos'`, `'win32'`

    console.log('This platform is ' + process.platform);


## process.memoryUsage()

Node 프로세스의 메모리 사용량을 바이트로 나타내서 보여주는 객체를 리턴한다.

    var util = require('util');

    console.log(util.inspect(process.memoryUsage()));

다음과 같이 출력될 것이다:

    { rss: 4935680,
      heapTotal: 1826816,
      heapUsed: 650472 }

`heapTotal`와 `heapUsed`는 V8의 메모리 사용량을 참조한다.


## process.nextTick(callback)

이벤트 루프의 다음 번 루프에서 이 callback을 호출한다.
이는 단순히 `setTimeout(fn, 0)`에 대한 별칭이 *아니라* 훨씬 더 효율적이다.
이는 보통 어떤 I/O 이벤트도 발생하기 전에 실행되지만 몇가지 예외가 존재한다.
아래의 `process.maxTickDepth`를 참고해라.

    process.nextTick(function() {
      console.log('nextTick callback');
    });

이는 객체가 만들어 진 후이지만 어떤 I/O도 발생하기 전에 사용자가 이벤트 핸들러를 할당할 기회를
주고자 하는 API를 개발할 때 중요하다.

    function MyThing(options) {
      this.setupOptions(options);

      process.nextTick(function() {
        this.startDoingStuff();
      }.bind(this));
    }

    var thing = new MyThing();
    thing.getReadyForStuff();

    // thing.startDoingStuff()는 이제 호출된다.(그 이전이 아니라)

이는 100% 동기이거나 100% 비동기여야 하는 API에 아주 중요한다.
다음 예제를 봐라.

    // WARNING!  DO NOT USE!  BAD UNSAFE HAZARD!
    function maybeSync(arg, cb) {
      if (arg) {
        cb();
        return;
      }

      fs.stat('file', cb);
    }

이 API는 위험하다. 다음과 같이 사용한다면

    maybeSync(true, function() {
      foo();
    });
    bar();

`foo()`와 `bar()` 중 어느 쪽이 먼저 호출될 것인지 명확하지 않다.

다음 접근방법이 훨씬 좋다.

    function definitelyAsync(arg, cb) {
      if (arg) {
        process.nextTick(cb);
        return;
      }

      fs.stat('file', cb);
    }

## process.maxTickDepth

* {Number} Default = 1000

`process.nextTick`에 전달되는 콜백은 *보통* 현재 실행 흐름의 끝에 호출될 것이므로
대략 함수를 동기적으로 호출한 것만큼 빠르게 호출된다. 확인하지 않은채로 놔두면 이는
어떤 I/O의 발생도 발생하지 못하게 막으면서 이벤트 루프가 굶주리게(starve)할 것이다.

다음 코드를 보자.

    process.nextTick(function foo() {
      process.nextTick(foo);
    });

nextTick의 재귀 호출로 인한 무한루프가 Node를 블락하는 상황을 피하려면 매번
일부 I/O가 수행되도록 지연시켜야 한다.

`process.maxTickDepth` 값은 다른 I/O가 일어나도록 하기전에 계산할
nextTick이 호출하는 nextTick 콜백의 최대 깊이이다.

## process.umask([mask])

프로세스의 파일 모드 생성 마스크를 설정하거나 읽는다. 자식 프로세스는 부모 프로세스에서
이 마스크를 상속받는다. `mask`아규먼트를 전달하면 이전의 마스크를 리턴하고 `mask`아규먼트를
전달하지 않으면 현재 마스크를 리턴한다.

    var oldmask, newmask = 0644;

    oldmask = process.umask(newmask);
    console.log('Changed umask from: ' + oldmask.toString(8) +
                ' to ' + newmask.toString(8));


## process.uptime()

Node가 실행되고 있는 시간을 초단위로 나타낸다.


## process.hrtime()

`[seconds, nanoseconds]` 튜플 배열 형식으로 현재의 고해상도(high-resolution)
실제 시간을 반환한다. 이는 과거 임의의 시간과 관계가 있고 시각과는 관련이 없으므로
클럭 드리프트(clock drift)를 따르지 않는다. 어떤 구간사이의 성능을 측정하는 것이
주요 사용처이다.

어떤 구간을 벤치마킹을 위해 시간 간격을 얻기 위해 `process.hrtime()`에 이전 호출의 결과를
전달한다.

    var time = process.hrtime();
    // [ 1800216, 25 ]

    setTimeout(function() {
      var diff = process.hrtime(time);
      // [ 1, 552 ]

      console.log('benchmark took %d nanoseconds', diff[0] * 1e9 + diff[1]);
      // benchmark took 1000000527 nanoseconds
    }, 1000);

[EventEmitter]: events.html#events_class_events_eventemitter
