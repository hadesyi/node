# Stream

    Stability: 2 - Unstable

스프림은 Node에서 여러 가지 객체로 구현되는 추상 인터페이스다. 예를 들어 HTTP 서버에 대한
요청은 stout과 같은 스트림이다. 스트림은 읽을수 있거나 쓸 수 있고 때로는 둘 다 가능하다.
모든 스트림은 [EventEmitter][]의 인스턴스다.

`require('stream')`을 사용해서 기반 Stream 클래스를 로드할 수 있다.

## Readable Stream

<!--type=class-->

`Readable Stream`에는 다음과 같은 메서드, 멤버, 이벤트가 있다.

### Event: 'data'

`function (data) { }`

`'data'` 이벤트는 `Buffer`(기본값)를 발생시키거나 `setEncoding()`가 사용된
경우 문자열을 발생시킨다.

`Readable Stream`가 `'data'` 이벤트를 발생시켰을 때 리스너가 없다면 
__데이터를 잃어버릴 것이다.__

### Event: 'end'

`function () { }`

스트림이 EOF(TCP 용어로 FIN)를 받았을 때 발생한다.
더이상 `'data'` 이벤트가 발생하지 않는다는 것을 나타낸다. 스트림이 쓰기도 가능하다면
쓰기는 계속해서 가능할 것이다.

### Event: 'error'

`function (exception) { }`

데이터를 받는데 오류가 있으면 발생한다.

### Event: 'close'

`function () { }`

의존하는 리소스(예를 들면 파일 디스크립터)가 닫혔을 때 발생한다. 모든 스트림이 
이 이벤트를 발생키시는 것은 아니다.

### stream.readable

`true`가 기본값인 불리언이지만 `'error'`가 발생하거나 스트림이 `'end'`가 되거나 
`destroy()`이 호출된 뒤에 `false`로 바뀐다.

### stream.setEncoding([encoding])

`'data'` 이벤트가 `Buffer` 대신 문자열을 발생시키도록 한다. `encoding`은 `'utf8'`, 
`'utf16le'` (`'ucs2'`), `'ascii'`, `'hex'`가 될 수 있다. 기본값은 `'utf8'`이다.

### stream.pause()

의존하는 통신계층에 `resume()`을 호출할 때까지 더이상 데이터를 보내지 않도록 요청하는 
신호를 발생시킨다. 

이런한 특성 때문에 특정 스크림은 즉시 멈추지 않고 `pause()`를 호출한 후에도 
불확실한 기간동안 `'data'` 이벤트는 발생할 것이다. 

### stream.resume()

`pause()`후에 들어오는 `'data'` 이벤트를 다시 시작한다.

### stream.destroy()

기반이 되는 파일 디스크립터를 닫는다. 스트림은 더 이상 `writable`도 아니고 
`readable`도 아니다. 스트림은 더는 'data'나 'end' 이벤트를 발생시키지 않는다. 
큐에 있는 어떤 작성데이터도 보내지 않을 것이다. 스트림은 관련된 리소스를 처리하는 
'close' 이벤트를 실행해야 한다.


### stream.pipe(destination, [options])

이 메서드는 모든 `Stream`에서 사용할 수 있는 `Stream.prototype` 메서드이다.

이 읽는 스트림을 `destination` 작성할 스트림에 연결한다. 이 스트림에 들어오는 데이터는
`destination`에 쓰여진다. 목적지 스트림과 출처 스트림은 필요에 따라 멈추거나 다시 
시작함으로써 동기화가 유지된다.

이 함수는 `destination` 스트림을 반환한다.

Unix의 `cat` 명령어를 에뮬레이팅한다.:

    process.stdin.resume(); process.stdin.pipe(process.stdout);


출처 스트림이 `end`를 발생하면 기본적으로 목적지 스트림도 `end()`를 호출하므로 
`destination`는 더이상 쓸 수 없다. 목적지 스트림을 열려진 상태로 놔두려면
`options`으로 `{ end: false }`를 전달한다.

이는 마지막에 "Goodbye"를 작성할 수 있도록 `process.stdout`를 열어둔 채로 놔둔다.

    process.stdin.resume();

    process.stdin.pipe(process.stdout, { end: false });

    process.stdin.on("end", function() {
    process.stdout.write("Goodbye\n"); });


## Writable Stream

<!--type=class-->

`Writable Stream`에는 다음의 메서드, 멤버, 이벤트가 있다.

### Event: 'drain'

`function () { }`

`write()` 메서드가 `false`를 반환한 뒤에 다시 작성할 수 있는 상태를
알리기 위해 이 이벤트를 발생시킨다.

### Event: 'error'

`function (exception) { }`

오류가 있을 때 예외 `exception`과 함께 발생시킨다.

### Event: 'close'

`function () { }`

기반이 되는 파일 디스크립터가 닫혔을 때 발생한다.

### Event: 'pipe'

`function (src) { }`

스트림이 읽을 수 있는 스트림의 pipe 메서드에 전달되었을 때 발생한다.

### stream.writable

기본값이 `true`인 불리언이지만 `'error'`가 발생하거나 `end()` / `destroy()`가 
호출된 후에 `false`로 바뀐다.

### stream.write(string, [encoding], [fd])

주어진 `encoding`으로 스트림에 `string`를 작성한다. 문자열이 커널 버퍼에 플러시되면
`true`를 반환한다. 커널 버퍼가 꽉차면 데이터가 차후에 보내진다는 것을 알리기 위해
`false`를 반환한다. 커널버퍼가 다시 비워지면 `'drain'` 이벤트로 알려줄 것이다.
`encoding`의 기본값은 `'utf8'`이다.

선택사항인 `fd` 파라미터를 지정하면 스트림에 보낸 필수 파일디스크립터로 해석된다.
이는 UNIX 스트림에서만 지원하고 다른 스트림에서는 경고없이 무시될 것이다. 이 방법으로
파일 디스크립터를 작성할 때 스트림의 drain 이전에 해당 디스크립터를 닫으면 유효하지
않은(닫힌) FD에 데이터를 보낼 위험이 있다.

### stream.write(buffer)

raw buffer를 사용한다는 점만 빼고는 위와 동일하다.

### stream.end()

EOF나 FIN로 스트림을 종료시킨다.
이 함수를 호출하면 스트림을 닫기 전에 큐에 있는 작성 데이터를 보낼 것이다.

### stream.end(string, encoding)

주어진 `encoding`으로 `string`을 보내고 EOF나 FIN로 스트림을 종료시킨다.
이는 보내는 패킷의 수를 줄이는 데 유용하다.

### stream.end(buffer)

위와 동일하지만 `buffer`를 사용한다.

### stream.destroy()

기반이 되는 파일 디스크립터를 닫는다. 스트림은 더이상 `writable`도 아니고 
`readable`도 아니다. 스트림은 더는 'data'나 'end' 이벤트를 발생시키지 않는다. 
큐에 있는 어떤 작성데이터도 보내지 않을 것이다. 스트림은 관련된 리소스를 처리하는 
'close' 이벤트를 실행해야 한다.

### stream.destroySoon()

작성 큐를 소모한 후에 파일 디스크립터를 닫는다. 작성 큐에 남아있는 데이터가 없다면
`destroySoon()`는 즉시 파일 디스크립터를 닫을 수 있다.

[EventEmitter]: events.html#events_class_events_eventemitter
