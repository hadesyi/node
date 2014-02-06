# Readline

    Stability: 2 - Unstable

이 모듈을 사용하려면 `require('readline')`를 실행해라. Readline은 줄마다
스트림(`process.stdin`같은)으로 읽을 수 있다.

이 모듈을 한번 호출하고 나면 인터페이스를 종료할 때까지 node 프로그램은 종료되지
않을 것이다. 어떻게 프로그램을 안전하게(gracefully) 종료할 수 있는지 보여주는
예제가 있다.

    var readline = require('readline');

    var rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

    rl.question("What do you think of node.js? ", function(answer) {
      // TODO: 데이터베이스에 답변을 로깅한다
      console.log("Thank you for your valuable feedback:", answer);

      rl.close();
    });

## readline.createInterface(options)

readline의 `Interface` 인스턴스를 생성한다. 다음 값의 "options" 객체를 받는다.

 - `input` - 리스닝할 읽을 수 있는 스트림 (필수).

 - `output` - readline 데이터를 작성할 쓰기가 가능한 스트림 (필수).

 - `completer` - 탭 자동완성에 사용한 선택적인 함수. 아래 이 값을 사용하는 예제를 봐라.

 - `terminal` - `input`스트림과 `output`스트림을 TTY처럼 다뤄야 하고 작성된 코드가
   ANSI/VT100로 이스케이프 되었다면 `true`를 전달한다.
   기본값은 인스턴스의 `output`스트림에서 `isTTY`를 확인하는 것이다.

`completer` 함수는 사용자가 입력하는 현재 라인을 받고 2가지 배열을 반환한다.

 1. 자동완성을 위해 일치하는 값들의 배열

 2. 매칭에 사용되는 부분문자열.

그래서 다음과 같은 결과가 나올 것이다.
`[[substr1, substr2, ...], originalsubstring]`

예제:

    function completer(line) {
      var completions = '.help .error .exit .quit .q'.split(' ')
      var hits = completions.filter(function(c) { return c.indexOf(line) == 0 })
      // 아무것도 찾지 못하면 모든 자동완성을 보여준다
      return [hits.length ? hits : completions, line]
    }

두 개의 아규먼트를 받으면 `completer`도 비동기로 실행할 수 있다.

    function completer(linePartial, callback) {
      callback(null, [['123'], linePartial]);
    }

`createInterface`는 보통 사용자 입력을 받기 위해
`process.stdin`와 `process.stdout`와 함께 사용한다.

    var readline = require('readline');
    var rl = readline.createInterface({
      input: process.stdin,
      output: process.stdout
    });

readline 인스턴스를 가지면 보통 `"line"` 이벤트를 리스닝한다.

해당 인스턴스의 `terminal`값이 `true`이면 `output` 스트림은 `output.columns`
프로퍼티를 정의하는 최적의 호환성을 가질 것이고 `output`은 칼럼이 변경될 때마다
`"resize"` 이벤트를 발생시킨다.
(`output`이 TTY이면 `process.stdout`는 자동으로 이렇게 한다.)

## Class: Interface

클래스는 입력스트림과 출력 스트림을 가진 readline 인터페이스를 나타낸다.

### rl.setPrompt(prompt, length)

프롬프트를 설정한다. 예를 들어 커맨드라인에서 `node`를 실행하면 node의 프롬프트인
`> `를 볼 수 있다.

### rl.prompt([preserveCursor])

사용자가 작성할 새로운 지점인 새로운 줄에서 현재의 `setPrompt` 옵션을 두어
사용자에게 입력을 받기 위해서 readline을 준비한다. 커서 위치를 `0`으로 초기화하는
것을 막으려면 `preserveCursor`을 `true`으로 설정해라.

`createInterface`과 함께 사용한 `input` 스트림이 멈춰있다면 이 함수는 다시
시작할 것이다.

### rl.question(query, callback)

`query`앞에 프로프트를 붙이고 사용자의 응답으로 `callback`을 호출한다.
사용자에게 쿼리를 보여주고 사용자가 입력한 응답으로 `callback`을 호출한다.

`createInterface`과 함께 사용한 `input` 스트림이 멈춰있다면 이 함수는 다시
시작할 것이다.

예제:

    interface.question('What is your favorite food?', function(answer) {
      console.log('Oh, so your favorite food is ' + answer);
    });

### rl.pause()

나중에 필요할 때 다시 시작할 수 있도록 readline `input` 스트림을 멈춘다.

### rl.resume()

readline `input` 스트림을 다시 시작한다.

### rl.close()

`input`과 `output` 스트림의 제어권을 포기하고 `Interface` 인스턴스를 닫는다.
"close" 이벤트도 발생할 것이다.

### rl.write(data, [key])

`output` 스트림에 `data`를 작성한다. `key`는 키의 순서를 나타내는 객체 리터럴로 터미널이
TTY일 때 사용할 수 있다.

`input` 스트림이 멈춰있다면 이 함수는 다시 시작할 것이다.

예제:

    rl.write('Delete me!');
    // 이전에 작성한 라인을 삭제하는 ctrl+u를 시뮬레이트한다
    rl.write(null, {ctrl: true, name: 'u'});

## Events

### Event: 'line'

`function (line) {}`

`in` 스트림이 `\n`를 받을 때마다 발생한다. 보통 사용자가 엔터를 쳤을 때 받게 된다.
이 이벤트는 사용자 입력을 받기 위한 좋은 훅(hook)이다.

`line` 이벤트를 받는 예제다.

    rl.on('line', function (cmd) {
      console.log('You just typed: '+cmd);
    });

### Event: 'pause'

`function () {}`

`input` 스트림이 멈출 때마다 발생한다.

`input` 스트림이 멈춰있지 않고 `SIGCONT` 이벤트를 받을 때도 발생한다.
(`SIGTSTP`이벤트와 `SIGCONT`이벤트를 참고해라.)

`pause` 이벤트를 리스닝하는 예제:

    rl.on('pause', function() {
      console.log('Readline paused.');
    });

### Event: 'resume'

`function () {}`

`input`스트림이 재시작 할 때마다 발생한다.

`resume` 이벤트를 리스닝하는 예제:

    rl.on('resume', function() {
      console.log('Readline resumed.');
    });

### Event: 'close'

`function () {}`

`close()`를 호출할 때 발생한다.

`input` 스트림이 "end" 이벤트를 받을 때도 발생한다. `Interface` 인터페이스는 "finished"
이벤트가 발생했다고 간주한다. 예를 들어 `input` 스트림이 `EOT`인 `^D`를 받은 경우이다.

`input` 스트림이 `SIGINT`의 의미인 `^C`를 받았을 때 `SIGINT` 이벤트에 등록된 리스너가
없으면 이 이벤트가 호출된다.

### Event: 'SIGINT'

`function () {}`

`input` 스트림이 `SIGINT`의 의미인 `^C`를 받을 때마다 발생한다. `input` 스트림이
`SIGINT`를 받을 때 `SIGINT` 이벤트에 등록된 리스너가 없으면 `pause` 이벤트가 발생한다.

`SIGINT`를 리스닝하는 예제:

    rl.on('SIGINT', function() {
      rl.question('Are you sure you want to exit?', function(answer) {
        if (answer.match(/^y(es)?$/i)) rl.pause();
      });
    });

### Event: 'SIGTSTP'

`function () {}`

**이 이벤트는 Windows에서는 동작하지 않는다.**

`input` 스트림이 `SIGTSTP`로 알려진 `^Z`를 받을 때마다 발생한다. `input` 스트림이
`SIGTSTP`을 받았을 때 `SIGTSTP`이벤트에 등록된 리스너가 없으면 프로그램은 백그라운드로
보내진다.

프로그램이 `fg`로 복귀했을 때 `pause`와 `SIGCONT` 이벤트가 실행될 것이다. 스트림으로
복귀하는 데 사용할 수도 있다.

프로그램이 백그라운드로 보내지기 전에 스트림이 멈춰있다면 `pause`와 `SIGCONT` 이벤트는
발생하지 않을 것이다.

`SIGTSTP` 이벤트를 리스닝하는 예제:

    rl.on('SIGTSTP', function() {
      // 이는 SIGTSTP를 오버라이드 하고 프로그램이 백그라운드로 가는 것을 막는다.
      console.log('Caught SIGTSTP.');
    });

### Event: 'SIGCONT'

`function () {}`

**이 이벤트는 Windows에서는 동작하지 않는다.**

`input` 스트림이 `SIGTSTP`로 알려진 `^Z`를 통해 백그라운드로 보내질 때마다 발생하고
이는 `fg(1)`로 다시 복귀할 수 있다. 이 이벤트는 프로그램을 백그라운드로 보내기 전에
스트림이 멈춰있지 않은 경우에만 발생한다.

`SIGCONT`이벤트를 리스닝하는 예제:

    rl.on('SIGCONT', function() {
      // `prompt`는 자동으로 스트림을 복구시킬 것이다
      rl.prompt();
    });


## Example: Tiny CLI

이 모두를 사용해서 어떻게 작은 명령행 인터페이스를 만드는지 보여주는 예제가 있다.

    var readline = require('readline'),
        rl = readline.createInterface(process.stdin, process.stdout);

    rl.setPrompt('OHAI> ');
    rl.prompt();

    rl.on('line', function(line) {
      switch(line.trim()) {
        case 'hello':
          console.log('world!');
          break;
        default:
          console.log('Say what? I might have heard `' + line.trim() + '`');
          break;
      }
      rl.prompt();
    }).on('close', function() {
      console.log('Have a great day!');
      process.exit(0);
    });

