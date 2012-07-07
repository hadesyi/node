# Stream

<!--english start-->

    Stability: 2 - Unstable

A stream is an abstract interface implemented by various objects in Node.
For example a request to an HTTP server is a stream, as is stdout. Streams
are readable, writable, or both. All streams are instances of `EventEmitter`.

You can load up the Stream base class by doing `require('stream')`.

<!--english end-->

    Stability: 2 - Unstable

스프림은 Node에서 여러 가지 객체로 구현되는 추상 인터페이스다. 예를 들어 HTTP 서버에 대한
요청은 stout과 같은 스트림이다. 스트림은 읽을수 있거나 쓸 수 있고 때로는 둘 다 가능하다.
모든 스트림은 `EventEmitter`의 인스턴스다.

`require('stream')`을 사용해서 기반 Stream 클래스를 로드할 수 있다.

## Readable Stream

<!--english start-->

<!--type=class-->

A `Readable Stream` has the following methods, members, and events.

<!--english end-->

<!--type=class-->

`Readable Stream`에는 다음과 같은 메서드, 멤버, 이벤트가 있다.

### Event: 'data'

<!--english start-->

`function (data) { }`

The `'data'` event emits either a `Buffer` (by default) or a string if
`setEncoding()` was used.

Note that the __data will be lost__ if there is no listener when a
`Readable Stream` emits a `'data'` event.

<!--english end-->

`function (data) { }`

`'data'` 이벤트는 `Buffer`(기본값)를 발생시키거나 `setEncoding()`가 사용된
경우 문자열을 발생시킨다.

`Readable Stream`가 `'data'` 이벤트를 발생시켰을 때 리스너가 없다면 
__데이터를 잃어버릴 것이다.__

### Event: 'end'

<!--english start-->

`function () { }`

Emitted when the stream has received an EOF (FIN in TCP terminology).
Indicates that no more `'data'` events will happen. If the stream is also
writable, it may be possible to continue writing.

<!--english end-->

`function () { }`

스트림이 EOF(TCP 용어로 FIN)를 받았을 때 발생한다.
더이상 `'data'` 이벤트가 발생하지 않는다는 것을 나타낸다. 스트림이 쓰기도 가능하다면
쓰기는 계속해서 가능할 것이다.

### Event: 'error'

<!--english start-->

`function (exception) { }`

Emitted if there was an error receiving data.

<!--english end-->

`function (exception) { }`

데이터를 받는데 오류가 있으면 발생한다.

### Event: 'close'

<!--english start-->

`function () { }`

Emitted when the underlying file descriptor has been closed. Not all streams
will emit this.  (For example, an incoming HTTP request will not emit
`'close'`.)

<!--english end-->

`function () { }`

기반이 되는 파일 디스크립터가 닫혔을 때 발생한다. 모든 스트림이 이 이벤트를 발생시키는 것은
아니다. (예를 들어 들어오는 HTTP 요청은 `'close'` 이벤트를 발생시키지 않을 것이다.)

### stream.readable

<!--english start-->

A boolean that is `true` by default, but turns `false` after an `'error'`
occurred, the stream came to an `'end'`, or `destroy()` was called.

<!--english end-->

`true`가 기본값인 불리언이지만 `'error'`가 발생하거나 스트림이 `'end'`가 되거나 
`destroy()`이 호출된 뒤에 `false`로 바뀐다.

### stream.setEncoding(encoding)

<!--english start-->

Makes the data event emit a string instead of a `Buffer`. `encoding` can be
`'utf8'`, `'ascii'`, or `'base64'`.

<!--english end-->

data 이벤트가 `Buffer` 대신 문자열을 발생시키도록 한다. `encoding`은 `'utf8'`, 
`'ascii'`, `'base64'`가 될 수 있다.

### stream.pause()

<!--english start-->

Pauses the incoming `'data'` events.

<!--english end-->

들어오는 `'data'` 이벤트를 멈춘다.

### stream.resume()

<!--english start-->

Resumes the incoming `'data'` events after a `pause()`.

<!--english end-->

`pause()`후에 들어오는 `'data'` 이벤트를 다시 시작한다.

### stream.destroy()

<!--english start-->

Closes the underlying file descriptor. Stream will not emit any more events.

<!--english end-->

기반이 되는 파일 디스크립터를 닫는다. 스트림은 더이상 어떤 이벤트로 발생키시지 않을 것이다.

### stream.pipe(destination, [options])

<!--english start-->

This is a `Stream.prototype` method available on all `Stream`s.

Connects this read stream to `destination` WriteStream. Incoming
data on this stream gets written to `destination`. The destination and source
streams are kept in sync by pausing and resuming as necessary.

This function returns the `destination` stream.

Emulating the Unix `cat` command:

    process.stdin.resume();
    process.stdin.pipe(process.stdout);


By default `end()` is called on the destination when the source stream emits
`end`, so that `destination` is no longer writable. Pass `{ end: false }` as
`options` to keep the destination stream open.

This keeps `process.stdout` open so that "Goodbye" can be written at the end.

    process.stdin.resume();

    process.stdin.pipe(process.stdout, { end: false });

    process.stdin.on("end", function() {
      process.stdout.write("Goodbye\n");
    });

<!--english end-->

이 메서드는 모든 `Stream`에서 사용할 수 있는 `Stream.prototype` 메서드이다.

이 읽는 스트림을 `destination` 작성할 스트림에 연결한다. 이 스트림에 들어오는 데이터는
`destination`에 쓰여진다. 목적지 스트림과 출처 스트림은 필요에 따라 멈추거나 다시 
시작함으로써 동기화가 유지된다.

이 함수는 `destination` 스트림을 반환한다.

Unix의 `cat` 명령어를 에뮬레이팅한다.:

    process.stdin.resume();
    process.stdin.pipe(process.stdout);


출처 스트림이 `end`를 발생하면 기본적으로 목적지 스트림도 `end()`를 호출하므로 
`destination`는 더이상 쓸 수 없다. 목적지 스트림을 열려진 상태로 놔두려면
`options`으로 `{ end: false }`를 전달한다.

이는 마지막에 "Goodbye"를 작성할 수 있도록 `process.stdout`를 열어둔 채로 놔둔다.

    process.stdin.resume();

    process.stdin.pipe(process.stdout, { end: false });

    process.stdin.on("end", function() {
      process.stdout.write("Goodbye\n");
    });


## Writable Stream

<!--english start-->

<!--type=class-->

A `Writable Stream` has the following methods, members, and events.

<!--english end-->

<!--type=class-->

`Writable Stream`에는 다음의 메서드, 멤버, 이벤트가 있다.

### Event: 'drain'

<!--english start-->

`function () { }`

After a `write()` method returned `false`, this event is emitted to
indicate that it is safe to write again.

<!--english end-->

`function () { }`

`write()` 메서드가 `false`를 반환한 뒤에 다시 작성할 수 있는 상태를
알리기 위해 이 이벤트를 발생시킨다.

### Event: 'error'

<!--english start-->

`function (exception) { }`

Emitted on error with the exception `exception`.

<!--english end-->

`function (exception) { }`

오류가 있을 때 예외 `exception`과 함께 발생시킨다.

### Event: 'close'

<!--english start-->

`function () { }`

Emitted when the underlying file descriptor has been closed.

<!--english end-->

`function () { }`

기반이 되는 파일 디스크립터가 닫혔을 때 발생한다.

### Event: 'pipe'

<!--english start-->

`function (src) { }`

Emitted when the stream is passed to a readable stream's pipe method.

<!--english end-->

`function (src) { }`

스트림이 읽을 수 있는 스트림의 pipe 메서드에 전달되었을 때 발생한다.

### stream.writable

<!--english start-->

A boolean that is `true` by default, but turns `false` after an `'error'`
occurred or `end()` / `destroy()` was called.

<!--english end-->

기본값이 `true`인 불리언이지만 `'error'`가 발생하거나 `end()` / `destroy()`가 
호출된 후에 `false`로 바뀐다.

### stream.write(string, [encoding], [fd])

<!--english start-->

Writes `string` with the given `encoding` to the stream.  Returns `true` if
the string has been flushed to the kernel buffer.  Returns `false` to
indicate that the kernel buffer is full, and the data will be sent out in
the future. The `'drain'` event will indicate when the kernel buffer is
empty again. The `encoding` defaults to `'utf8'`.

If the optional `fd` parameter is specified, it is interpreted as an integral
file descriptor to be sent over the stream. This is only supported for UNIX
streams, and is silently ignored otherwise. When writing a file descriptor in
this manner, closing the descriptor before the stream drains risks sending an
invalid (closed) FD.

<!--english end-->

주어진 `encoding`으로 스트림에 `string`를 작성한다. 문자열이 커널 버퍼에 플러시되면
`true`를 반환한다. 커널 버퍼가 꽉차면 데이터가 차후에 보내진다는 것을 알리기 위해
`false`를 반환한다. 커널버퍼가 다시 비워지면 `'drain'` 이벤트로 알려줄 것이다.
`encoding`의 기본값은 `'utf8'`이다.

선택사항인 `fd` 파라미터를 지정하면 스트림에 보낸 필수 파일디스크립터로 해석된다.
이는 UNIX 스트림에서만 지원하고 다른 스트림에서는 경고없이 무시될 것이다. 이 방법으로
파일 디스크립터를 작성할 때 스트림의 drain 이전에 해당 디스크립터를 닫으면 유효하지
않은(닫힌) FD에 데이터를 보낼 위험이 있다.

### stream.write(buffer)

<!--english start-->

Same as the above except with a raw buffer.

<!--english end-->

raw buffer를 사용한다는 점만 빼고는 위와 동일하다.

### stream.end()

<!--english start-->

Terminates the stream with EOF or FIN.
This call will allow queued write data to be sent before closing the stream.

<!--english end-->

EOF나 FIN로 스트림을 종료시킨다.
이 함수를 호출하면 스트림을 닫기 전에 큐에 있는 작성 데이터를 보낼 것이다.

### stream.end(string, encoding)

<!--english start-->

Sends `string` with the given `encoding` and terminates the stream with EOF
or FIN. This is useful to reduce the number of packets sent.

<!--english end-->

주어진 `encoding`으로 `string`을 보내고 EOF나 FIN로 스트림을 종료시킨다.
이는 보내는 패킷의 수를 줄이는 데 유용하다.

### stream.end(buffer)

<!--english start-->

Same as above but with a `buffer`.

<!--english end-->

위와 동일하지만 `buffer`를 사용한다.

### stream.destroy()

<!--english start-->

Closes the underlying file descriptor. Stream will not emit any more events.
Any queued write data will not be sent.

<!--english end-->

기반이 되는 파일 디스크립터를 닫는다. 스트림은 더이상 어떤 이벤트를 발생시키지 않을 것이다.
큐에 있는 어떤 데이터도 보내지 않을 것이다.

### stream.destroySoon()

<!--english start-->

After the write queue is drained, close the file descriptor. `destroySoon()`
can still destroy straight away, as long as there is no data left in the queue
for writes.

<!--english end-->

작성 큐를 소모한 후에 파일 디스크립터를 닫는다. 작성 큐에 남아있는 데이터가 없다면
`destroySoon()`는 즉시 파일 디스크립터를 닫을 수 있다.
