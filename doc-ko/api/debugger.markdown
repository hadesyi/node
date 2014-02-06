# Debugger

    Stability: 3 - Stable

<!-- type=misc -->

V8에는 확장 가능한 디버거가 들어 있다. 이 디버거는 간단한
[TCP 프로토콜](http://code.google.com/p/v8/wiki/DebuggerProtocol)을 통해서
프로세스 외부에서 접근할 수 있다. 그리고 Node에는 이 디버거를 사용하는 빌트인 디버거
클라이언트가 들어 있다. Node를 실행할 때 `debug` 인자를 주면 디버거를 사용할 수 있다:

    % node debug myscript.js
    < debugger listening on port 5858
    connecting... ok
    break in /home/indutny/Code/git/indutny/myscript.js:1
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
    debug>

이 내장 클라이언트는 명령어를 다 지원하지 않지만, step과 inspection을 간단히
해볼 수 있다. 소스코드에 `debugger;`를 집어넣으면 그 부분이 breakpoint가 된다.

예를 들어, 다음과 같은 `myscript.js`를 보자.

    // myscript.js
    x = 5;
    setTimeout(function () {
      debugger;
      console.log("world");
    }, 1000);
    console.log("hello");

그러고 나서 디버거를 실행하면 4번째 줄에서 멈춘다.

    % node debug myscript.js
    < debugger listening on port 5858
    connecting... ok
    break in /home/indutny/Code/git/indutny/myscript.js:1
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
    debug> cont
    < hello
    break in /home/indutny/Code/git/indutny/myscript.js:3
      1 x = 5;
      2 setTimeout(function () {
      3   debugger;
      4   console.log("world");
      5 }, 1000);
    debug> next
    break in /home/indutny/Code/git/indutny/myscript.js:4
      2 setTimeout(function () {
      3   debugger;
      4   console.log("world");
      5 }, 1000);
      6 console.log("hello");
    debug> repl
    Press Ctrl + C to leave debug repl
    > x
    5
    > 2+2
    4
    debug> next
    < world
    break in /home/indutny/Code/git/indutny/myscript.js:5
      3   debugger;
      4   console.log("world");
      5 }, 1000);
      6 console.log("hello");
      7
    debug> quit
    %

`repl` 명령어는 원격에서 코드를 실행해볼 수 있도록 해준다. `next` 명령어를 실행하면 다음 줄로 넘어간다(step over). 그 외에도 사용할 수 있는 명령어가 꽤 있다. `help`를 입력하면 뭐가 있는지 보여준다.

## Watchers

디버깅하는 동안 표현식(expression)과 변수의 값을 watch할 수 있다. breakpoint마다 watcher에 등록된 표현을 현 컨텍스트에서 실행해서 보여준다. 그다음에 breakpoint가 있는 소스코드가 출력된다.

`watch("my_expression")`으로 watcher를 등록하고 `watchers` 명령으로 등록된 watcher들을 확인할 수 있다. 그리고 `unwatch("my_expression")`으로 등록된 watcher를 제거할 수 있다.

## Commands reference

### Stepping

* `cont`, `c` - 계속 실행
* `next`, `n` - Step next
* `step`, `s` - Step in
* `out`, `o` - Step out
* `pause` - 실행을 멈춘다(크롬 개발자 도구에서의 pause 버튼처럼)

### Breakpoints

* `setBreakpoint()`, `sb()` - 현 라인에 breakpoint를 설정한다.
* `setBreakpoint(line)`, `sb(line)` - 특정 라인에 breakpoint를 설정한다.
* `setBreakpoint('fn()')`, `sb(...)` - 함수 바디의 첫 라인에 breakpoint를 설정한다.
* `setBreakpoint('script.js', 1)`, `sb(...)` - script.js의 첫 라인에 breakpoint를 설정한다.
* `clearBreakpoint`, `cb(...)` - breakpoint를 제거

아직 로딩되지 않은 파일(모듈)에 breakpoint를 설정할 수도 있다.

    % ./node debug test/fixtures/break-in-module/main.js
    < debugger listening on port 5858
    connecting to port 5858... ok
    break in test/fixtures/break-in-module/main.js:1
      1 var mod = require('./mod.js');
      2 mod.hello();
      3 mod.hello();
    debug> setBreakpoint('mod.js', 23)
    Warning: script 'mod.js' was not loaded yet.
      1 var mod = require('./mod.js');
      2 mod.hello();
      3 mod.hello();
    debug> c
    break in test/fixtures/break-in-module/mod.js:23
     21
     22 exports.hello = function() {
     23   return 'hello from module';
     24 };
     25
    debug>

### Info

* `backtrace`, `bt` - 현 execution의 backtrace를 출력한다.
* `list(5)` - 스크립트 소스코드를 다섯 라인 나열한다. 다섯 라인은 현재 멈춘 컨텍스트 전후에 있는 라인을 의미한다.
* `watch(expr)` - watch 목록에 표현식을 넣는다.
* `unwatch(expr)` - watch 목록에서 표현식을 제거한다.
* `watchers` - 등록된 watcher와 그 값을 함께 보여준다(breakpoint에 멈출 때에도 자동으로 보여준다.).
* `repl` - 디버그하는 스크립트의 컨텍스트에서 실행되는 repl을 연다.

### Execution control

* `run` - 스크립트 실행 (디버거를 실행하면 자동으로 이 명령이 수행된다.)
* `restart` - 스크립트 재시작
* `kill` - 스크립트 끝내기

### Various

* `scripts` - 열린 스크립트를 모두 보여준다.
* `version` - v8의 버전을 보여준다.

## Advanced Usage

커맨드 라인에서 `--debug` 플래그 주고 Node를 실행하면 V8 디버거가 켜진다. 또 이미
실행 중인 Node 프로세스에 `SIGUSR1` 신호를 보내면 V8 디버거가 켜지고 접근해서 사용할 수 있다.

프로세스가 디버그 모드로 설정되면 노드 디버거로 접속할 수 있다. 디버거에 `pid`나 URI로 접속할 수 있다.
문법은 다음과 같다.

* `node debug -p <pid>` - `pid`로 프로세스에 연결한다
* `node debug <URI>` - localhost:5858 같은 URI로 프로세스에 연결한다.
