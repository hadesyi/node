# Stream

    Stability: 2 - Unstable

스프림은 Node에서 여러 가지 객체로 구현되는 추상 인터페이스다. 예를 들어 HTTP 서버에 대한
요청은 stout과 같은 스트림이다. 스트림은 읽을수 있거나 쓸 수 있고 때로는 둘 다 가능하다.
모든 스트림은 [EventEmitter][]의 인스턴스다.

`require('stream')`을 사용해서 기반 Stream 클래스를 로드할 수 있다.
Readable 스트림, Writable 스트림, Duplex 스트림, Transform 스트림을 위한
기반 클래스들이 존재한다.

## Compatibility

Node의 이전 버전에서는 Readable 스트림 인터페이스가 더 간단했지만
강력하지도 않았고 많이 유용하지도 않았다.

* `read()` 메서드를 호출하기를 기다리기 보다는 `'data'` 이벤트를 즉시 발생시키기
  시작할 것이다. 데이터를 처리할 방법을 결정하려고 어떤 I/O를 수행해야 한다면
  청크를 잃어버리지 않도록 어떤 종류의 버터에 저장해야 한다.
* `pause()` 메서드는 보장되지 않고 오히려 권고사항이었다. 즉, 스트림이 멈춰진
  상태에 있을 때 조차도 `'data'`를 받을 준비를 계속 하고 있어야 한다.

Node v0.10에서는 아래에서 설명하는 Readable 클래스가 추가되었다. 이전의 Node 프로그램과의
하위호환성을 맞추기 위해 `'data'` 이벤트 핸들러를 추가하거나 `pause()`나 `resume()`
메서드를 호출했을 때 Readable 스트림은 "old mode"로 변경된다. 그래서 새로운 `read()`
메서드와 `'readable'` 이벤트를 사용하지 않더라도 `'data'` 청크를 잃어버릴 걱정을 하지
않아도 되는 효과가 있다.

대부분의 프로그램은 계속 정상적으로 동작할 것이다. 하지만 다음 상황에서는 특수한 경우
(edge case)가 생긴다.

* 추가한 `'data'` 이벤트 핸들러가 없다.
* `pause()`와 `resume()` 메서드를 전혀 호출하지 않는다.

예를 들어서 다음 코드를 보자.

```javascript
// WARNING!  BROKEN!
net.createServer(function(socket) {

  // 'end' 메서드를 추가했지만 데이터를 전혀 소비하지 않는다
  socket.on('end', function() {
    // 절대 이곳에 도달하지 않는다
    socket.end('I got your message (but didnt read it)\n');
  });

}).listen(1337);
```

Node v0.10 이전의 버전에서는 들어오는 메시지 데이터를 그냥 버려버릴 것이다.
하지만 Node v0.10이상에서는 소켓이 영원히 멈춰진 상태로 남아있을 것이다.

이 상황을 피하려면 "old mode" 동작이 실행되도록 `resume()` 메서드를 호출해라.

```javascript
// 우회방법
net.createServer(function(socket) {

  socket.on('end', function() {
    socket.end('I got your message (but didnt read it)\n');
  });

  // 데이터 흐름을 시작하고 데이터를 버린다
  socket.resume();

}).listen(1337);
```

새로운 Readable 스트림을 old mode로 변경하는 것과 관련해서 v0.10 이전 방식의 스트림을
`wrap()` 메서드를 사용해서 Readable 클래스로 감쌀 수 있다.

## Class: stream.Readable

<!--type=class-->

`Readable Stream`은 다음 메서드, 멤버, 이벤트를 가진다.

`stream.Readable`는 의존하는 `_read(size)` 메서드의 구현체로 확장하도록
설계된 추상클래스다.(아래 참고)

### new stream.Readable([options])

* `options` {Object}
  * `highWaterMark` {Number} 의존 리소스를 읽어들이는 것을 중단하기 전에 내부 버퍼에
    저장할 수 있는 최대 바이트 수. 기본값=16kb
  * `encoding` {String} 지정하면 지정한 인코딩을 사용해서 버퍼를 문자열로 디코딩할
     것이다. 기본값=null
  * `objectMode` {Boolean} 해당 스트림이 객체의 스트림처럼 동작해야 하는지에 대한 여부.
    즉, stream.read(n)는 n 크기의 Buffer 대신 단일 값을 반환한다.

Readable 클래스를 확장한 클래스에서는 버퍼링 설정을 적절하게 초기화할 수 있도록
생성자를 호출해야 한다.

### readable.\_read(size)

* `size` {Number} 비동기적으로 읽을 바이트 수

Note: **이 함수는 직접 호출하지 말아야 한다.**  이 자식 프로세스가 구현해야 하고
내부 Readable 클래스 메서드만 호출해야 한다.

모든 Readable 스트림 구현체는 의존하는 리소스에서 데이터를 가져오는 `_read` 메서드를
반드시 제공해야 한다.

이 메서드는 메서드를 정의한 클래스 내부에서 사용하고 사용자 프로그램이 직접 호출하면
안되기 때문에 언더스코어로 시작한다. 하지만 자신의 확장 클래스에서 이 메서드를
오버라이드할 수 있다.

데이터를 사용할 수 있을 때 `readable.push(chunk)`를 호출해서 읽기 큐에 데이터를
넣어라. `push`가 false를 반환하면 읽기를 멈춰야 한다. `_read`를 다시 호출했을 때
추가적인 데이터를 넣기(push) 시작해야 한다. TCP나 TLS처럼 관련이 없는 구현체는
이 인자를 무시하고 데이터를 이용할 수 있을 때마다 데이터를 가져올 것이다. 예를 들어
`stream.push(chunk)`를 호출하기 전 `size` 바이트를 사용할 수 있을 때까지
"기다릴" 필요가 없다.

### readable.push(chunk, [encoding])

* `chunk` {Buffer | null | String} 읽기 큐에 넣을 데이터의 청크
* `encoding` {String} 문자열 청크의 인코딩. `'utf8'`이나 `'ascii'`처럼
  유효한 Buffer 인코딩이어야 한다
* return {Boolean} 추가적인 push를 수행해야 하는지 여부

Note: **이 함수는 Readable 스트림의 컨슈머가 아닌 Readable 구현체가 호출해야 한다.**
최소 하나의 `push(chunk)` 호출이 이뤄질 때까지 `_read()` 함수를 다시 호출하지 않을 것이다.
사용할 수 있는 데이터가 없다면 큐에 데이터를 추가히자 않고 차후의 `_read`를 호출할 수 있도록
`push('')`(빈 문자열)를 호출할 것이다.

`Readable` 클래스는 나중에 `'readable'` 이벤트가 발생했을 때 `read()` 메서드를 호출해서
데이터를 가져가도록 읽기 큐에 데이터를 넣는 방식으로 동작한다.

`push()` 메서드는 명시적으로 읽기 큐에 데이터를 추가할 것이다. `push()` 메서드를 `null`로
호출하면 데이터가 끝났다는 신호가 될 것이다.

일부의 경우에는 어떤 종류의 pause/resume 메카니즘과 데이터 콜백을 가진 저수준 소스를
감싸고 있을 것이다. 이러한 경우에 다음과 같이 해서 저수준 소스 객체를 감쌀 수 있다.

```javascript
// source는 readStop(), readStart() 메서드와
// source가 데이터를 가졌을 때 호출되는 `ondata` 멤버,
// 데이터가 끝났을 때 호출되는 `onend` 멤버를 가진 객체이다.

var stream = new Readable();

source.ondata = function(chunk) {
  // push()가 false를 반환하면 소스에서 읽는 것을 멈춰야 한다
  if (!stream.push(chunk))
    source.readStop();
};

source.onend = function() {
  stream.push(null);
};

// 이 경우에 권고사항인 size 인자를 무시하고 스트림이 추가적인 데이터를
// 가져오도록 하고 싶을 때 _read를 호출할 것이다.
stream._read = function(n) {
  source.readStart();
};
```

### readable.unshift(chunk)

* `chunk` {Buffer | null | String} 읽기 큐로 unshift하는 데이터의 청크
* return {Boolean} 추가적인 push를 수행해야 하는지 여부

Note: **이 함수는 보통 Readable 하위클래스의 구현체가 아니라 Readable 컨슈머가 호출할
것이다.** 이는 `readable.push(chunk)`가 하는 방법으로 `_read()` 트랜잭션의 끝을 나타내지
않는다. Readable 클래스에서 `this.unshift(chunk)`를 호출해야 하는지 찾는다면
[stream.Transform](#stream_class_stream_transform) 클래스를 대신 사용해야하는
좋은 기회이다.

이 메서드는 `readable.push(chunk)` 계열이다. 읽기 큐의 *끝*에 데이터를 두기 보다는
읽기 큐의 *앞*에 데이터를 둔다.

이는 소스에서 낙관적으로 가져온 "소비하지 않는" 일부 데이터가 필요한 파서가 스트림을 소비하는
경우 유용하므로 스트림을 다른 파티(party)로 전달할 수 있다.

```javascript
// 간단한 데이터 프로토콜에 대한 파서.
// "header"는 JSON 객체이고 이어서 두개의 \n 문자가 오로고 이어서 메시지 바디가 온다.
//
// Note: 이는 Transform 스트림으로 훨씬 간단히 할 수 있다!
// 이를 위해 Readable를 직접 사용하는 것이 차선책이다. 이에 대한 예제은 아래의
// Transform 부분을 참고해라.

function SimpleProtocol(source, options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Readable.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;

  // 소스는 소켓이나 파일같은 readable 스트림이다
  this._source = source;

  var self = this;
  source.on('end', function() {
    self.push(null);
  });

  // 소스를 읽을 수 있을 때마다 무시한다.
  // read(0)은 바이트를 전혀 소비하지 않을 것이다
  source.on('readable', function() {
    self.read(0);
  });

  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype = Object.create(
  Readable.prototype, { constructor: { value: SimpleProtocol }});

SimpleProtocol.prototype._read = function(n) {
  if (!this._inBody) {
    var chunk = this._source.read();

    // 소스에 데이터가 없으면 아직 데이터를 갖지 않는다.
    if (chunk === null)
      return this.push('');

    // 청크가 \n\n를 가졌는지 검사한다
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // 아직 \n\n를 기다리고 있다
      // 청크를 치워두고 다시 시도한다.
      this._rawHeader.push(chunk);
      this.push('');
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // 이제 어떤 추가적인 데이터를 가졌으므로 컨슈머가 볼 수 있도록
      // 읽기 큐에 남은 부분을 다시 unshift한다.
      var b = chunk.slice(split);
      this.unshift(b);

      // 그리고 헤더 파싱을 완료했다는 것을 알려준다.
      this.emit('header', this.header);
    }
  } else {
    // 여기서 단순히 컨슈머에게 데이터를 제공한다.
    // push(null)이 EOF를 의미하므로 push(null)을 사용하지 않도록 조심해라.
    var chunk = this._source.read();
    if (chunk) this.push(chunk);
  }
};

// 사용방법:
var parser = new SimpleProtocol(source);
// 이제 파서는 파싱된 헤더 데이터를 가진 'header'를 발생시킬
// readable 스트림이다.
```

### readable.wrap(stream)

* `stream` {Stream} An "old style" readable stream

`'data'` 이벤트를 발생시키고 권고 전용인 `pause()` 메서드를 가진 오래된 Node 라이브러리를
사용한다면 데이터 소스로 과거의 스트림을 사용하는 Readable 스트림을 생성하는 `wrap()`
메서드를 사용할 수 있다.

예를 들어

```javascript
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```

### Event: 'readable'

사용될 준비가 된 데이터가 있을 때 이 이벤트가 발생할 것이다.

이 이벤트가 발생하면 데이터를 소비하는 `read()` 메서드를 호출한다.

### Event: 'end'

스트림이 EOF(TCP 용어로 FIN)를 받았을 때 발생한다.
더이상 `'data'` 이벤트가 발생하지 않는다는 것을 나타낸다. 스트림이 쓰기도 가능하다면
쓰기는 계속해서 가능할 것이다.

### Event: 'data'

`'data'`이벤트를 `Buffer`(기본값)이나 `setEncoding()`을 사용한 경우에는 문자열을
발생시킨다.

`'data'` 이벤트 리스너를 추가하면 Readable 스트림을 데이터를 소비하려고 `read()`를
호출하기를 기다리기 보다는 데이터가 이용가능하자마자 데이터를 발생시키는 "old mode"로
변경한다.

### Event: 'error'

데이터를 받는데 오류가 있으면 발생한다.

### Event: 'close'

의존하는 리소스(예를 들면 파일 디스크립터)가 닫혔을 때 발생한다. 모든 스트림이
이 이벤트를 발생키시는 것은 아니다.

### readable.setEncoding(encoding)

`'data'` 이벤트가 `Buffer` 대신 문자열을 발생시키도록 한다. `encoding`은 `'utf8'`,
`'utf16le'` (`'ucs2'`), `'ascii'`, `'hex'`가 될 수 있다.

생성자에 `encoding` 필드를 지정해서 인코딩을 설정할 수도 있다.

### readable.read([size])

* `size` {Number | null} 읽어들일 바이트 수. 선택적인 값이다.
* Return: {Buffer | String | null}

Note: **이 함수는 Readable 스트림 사용자가 호출하지 말아야 한다.**

`'readable'` 이벤트가 발생하고 나면 데이터를 소비하기 위해서 이 메서드를 호출해라.

`size`는 최대 바이트 수를 지정할 것이다. 설정하지 않으면 내부 버퍼의 전체 내용을 반환한다.

소비할 데이터가 없거나 내부 버퍼에 `size` 인자보다 적은 바이트가 존재할 경우에는
`null`을 반환하고 데이터를 더 이용하게 되었을 때 `'readable'` 이벤트가 발생할 것이다.

`stream.read(0)`를 호출하면 항상 `null`를 반환한고 내부 버터를 갱신할 것이지만
그렇지 않으면 작업을 수행하지 않는다.

### readable.pipe(destination, [options])

* `destination` {Writable Stream}
* `options` {Object} 선택적인 값
  * `end` {Boolean} Default=true

해당 readable 스트림을 `destination` WriteStream으로 연결한다. 이 스트림으로
들어오는 데이터는 `destination`에 쓰여진다. 빠른 readable 스트림이 느린 목적지 스트림을
압도하지 않도록 역류를 적절히 관리해라.

이 함수는 `destination` 스트림을 반환한다.

예를 들어 Unix `cat` 명령어를 에뮬레이팅한다고 해보자.

    process.stdin.pipe(process.stdout);

출처 스트림이 `end`를 발생하면 기본적으로 목적지 스트림도 `end()`를 호출하므로
`destination`는 더이상 쓸 수 없다. 목적지 스트림을 열려진 상태로 놔두려면
`options`으로 `{ end: false }`를 전달한다.

이는 마지막에 "Goodbye"를 작성할 수 있도록 `writer`를 열어둔 채로 놔둔다.

    reader.pipe(writer, { end: false });
    reader.on("end", function() {
      writer.end("Goodbye\n");
    });

지정한 옵션에 상관없이 프로세스가 종료될 때까지 `process.stderr`와 `process.stdout`는
절대로 닫히지 않는다는 점을 유념해라.

### readable.unpipe([destination])

* `destination` {Writable Stream} 선택적인 값

이전에 만들어진 `pipe()`를 되돌린다. destination을 제공하지 않으면
이전에 만들어진 모든 파이프를 제거한다.

### readable.pause()

readable 스트림을 `read()` 메서드를 통해 버퍼링된 데이터를 소비하기 보다는
`'data'` 이벤트를 발생시키는 "old mode"로 변경한다.

데이터의 흐름을 중단한다. 스트림이 멈춰진 상태에 있는 동안에는 `'data'` 이벤트를
발생시키지 않는다.

### readable.resume()

readable 스트림을 `read()` 메서드를 통해 버퍼링된 데이터를 소비하기 보다는
`'data'` 이벤트를 발생시키는 "old mode"로 변경한다.

`pause()`후에 들어오는 데이터에 대한 `'data'` 이벤트를 복구한다.


## Class: stream.Writable

<!--type=class-->

`Writable` 스트림에는 다음의 메서드, 멤버, 이벤트가 있다.

`stream.Writable`은 의존하는 `_write(chunk, encoding, cb)` 메서드의
구현체로 확장하도록 설계된 추상클래스다.(아래 참고)

### new stream.Writable([options])

* `options` {Object}
  * `highWaterMark` {Number} `write()`가 false를 반환하기 시작했을 때의
    버퍼 수준(Buffer level). 기본값=16kb
  * `decodeStrings` {Boolean} `_write()`에 전달하기 전에 문자열을 Buffer로
    디코딩할 지 여부. 기본값=true

Writable 클래스를 확장한 클래스에서는 버퍼링 설정을 적절히 초기화할 수 있도록
생성자를 호출해야 한다.

### writable.\_write(chunk, encoding, callback)

* `chunk` {Buffer | String} 작성할 청크. `decodeStrings` 옵션을
  `false`로 설정하지 않으면 항상 버퍼다.
* `encoding` {String} 청크가 문자열인 경우 인코딩 타입. 청크가 버퍼이면 무시한다.
  `decodeStrings` 옵션을 명시적으로 `false`로 설정하지 않으면 **항상** 버퍼다.
* `callback` {Function} 제공된 청크의 처리를 완료했을 때 이 함수를 호출한다.
   (선택적인 오류 인자와 함께)

모든 Writable 스트림 구현체는 의존하는 리소스에 데이터를 보내는 `_write` 메서드를
제공해야 한다.

Note: **이 함수는 절대로 직접 호출하지 말아야 한다.**  이는 자식 클래스가 구현하거나
내부 Writable 클래스 메서드만 호출해야 한다.

쓰기 작업이 성공적으로 완료되었거나 오류가 발생했다는 것을 알리기 위해서
표준 `callback(error)` 패턴으로 callback을 호출한다.

생성자 옵션으로 `decodeStrings` 플래그를 설정하면 `chunk`는 Buffer가 아니라 문자열이
될 것이고 `encoding`는 문자열의 종류를 나타낼 것이다. 이는 특정 문자열 데이터 인코딩을
최적화해서 처리하는 구현체를 지원한다. 명시적으로 `decodeStrings` 옵션을 `false`로
설정하지 않았다면 `encoding` 인자는 그냥 무시하고 `chunk`가 항상 Buffer라고 가정한다.

이 메서드는 메서드를 정의한 클래스 내부에서 사용하고 사용자 프로그램이 직접 호출하면
안되기 때문에 언더스코어로 시작한다. 하지만 자신의 확장 클래스에서 이 메서드를
오버라이드할 수 있다.


### writable.write(chunk, [encoding], [callback])

* `chunk` {Buffer | String} 작성할 데이터
* `encoding` {String} 선택적인 값. `chunk`가 문자열이면
  기본 인코딩은 `'utf8'`이다.
* `callback` {Function} 선택적인 값.  해당 청크를 성공적으로
  썼을 때 호출된다.
* Returns {Boolean}

`chunk`를 스트림에 작성한다. 의존하는 리소스로 데이터가 모두 나가면 `true`를 반환한다.
버퍼가 꽉 찼찼고 데이터는 나중에 보낼 것이라는 의미로 `false`를 반환한다. `'drain'`
이벤트는 버퍼가 다시 비워졌을 때를 의미한다.

`write()`가 false를 반환하는 자세한 내용은 생성자에 제공한 `highWaterMark`
옵션으로 결정한다.

### writable.end([chunk], [encoding], [callback])

* `chunk` {Buffer | String} 선택적인 값으로 작성할 마지막 데이터
* `encoding` {String} 선택적인 값.  `chunk`가 문자열이면
  기본 인코딩은 `'utf8'`이다.
* `callback` {Function} 선택적인 값.  마지막 청크를 성공적으로
  썼을 때 호출한다.

마지막 데이터가 스트림에 쓰여졌다는 신호로 이 메서드를 호출한다.

### Event: 'drain'

스트림의 쓰기 큐가 비워지고 다시 버퍼링하지 않고 안전하게 쓸 수 있게 되었을 때 발생한다.
`stream.write()`이 `false`를 반환했을 때 리스닝한다.

### Event: 'error'

데이터를 받는데 오류가 있으면 발생한다.

### Event: 'close'

의존하는 리소스(예를 들면 지원하는 파일 디스크립터)가 닫혔을 때 발생한다.
모든 스트림이 이 이벤트를 발생시키는 것은 아니다.

### Event: 'finish'

`end()`가 호출되고 더이상 작성한 청크가 없을 때 이 이벤트가 발생한다.

### Event: 'pipe'

* `source` {Readable Stream}

스트림이 읽을 수 있는 스트림의 pipe 메서드에 전달되었을 때 발생한다.

### Event 'unpipe'

* `source` {Readable Stream}

소스 Readable 스트림의 `unpipe()` 메서드를 사용해서 이전에 만들어진 `pipe()`를
제거했을 때 발생한다.

## Class: stream.Duplex

<!--type=class-->

"duplex" 스트림은 TCP 소켓연결처럼 Readable하면서 Writable한 스트림이다.

`stream.Duplex`는 Readable 스트림 클래스나 Writable 스트림 클래스에서처럼
의존하는 `_read(size)`와 `_write(chunk, encoding, callback)` 메서드의
구현체로 확장하도록 설계된 추상클래스이다.

JavaScript가 프로토타입 다중 상속을 지원하지 않으므로 이 클래스는 Readable을
프로토타입으로 상속받고 Writable이 기생하도록 한다. 그러므로 확장 듀플렉스 클래스에서
저수준 `_write(chunk, encoding, cb)` 메서드와 저수준 `_read(n)` 메서드를
모두 구현하는 것은 개발자의 몫이다.

### new stream.Duplex(options)

* `options` {Object} Writable과 Readable 생성자 모두에 전달된다.
  다음의 필드를 가진다.
  * `allowHalfOpen` {Boolean} 기본값=true.  `false`로 설정하면 스트림은
    쓰기 쪽이 끝났을 때 읽기 쪽도 자동으로 종료하고 그 반대도 마찬가지다.

Duplex 클래스를 확장한 클래스에서 버퍼링 설정을 적절하게 초기화할 수 있도록
생성자를 호출해야 한다.

## Class: stream.Transform

"transform" 스트림은 zlib 스트림이나 crypto 스트림처럼 출력이 원인이 되어
어떤 방법으로 입력에 연결되는 duplex 스트림이다.

출력이 입력과 같은 크기이거나 청크수가 같아야 하거나 동시에 도착해야할 필요는 없다.
예를 들어 Hash 스트림은 입력이 종료되었을 때 제공되는 출력의 단일 청크만을 가질 것이다.
zlib 스트림은 입력보다 훨씬 작거나 훨씬 더 큰 출력을 만들 것이다.

`_read()`와 `_write()` 메서드를 구현하는 대신 Transform 클래스는 `_transform()`
메서드를 구현해야 하고 선택적으로 `_flush()` 메서드도 구현할 수 있다.(하단 참고)

### new stream.Transform([options])

* `options` {Object} Writable과 Readable 생성자에 모두 전달된다.

Transform 클래스를 확장한 클래스에서는 버퍼링 설정을 적절하게 초기화할 수 있도록
생성자를 호출해야 한다.

### transform.\_transform(chunk, encoding, callback)

* `chunk` {Buffer | String} 변환할 청크.  `decodeStrings` 옵션을
  `false`로 설정하지 않는한 항상 버퍼가 될 것이다.
* `encoding` {String} 청크가 문자열인 경우 이 인코딩 타입이다.
  (`decodeStrings` 청크가 버퍼이면 무시한다.)
* `callback` {Function} 제공된 청크의 처리가 끝났을 때 이 함수를 호출한다.
  (선택적으로 오류 인자와 함께)

Note: **이 함수는 절대로 직접 호출하지 말아야 한다.**  자식 클래스가 구현하거나
내부 Transform 클래스 메서드만 호출해야 한다.

모든 Transform 스트림 구현체는 입력을 받고 출력을 생산하도록 `_transform`를
제공해야 한다.

해당 Transform 클래스가 무엇을 하던지 간에 쓰여진 바이트를 다루고 인터페이스의 읽기
쪽에 이를 전달하도록 `_transform`을 수행해야 한다. 비동기 I/O를 수행하거나
어떤 것을 처리하는 등이다.

해당 입력 청크에서 출력을 생성하도록 해당 청크의 결과로 얼마나 많은 데이터를 출력하기
원하는지에 따라 `transform.push(outputChunk)`를 0번이나 여러번 호출해라.

현재 청크가 완전히 소비되었을 때만 callback 함수를 호출한다. 특정 입력 청크의
결과와 출력이 같을 수도 있고 같지 않을수도 있다.

이 메서드는 메서드를 정의한 클래스 내부에서 사용하고 사용자 프로그램이 직접 호출하면
안되기 때문에 언더스코어로 시작한다. 하지만 자신의 확장 클래스에서 이 메서드를
오버라이드할 수 있다.

### transform.\_flush(callback)

* `callback` {Function} 남아있는 데이터를 모두 내보냈을 때 이 함수를 호출한다.
  (선택적으로 오류 인자와 함께)

Note: **이 함수는 절대로 직접 호출하지 말아야 한다.**  자식 클래스가 구현할 수도 있는데
자식 클래스가 구현했다면 내부 Transform 클래스 메서드만 호출할 것이다.

일부의 경우 변환 작업이 스트림의 끝에서 약간 더 많은 데이터를 발생시키도록 해야할 수도 있다.
예를 들어 `Zlib` 압축 스트림은 출력을 최적화해서 압축할 수 있도록 어떤 내부 상태를 저장할
것이다. 하지만 결국에는 데이터가 완료될 수 있도록 남아있는 것으로 최선의 작업을 해야할
필요가 있다.

이러한 경우 쓰진 데이터를 모두 소비했지만 읽기 쪽에 끝났음을 알리는 `end`를 발생하기 전인
아주 마지막에 호출될 `_flush` 메서드를 구현할 수 있다. `_transform`와 마찬가지로 적절하게
0번이나 여러번 `transform.push(chunk)`를 호출하고 flush작업이 완료되었을 때
`callback`을 호출해라.

이 메서드는 메서드를 정의한 클래스 내부에서 사용하고 사용자 프로그램이 직접 호출하면
안되기 때문에 언더스코어로 시작한다. 하지만 자신의 확장 클래스에서 이 메서드를
오버라이드할 수 있다.

### Example: `SimpleProtocol` parser

위의 간단한 프로토콜 파서 에제를 고수준의 `Transform` 스트림 클래스를 사용해서
훨씬 더 간단하게 구현할 수 있다.

이 예제에서는 인자로 입력을 제공하기 보다는 더 이상적인 Node 스트림 접근은로
파서에 파이프를 연결한다.

```javascript
function SimpleProtocol(options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Transform.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;
  this._rawHeader = [];
  this.header = null;
}

SimpleProtocol.prototype = Object.create(
  Transform.prototype, { constructor: { value: SimpleProtocol }});

SimpleProtocol.prototype._transform = function(chunk, encoding, done) {
  if (!this._inBody) {
    // 청크가 \n\n를 가졌는지 검사한다
    var split = -1;
    for (var i = 0; i < chunk.length; i++) {
      if (chunk[i] === 10) { // '\n'
        if (this._sawFirstCr) {
          split = i;
          break;
        } else {
          this._sawFirstCr = true;
        }
      } else {
        this._sawFirstCr = false;
      }
    }

    if (split === -1) {
      // 여전히 \n\n를 기다린다
      // 청크를 치워두고 다시 시도한다.
      this._rawHeader.push(chunk);
    } else {
      this._inBody = true;
      var h = chunk.slice(0, split);
      this._rawHeader.push(h);
      var header = Buffer.concat(this._rawHeader).toString();
      try {
        this.header = JSON.parse(header);
      } catch (er) {
        this.emit('error', new Error('invalid simple protocol data'));
        return;
      }
      // 헤더 피싱이 끝났음을 알려준다.
      this.emit('header', this.header);

      // 일부 추가적인 데이터를 얻었으므로 이를 먼저 발생시킨다.
      this.push(chunk.slice(split));
    }
  } else {
    // 여기서 기존처럼 컨슈머에 데이터를 제공한다.
    this.push(chunk);
  }
  done();
};

var parser = new SimpleProtocol();
source.pipe(parser)

// 이제 파서는 파싱된 헤더 데이터를 가진 'header'를 발생시킬
// readable 스트림이다.
```


## Class: stream.PassThrough

이는 입력 바이트를 출력으로 단순히 전달하는 `Transform` 스트림의 별로 중요치 않은 구현체이다.
이 클래스의 목적은 주로 예제나 테스트이지만 가끔은 유용한 경우가 있다.


[EventEmitter]: events.html#events_class_events_eventemitter
