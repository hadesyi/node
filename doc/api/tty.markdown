# TTY

<!--english start-->

    Stability: 3 - Stable

Use `require('tty')` to access this module.

Example:

    var tty = require('tty');
    process.stdin.resume();
    tty.setRawMode(true);
    process.stdin.on('keypress', function(char, key) {
      if (key && key.ctrl && key.name == 'c') {
        console.log('graceful exit');
        process.exit()
      }
    });

<!--english end-->

    Stability: 3 - Stable

이 모듈에 접근하려면 `require('tty')`를 사용해라.

예제:

    var tty = require('tty');
    process.stdin.resume();
    tty.setRawMode(true);
    process.stdin.on('keypress', function(char, key) {
      if (key && key.ctrl && key.name == 'c') {
        console.log('graceful exit');
        process.exit()
      }
    });



## tty.isatty(fd)

<!--english start-->

Returns `true` or `false` depending on if the `fd` is associated with a
terminal.

<!--english end-->

`fd`가 터미널과 연결되었는 지에 따라 `true`나 `false`를 반환한다.


## tty.setRawMode(mode)

<!--english start-->

`mode` should be `true` or `false`. This sets the properties of the current
process's stdin fd to act either as a raw device or default.

<!--english end-->

`mode`는 `true`나 `false`가 되어야 한다. 이는 현재 프로세스의 stdin fd가 
로우(raw) 디바이스으로써나 기본값으로 동작하도록 프로퍼티를 설정한다. 
