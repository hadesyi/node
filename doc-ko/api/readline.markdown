# Readline

    Stability: 3 - Stable

이 모듈을 사용하려면 `require('readline')`를 실행해라. Readline은 줄마다 
스트림(STDIN같은)으로 읽을 수 있다.

이 모듈을 한번 호출하고 나면 인터페이스를 종료할 때까지 node 프로그램은 종료되지 
않을 것이다. 어떻게 프로그램을 안전하게(gracefully) 종료할 수 있는 지 보여주는 
예제가 있다.

    var rl = require('readline');

    var i = rl.createInterface(process.stdin, process.stdout, null);
    i.question("What do you think of node.js?", function(answer) {
      // TODO: 데이터베이스에 답변을 로깅한다.
      console.log("Thank you for your valuable feedback.");

      // 이 두 줄은 프로그램을 종료하게 한다. 이 두 라인이 없으면 
      // 영원히 종료되지 않을 것이다.
      i.close();
      process.stdin.destroy();
    });

## rl.createInterface(input, output, completer)

두 스트림을 받아서 readline 인터펭스를 생성한다. `completer` 함수는 자동완성에 
사용한다. 부분문자열을 전달했을 때 
`[[substr1, substr2, ...], originalsubstring]`를 반환한다.

`completer`가 두 개의 아규먼트를 받아들이면 `completer`는 비동기 모드로 
실행할 수도 있다.

  function completer(linePartial, callback) {
    callback(null, [['123'], linePartial]);
  }

`createInterface`는 사용자 입력을 받기 위해 `process.stdin`와 
`process.stdout`를 보통 사용한다.

    var readline = require('readline'),
      rl = readline.createInterface(process.stdin, process.stdout);

## Class: Interface

클래스는 stdin와 stdout 스트림을 가진 readline 인터페이스를 나타낸다.

### rl.setPrompt(prompt, length)

프롬프트를 설정한다. 예를 들어 커맨드라인에서 `node`를 실행하면 node의 프롬프트인 
`> `를 볼 수 있다.

### rl.prompt()

사용자가 작성할 새로운 지점인 새로운 줄에서 현재의 `setPrompt` 옵션을 두어 
사용자에게 입력을 받기 위해서 readline을 준비한다.

### rl.question(query, callback)

`query`앞에 프로프트를 붙히고 사용자의 응답으로 `callback`을 호출한다. 
사용자에게 쿼리를 보여주고 사용자가 입력한 응답으로 `callback`을 호출한다.

### rl.close()

  tty를 닫는다.

### rl.pause()

  tty를 멈춘다.

### rl.resume()

  tty를 복구한다.

### rl.write()

  tty에 작성한다.

### Event: 'line'

`function (line) {}`

`in` 스트림이 `\n`를 받을 때마다 발생한다. 보통 사용자가 엔터를 쳤을 때 받게 된다.
이 이벤트는 사용자 입력을 받기 위한 좋은 후크(hook)이다.

`line` 이벤트를 받는 예제다.

    rl.on('line', function (cmd) {
      console.log('You just typed: '+cmd);
    });

### Event: 'close'

`function () {}`

`in` 스트림이 각각 `SIGINT`와 `EOT`로 알려진 `^C`나 `^D`를 받을 때마다 발생한다. 
이 이벤트는 사용자가 사용하는 프로그램을 종료했다는 것을 아는 좋은 방법이다.

`close` 이벤트를 받아서 이어서 프로그램을 종료하는 예제다.

    rl.on('close', function() {
      console.log('goodbye!');
      process.exit(0);
    });

이 모두를 사용해서 어떻게 작은 명령행 인터페이스를 만드는지 보여주는 예제가 있다.

    var readline = require('readline'),
      rl = readline.createInterface(process.stdin, process.stdout),
      prefix = 'OHAI> ';

    rl.on('line', function(line) {
      switch(line.trim()) {
        case 'hello':
          console.log('world!');
          break;
        default:
          console.log('Say what? I might have heard `' + line.trim() + '`');
          break;
      }
      rl.setPrompt(prefix, prefix.length);
      rl.prompt();
    }).on('close', function() {
      console.log('Have a great day!');
      process.exit(0);
    });
    console.log(prefix + 'Good to see you. Try typing stuff.');
    rl.setPrompt(prefix, prefix.length);
    rl.prompt();


약간 더 복잡한 [예제](https://gist.github.com/901104)에서 볼 수 있고
실사용 예제는 [http-console](https://github.com/cloudhead/http-console)에서 
볼 수 있다.
