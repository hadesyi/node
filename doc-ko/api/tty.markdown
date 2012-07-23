# TTY

    Stability: 2 - Unstable

`tty` 모듈에는 `tty.ReadStream`과 `tty.WriteStream` 클래스가 들어 있다. 보통 이 모듈을 직접 사용할 일은 별로 없다.

TTY 컨텍스트에서 실행되면 자동으로 `process.stdin`을 `tty.ReadStream` 인스턴스로 만들고 `process.stdout`은 `tty.WriteStream` 인스턴스로 만든다. `process.stdout.isTTY`를 검사하면 node가 TTY 컨텍스트에서 실행되는지 아닌지 알 수 있다.
 
    $ node -p -e "Boolean(process.stdout.isTTY)"
    true
    $ node -p -e "Boolean(process.stdout.isTTY)" | cat
    false
 
## tty.isatty(fd)

fd가 터미널 파일 디스크립터인지에 따라 `true`나 `false`를 리턴한다.

## tty.setRawMode(mode)

Deprecated 됐다. `tty.ReadStream#setRawMode()`를 사용하라(i.e. `process.stdin.setRawMode()`).

## Class: ReadStream

TTY 읽기를 담당하는 `net.Socket` 서브클래스이다. 보통 node 프로그램의 `process.stdin`은 `tty.ReadStream` 인스턴스이다(`isatty(0)`가 true일 때만). 

### rs.isRaw

이 프로퍼티는 `false`로 초기화된다. `tty.ReadStream` 인스턴스가 현재 "raw" 상태임을 나타낸다.

### rs.setRawMode(mode)

`mode`에는 `true`나 `false`를 넘긴다. mode가 true이면 `tty.ReadStream` 프로퍼티가 raw 디바이스 처럼 동작하도록 설정하고 아니면 기본 값이 설정된다. `isRaw` 프로퍼티로 어떻게 설정했는지 알 수 있다.

## Class WriteStream

TTY 쓰기를 담당하는 `net.Socket` 서브클래스이다. 보통 node 프로그램의 `process.stdout`은 `tty.ReadStream` 인스턴스이다(`isatty(1)`가 true일 때만). 


### ws.columns

이 프로퍼티는 현 TTY의 컬럼 수이다. "resize" 이벤트 시 업데이트된다.

### ws.rows

이 프로퍼티는 현 TTY의 로우 수이다. "resize" 이벤트 시 업데이트된다.

### Event: 'resize'

`function () {}`

`columns`나 `rows` 프로퍼티가 변경될 때 발생한다.

    process.stdout.on('resize', function() {
      console.log('screen size has changed!');
      console.log(process.stdout.columns + 'x' + process.stdout.rows);
    });
