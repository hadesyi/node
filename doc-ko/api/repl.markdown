# REPL

Read-Eval-Print-Loop (REPL)는 단독 프로그램과 다른 프로그램에 쉽게 포함해서 사용할 수 있다.
REPL은 자바스크립트를 실행하고 결과를 보는 대화식 방법을 제공한다. 이는 디버깅, 테스팅이나
그냥 간단한 것을 시도해 볼 때 사용할 수 있다.

명령행에서 아규먼트없이 `node`를 실행하면 REPL에 들어간다.
REPL은 극도로 단순한 emacs 라인수정을 가진다.

    mjr:~$ node
    Type '.help' for options.
    > a = [ 1, 2, 3];
    [ 1, 2, 3 ]
    > a.forEach(function (v) {
    ...   console.log(v);
    ...   });
    1
    2
    3

향상된 라인에디터를 위해서는 환경변수 `NODE_NO_READLINE=1`로 node를 시작해라.
이는 `rlwrap`를 사용할 수 있도록 인정된 터미널 설정에서 메인 REPL과
디버거 REPL을 시작한다.

예를 들어 bashrc 파일에 다음을 추가할 수 있다.

    alias node="env NODE_NO_READLINE=1 rlwrap node"


## repl.start(options)

`REPLServer` 인스턴스를 시작하고 반환한다. 다음 값의 "options" 객체를 받는다.

 - `prompt` - 모든 I/O에 대한 프롬프트와 `stream`. 기본값은 `> `이다.

 - `input` - 리스닝할 읽을 수 있는 스트림. 기본값은 `process.stdin`이다.

 - `output` - readline 데이터를 작성할 쓰기가능한 스트림. 기본값은
   `process.stdout`이다.

 - `terminal` - `stream`을 TTY처럼 다뤄야 하고 작성된 데이터가 ANSI/VT100로
   이스케이프되어 있다면 `true`를 전달해라. 기본값은 인스턴스의 `output` 스트림에서
   `isTTY`를 확인하는 것이다.

 - `eval` - 각 주어진 라인을 평가(eval)하는데 사용할 함수. 기본값은 `eval()`에 대한
   비동기 래퍼이다. 아래의 커스텀 `eval`의 예제를 참조해라.

 - `useColors` - `writer` 함수가 색상을 출력해야 하는지 아닌지를 지정하는 불리언값.
   다른 `writer` 함수를 설정하면 이 값은 아무것도 하지 않는다. 기본값은 repl의
   `terminal`값이다.

 - `useGlobal` - `true`로 설정하면 repl이 분리된 컨텍스트에서 스크립트를 실행하는
   대신에 `global` 객체를 사용한다. 기본값은 `false`이다.

 - `ignoreUndefined` - `true`로 설정하면 repl은 명령의 결과값이 `undefined`인
   경우 출력하지 않는다. 기본값은 `false`이다.

 - `writer` - 화면에 표시하기 위해서 포매팅(컬러링 포함)된 값을 평가하려고
   명령마다 호출할 함수이다. 기본값은 `util.inspect`이다.

다음과 같은 시그니처를 가진 자신만의 `eval`함수를 사용할 수 있다.

    function eval(cmd, context, filename, callback) {
      callback(null, result);
    }

다중 REPL은 node에서 실행되는 같은 인스턴스에서 시작될 것이다. 각 REPL은 같은 전역객체를
공유하지만 각각 유일한 I/O를 가질 것인다.

stdin, Unix 소켓, TCP 소켓에서 REPL을 시작하는 예제는 다음과 같다.

    var net = require("net"),
        repl = require("repl");

    connections = 0;

    repl.start({
      prompt: "node via stdin> ",
      input: process.stdin,
      output: process.stdout
    });

    net.createServer(function (socket) {
      connections += 1;
      repl.start({
        prompt: "node via Unix socket> ",
        input: socket,
        output: socket
      }).on('exit', function() {
        socket.end();
      })
    }).listen("/tmp/node-repl-sock");

    net.createServer(function (socket) {
      connections += 1;
      repl.start({
        prompt: "node via TCP socket> ",
        input: socket,
        output: socket
      }).on('exit', function() {
        socket.end();
      });
    }).listen(5001);

명령행에서 이 프로그램을 실행하면 stdin에서 REPL을 시작할 것이다. 다른 REPL 클라이언트는
Unix 소켓이나 TCP 소켓을 통해서 연결할 것이다. `telnet`은 TCP 소켓에 연결하는 데 유용하고
`socat`은 Unix와 TCP 소켓에 연결하는 데 사용할 수 있다.

stdin 대신 Unix 소켓에 기반한 서버에서 REPL을 시작하면 재시작 없이 오랫동안
실행되는 node 프로세스에 연결할 수 있다.

`net.Server`와 `net.Socket` 인스턴스에서 실행되는 "완전한 기능의" (`terminal`) REPL의
예제는 https://gist.github.com/2209310 를 참고해라.

`curl(1)`을 실행하는 REPL 인스턴스의 예제는
https://gist.github.com/2053342 를 참고해라.

### Event: 'exit'

`function () {}`

사용자가 어떤 방법으로든 REPL을 종료했을 때 발생한다. 즉 repl에서 `.exit`를 입력하거나
SIGINT 신호를 위해 Ctrl+C를 두번 입력하거나 `input` 스트림에서 "end" 신호를 위해
Ctrl+D를 입력하는 등이다.

`exit` 이벤트를 리스닝하는 예제:

    r.on('exit', function () {
      console.log('Got "exit" event from repl!');
      process.exit();
    });


## REPL Features

<!-- type=misc -->

REPL내에서 Control+D를 누르면 종료될 것이다. 다중라인 포현식은 입력이 될 수 있다.
전역 변수와 지역 변수에 모두 탭 자동완성을 지원한다.

특수한 변수 `_` (언더스코어)는 마지막 표현식의 결과를 담고 있다.

    > [ "a", "b", "c" ]
    [ 'a', 'b', 'c' ]
    > _.length
    3
    > _ += 1
    4

REPL은 전역범위의 어떤 변수라도 접근할 수 있다.
각 `REPLServer`과 연결된 `context` 객체에 할당해서 명시적으로 REPL에 변수를
노출할 수 있다. 예를 들어

    // repl_test.js
    var repl = require("repl"),
        msg = "message";

    repl.start("> ").context.m = msg;

REPL내 `context` 객체에서 지역변수로 나타나는 변수들이 있다.

    mjr:~$ node repl_test.js
    > m
    'message'

몇몇 REPL 명령어가 있다.

  - `.break` - 다중 라인 표현식을 입력하는 동안 종종 멈추거나 표현식을 완성하기를 신경쓰지
    않을 때 사용한다. `.break`은 다시 시작할 것이다.
  - `.clear` - `context` 객체를 비어있는 객체로 리셋하고 모든 다중라인 표현식을
    정리한다.
  - `.exit` - REPL이 종료되도록 I/O 스트림을 닫는다.
  - `.help` - 이 특수한 명령어 리스트를 보여준다.
  - `.save` - 현재 REPL 세션을 파일로 저장한다.
    >.save ./file/to/save.js
  - `.load` - 파일에서 현재 REPL 세션으로 로드한다.
    >.load ./file/to/load.js

REPL에서 다음의 키 조합은 다음과 같은 특수한 효가가 있다.

  - `<ctrl>C` - `.break` 키워드와 유사하다. 현재 명령어를 종료한다.
    비어있는 라인에서 두 번 입력하면 강제적으로 종료한다.
  - `<ctrl>D` - `.exit` 키워드와 유사하다.

