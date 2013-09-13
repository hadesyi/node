# Stream

    Stability: 2 - Unstable

스트림은 Node에서 여러 가지 객체로 구현되는 추상 인터페이스다. 예를 들어 [HTTP 서버에 대한
요청](http.html#http_http_incomingmessage)은 [stdout][]과 같은 스트림이다.
스트림은 읽을수 있거나 쓸 수 있고 때로는 둘 다 가능하다.
모든 스트림은 [EventEmitter][]의 인스턴스다.

`require('stream')`을 사용해서 기반 Stream 클래스를 로드할 수 있다.
[Readable][] 스트림, [Writable][] 스트림, [Duplex][] 스트림, [Transform][]
스트림을 위한 기반 클래스들이 존재한다.

이 문서는 세 부분으로 나뉜다. 첫번째 부분은 프로그램에서 스트림을 사용하려면 알아야 하는
API를 설명한다. 스트리밍 API를 직접 구현하지 않는다면 이 부분을 건너띌 수 있다.

두번째 부분은 커스텀 스트림을 구현한다면 사용해야 하는 API을 설명한다.
API는 커스텀 스트림을 쉽게 만들 수 있도록 설계되었다.

세번째 부분은 스트림이 어떻게 동작하는지 자세히 살펴보고 여기서 내부 메카니즘과
수정하는 부분을 확실히 알지 못한다면 수정하지 않아야 하는 함수를 설명한다.


## API for Stream Consumers

<!--type=misc-->

스트림은 [Readable][], [Writable][]이 될 수 있고 둘 다([Duplex][]) 가능할 수도 있다.

모든 스트림은 EventEmitter지만 추가적인 커스텀 메서드들과 프로퍼티들이 있고 이는
Readable, Writable, Duplex냐에 따라 약간씩 다르다.

스트림이 Readable이면서 Writable이면 아래 나와있는 모든 메서드와 이벤트를 구현한다.
그러므로 [Duplex][]나 [Transform][]도 이 API에서 완전히 설명하고 있지만
이 둘의 구현은 약간 다르다.

프로그램에서 스트림을 사용하려고 Stream 인터페이스를 구현할 필요는 없다. 프로그램에서
스트리밍 인터페이스를 **구현한다면** 아래의 [API for Stream Implementors][]도
참고해라.

거의 대부분의 Node 프로그램(얼마나 간단하냐에 상관없이)은 어떤 방법으로든 Stream을
사용한다. 다음은 Node 프로그램에서 Stream을 사용하는 예제이다.

```javascript
var http = require('http');

var server = http.createServer(function (req, res) {
  // req는 Readable 스트림인 http.IncomingMessage 이다.
  // res는 Writable 스트림인 http.ServerResponse 이다.

  var body = '';
  // utf8 문자열로 데이터를 받기 원한다.
  // 인코딩을 설정하지 않으면 Buffer 객체를 받을 것이다.
  req.setEncoding('utf8');

  // 리스너를 추가했으면 Readable 스트림은 'data' 이벤트를 발생시킨다.
  req.on('data', function (chunk) {
    body += chunk;
  })

  // end 이벤트는 바디 전체를 받았다고 알려준다.
  req.on('end', function () {
    try {
      var data = JSON.parse(body);
    } catch (er) {
      // json 형식이 잘못됐다.
      res.statusCode = 400;
      return res.end('error: ' + er.message);
    }

    // 사용자에게 데이터를 작성해서 돌려준다.:
    res.write(typeof data);
    res.end();
  })
})

server.listen(1337);

// $ curl localhost:1337 -d '{}'
// object
// $ curl localhost:1337 -d '"foo"'
// string
// $ curl localhost:1337 -d 'not json'
// error: Unexpected token o
```

### Class: stream.Readable

<!--type=class-->

Readable 스트림 인터페이스는 데이터를 읽어 오는 *소스*에 대한 추상화이다. 다시 말하면
Readable 스트림에서 데이터가 온다.

Readable 스트림은 데이터를 받을 준비가 되었다고 알려줄 때까지 데이터를 보내지 않는다.

Readable 스트림에는 **flowing 모드**와 **non-flowing 모드** 두 가지 "모드"가 있다.
flowing mode에서는 기반시스템에서 데이터를 읽어와서 가능한한 빨리 프로그램에 제공한다.
non-flowing 모드에서 데이터 청크를 받으려면 `stream.read()`를 명시적으로 호출해야 한다.

readable 스트림의 예제에는 다음이 포함되어 있다.

* [http responses, on the client](http.html#http_http_incomingmessage)
* [http requests, on the server](http.html#http_http_incomingmessage)
* [fs read streams](fs.html#fs_class_fs_readstream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdout and stderr][]
* [process.stdin][]

#### Event: 'readable'

스트림에서 데이터의 청크를 읽을 수 있을 때 `'readable'` 이벤트를 발생시킬 것이다.

몇몇 경우에서는 `'readable'` 이벤트를 처리할 때 데이터가 기존에 없다면 데이터를
의존 시스템에서 내부 퍼버로 읽도록 할 것이다.

```javascript
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  // 이제 읽을 데이터가 있다.
})
```

내부 버퍼가 비워지면 추가적인 데이터가 있을 때 `readable` 이벤트를 다시 발생시킨다.

#### Event: 'data'

* `chunk` {Buffer | String} 데이터의 청크.

`data` 이벤트 리스너를 추가하면 스트림이 flowing 모드로 바뀌고 데이터를 사용할 수 있게
되면 바로 핸들러로 전달할 것이다.

가능한한 빨리 스트림에서 데이터를 모두 받으려면 다음과 같이 하는 것이 가장 좋은 방법이다.

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
})
```

#### Event: 'end'

더 이상 데이터가 없으면 이 이벤트가 발생한다.

데이터를 완전히 소비하지 않는 한 `end` 이벤트는 **발생하지 않는다**. 이는 flowing
모드로 바꾸거나 끝까지 반복해서 `read()`를 호출해서 데이터를 모두 소비할 수 있다.

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
})
readable.on('end', function() {
  console.log('there will be no more data.');
});
```

#### Event: 'close'

의존 리소스(예를 들어 기반(backing) 파일 디스크립터)가 닫혔을 때 발생한다. 모든 스크림은
이 이벤트를 발생할 것이다.

#### Event: 'error'

데이터를 받을 때 오류가 있으면 발생한다.

#### readable.read([size])

* `size` {Number} 읽을 데이터의 양을 지정하기 위한 선택적인 인자.
* Return {String | Buffer | null}

`read()` 메서드는 내부 버퍼에서 데이터를 가져와서 반환한다. 데이터가 없다면
`null`을 반환할 것이다.

`size` 인자를 전달하면 `size` 만큼의 바이트를 반환할 것이다.
`size` 만큼의 데이터가 없다면 `null`을 반환할 것이다.

`size` 인자를 지정하지 않으면 내부 버퍼의 데이터 모두를 반환할 것이다.

이 메서드는 non-flowing 모드에서만 호출해야 한다. flowing 모드에서
이 메서드는 내부 버퍼가 비워질 때까지 자동으로 호출된다.

```javascript
var readable = getReadableStreamSomehow();
readable.on('readable', function() {
  var chunk;
  while (null !== (chunk = readable.read())) {
    console.log('got %d bytes of data', chunk.length);
  }
});
```

#### readable.setEncoding(encoding)

* `encoding` {String} 사용할 인코딩.

이 함수를 호출하면 스트림이 Buffer 객체 대신 지정한 인코딩의 문자열을 반환하도록 한다.
예를 들어, `readable.setEncoding('utf8')`를 실행하면 출력데이터를 UTF-8로
인터프리팅해서 문자열을 반환한다. `readable.setEncoding('hex')`를 실행하면
데이터를 16진수 문자열 형식으로 인코딩할 것이다.

이 함수는 버퍼를 가져와서 그냥 `buf.toString(encoding)`를 실행하는 경우 엉망이
될 가능성이 있는 멀티바이트 문자를 잘 다룬다. 문자열로 데이터를 읽으려면 이 메서드를
항상 사용해라.

```javascript
var readable = getReadableStreamSomehow();
readable.setEncoding('utf8');
readable.on('data', function(chunk) {
  assert.equal(typeof chunk, 'string');
  console.log('got %d characters of string data', chunk.length);
})
```

#### readable.resume()

이 메서드는 readable 스트림이 다시 `data` 이벤트를 발생키실 수 있게 한다.

이 메서드는 스트림을 flowing-mode로 바꿀 것이다. 스트림에서 데이터를 소비하기를
원치 *않지만* `end` 이벤트는 받기를 *원한다면* 데이터의 흐름(flow)를 여는
`readable.resume()`를 호출할 수 있다.

```javascript
var readable = getReadableStreamSomehow();
readable.resume();
readable.on('end', function(chunk) {
  console.log('got to the end, but did not read anything');
})
```

#### readable.pause()

이 메서드는 flowing-mode의 스트림이 `data` 이벤트 발생을 멈추도록 할 것이다.
사용가능한 모든 데이터는 내부 버퍼에 남아있을 것이다.

이 메서드는 flowing mode에서만 유효한 메서드이다. non-flowing 스트림에서
호출한 경우 flowing mode로 바꾸고 멈춘 채로 유지할 것이다.

```javascript
var readable = getReadableStreamSomehow();
readable.on('data', function(chunk) {
  console.log('got %d bytes of data', chunk.length);
  readable.pause();
  console.log('there will be no more data for 1 second');
  setTimeout(function() {
    console.log('now data will start flowing again');
    readable.resume();
  }, 1000);
})
```

#### readable.pipe(destination, [options])

* `destination` {[Writable][] Stream} 데이터를 쓰는 목적지
* `options` {Object} Pipe 옵션
  * `end` {Boolean} reader가 종료되면 writer도 종료한다. 기본값은 `true`이다.

이 메서드는 readable 스트림에서 오는 데이터를 모두 가져오고 제공된 목적지에 데이터를
작성하고 빠른 readable 스트림이 목적지를 장악하지 않도록 흐름을 자동으로 관리한다.

안전을 위해서 여러 목적지를 파이프로 연결할 수 있다.

```javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// readable에서 오는 모든 데이터는 'file.txt'로 간다
readable.pipe(writable);
```

이 함수는 목적지 스트림을 반환하므로 다음과 같이 파이프 체인을 설정할 수 있다.

```javascript
var r = fs.createReadStream('file.txt');
var z = zlib.createGzip();
var w = fs.createWriteStream('file.txt.gz');
r.pipe(z).pipe(w);
```

예를 들면 다음과 같이 Unix `cat` 명령어를 에뮬레이팅할 수 있다.

```javascript
process.stdin.pipe(process.stdout);
```

기본적으로 소스 스트림이 `end`를 발생시켰을 때 목적지에서 [`end()`][]가 호출되므로
`destination`는 더이상 writable이 아니다. 목적지 스트림을 열어둔 채로 두려면
`options`으로 `{ end: false }`를 전달해라.

이 코드는 마지막에 "Goodbye"를 쓸 수 있게 `writer`를 열어둔 채로 둔다.

```javascript
reader.pipe(writer, { end: false });
reader.on('end', function() {
  writer.end('Goodbye\n');
});
```

`process.stderr`와 `process.stdout`는 옵션에 상관없이 프로세스가 종료되기
전에는 절대 닫히지 않는다.

#### readable.unpipe([destination])

* `destination` {[Writable][] Stream} 파이프를 해제할 스트림(선택적인 값)

이 메서드는 이전에 호출한 `pipe()`를 설정하는 훅을 제거할 것이다.

목적지를 지정하지 않으면 모든 파이프를 제거한다.

목적지를 지정했지만 목적지에 파이프가 설정되어 있지 않으면 아무런 동작도 일어나지 않는다.

```javascript
var readable = getReadableStreamSomehow();
var writable = fs.createWriteStream('file.txt');
// readable에서 오는 모든 데이터를 딱 1초동안만
// 'file.txt'로 보낸다.
readable.pipe(writable);
setTimeout(function() {
  console.log('stop writing to file.txt');
  readable.unpipe(writable);
  console.log('manually close the file stream');
  writable.end();
}, 1000);
```

#### readable.unshift(chunk)

* `chunk` {Buffer | String} 읽기 큐에서 언쉬프트할 데이터의 청크

이 함수는 소스에서 데이터를 가져와서 일부 데이터는 "소비하지 않아야" 하고 스트림을 다른
어딘가로 전달할 수 있는 파서가 스트림을 사용하는 등의 특수한 경우에 유용하다.

프로그램에서 `stream.unshift(chunk)`를 호출해야만 한다면 [Transform][] 스트림을
구현하는 걸 고려해 봐라. (아래 API for Stream Implementors 참고.)

```javascript
// \n\n를 경계로 헤더를 분리한다.
// 너무 많이 받았다면 unshift()를 사용해라.
// (error, header, stream)로 callback을 호출한다.
var StringDecoder = require('string_decoder').StringDecoder;
function parseHeader(stream, callback) {
  stream.on('error', callback);
  stream.on('readable', onReadable);
  var decoder = new StringDecoder('utf8');
  var header = '';
  function onReadable() {
    var chunk;
    while (null !== (chunk = stream.read())) {
      var str = decoder.write(chunk);
      if (str.match(/\n\n/)) {
        // 헤더 경계를 찾았다
        var split = str.split(/\n\n/);
        header += split.shift();
        var remaining = split.join('\n\n');
        var buf = new Buffer(remaining, 'utf8');
        if (buf.length)
          stream.unshift(buf);
        stream.removeListener('error', callback);
        stream.removeListener('readable', onReadable);
        // 이제 메시지의 바디를 스트림에서 읽을 수 있다.
        callback(null, header, stream);
      } else {
        // 아직 헤더를 읽는 중이다
        header += str;
      }
    }
  }
}
```

#### readable.wrap(stream)

* `stream` {Stream} "예전 방식의" readable 스트림

Node v0.10 이전 버전의 스트림은 지금의 스트림 API 전체를 구현하지 않았다. 자세한 내용은
아래 "Compatibility"를 참고해라.)

`'data'` 이벤트를 발생시키고 경고(advisory) 전용인 `pause()` 메서드를 가지는
과거 버전의 Node 라이브러리를 사용하고 있다면 데이터 소스로 과거의 스트림을 사용하는
[Readable][] 스크림을 생성하기 위해 `wrap()` 메서드를 사용할 수 있다.

이 함수를 호출해야 하는 경우는 극히 드물 것이지만 오래된 Node 프로그램과 라이브러리와 상호작용이

예를 들면 다음과 같다.

```javascript
var OldReader = require('./old-api-module.js').OldReader;
var oreader = new OldReader;
var Readable = require('stream').Readable;
var myReader = new Readable().wrap(oreader);

myReader.on('readable', function() {
  myReader.read(); // etc.
});
```


### Class: stream.Writable

<!--type=class-->

Writable 스트림 인터페이스는 데이터를 *작성할* *목적지*에 대한 추상화다.

writable 스트림 예제는 다음을 포함하고 있다.

* [http requests, on the client](http.html#http_class_http_clientrequest)
* [http responses, on the server](http.html#http_class_http_serverresponse)
* [fs write streams](fs.html#fs_class_fs_writestream)
* [zlib streams][]
* [crypto streams][]
* [tcp sockets][]
* [child process stdin](child_process.html#child_process_child_stdin)
* [process.stdout][], [process.stderr][]

#### writable.write(chunk, [encoding], [callback])

* `chunk` {String | Buffer} 작성할 데이터
* `encoding` {String} `chunk`가 문자열인 경우의 인코딩
* `callback` {Function} 데이터의 청크를 내보냈을 때(flushed)의 콜백함수
* Returns: {Boolean} 데이터를 모두 처리했으면 true이다.

이 메서드는 기반 시스템에 데이터를 작성하고 데이터를 완전히 처리하고나면 전달받은
콜백함수를 호출한다.

반환값은 바로 이어서 작성해야 하는지를 나타낸다. 데이터가 내부적으로 버퍼링되었다면 `false`를
반환할 것이고 그렇지 않으면 `true`를 반환할 것이다.

이 반환값이 나타내는 경고는 엄격하다. 이 함수가 `false`를 반환했더라도 계속해서 작성이 가능할
수도 있다. 하지만 작성은 메모리에 버퍼링될 것이므로 지나치게 이렇게 하지 않는 것이 좋다.
대신 추가적인 데이터를 작성하기 전에 `drain` 에빈트를 기다려라.

#### Event: 'drain'

[`writable.write(chunk)`][] 호출이 false를 반환한 뒤 스트림에 추가적인 데이터를
시작해도 될 때를 나타내기 위해서 `drain` 이벤트가 발생한다.

```javascript
// 제공된 writable 스트림에 백만번 데이터를 작성한다.
// 역압력(back-pressure)부분을 주의깊게 봐라.
function writeOneMillionTimes(writer, data, encoding, callback) {
  var i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i -= 1;
      if (i === 0) {
        // 마지막!
        writer.write(data, encoding, callback);
      } else {
        // 계속할 지 기다릴지 확인
        // 아직 완료되지 않았으므로 callback을 전달하지 않는다.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // 일찍 마쳐야 한다!
      // drain되면 추가적인 데이터를 작성한다
      writer.once('drain', write);
    }
  }
}
```

#### writable.end([chunk], [encoding], [callback])

* `chunk` {String | Buffer} 선택적인 값으로 작성할 데이터
* `encoding` {String} `chunk`가 문자열인 경우 인코딩
* `callback` {Function} 스트림이 완료되었을 때 호출한 콜백함수로 선택적인 값이다

스트림에 작성할 데이터가 더이상 없을 때 이 함수를 호출해라. 콜백함수를 전달하면 `finish`
이벤트의 리스너로 등록된다.

[`end()`][]를 호출한 뒤에 [`write()`][]를 호출하면 오류가 발생할 것이다.

```javascript
// 'hello, '를 작성한 뒤 'world!'로 종료한다.
http.createServer(function (req, res) {
  res.write('hello, ');
  res.end('world!');
  // 더이상 데이터를 작성할 수 없다!
});
```

#### Event: 'finish'

[`end()`][] 메서드가 호출되었을 때 모든 데이터를 의존 시스템으로 내보내고 이 이벤트가 발생한다.

```javascript
var writer = getWritableStreamSomehow();
for (var i = 0; i < 100; i ++) {
  writer.write('hello, #' + i + '!\n');
}
writer.end('this is the end\n');
write.on('finish', function() {
  console.error('all writes are now complete.');
});
```

#### Event: 'pipe'

* `src` {[Readable][] Stream} 해당 writable 스트림에 파이프로 연결할 소스 스트림

이 이벤트는 readable 스트림에서 `pipe()` 메서드가 호출될 때마다 해당 writable 스트림을
목적지로 설정하면서 발생한다.

```javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('pipe', function(src) {
  console.error('something is piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
```

#### Event: 'unpipe'

* `src` {[Readable][] Stream} 해당 writable 스트림을 [unpiped][]하는 소스 스트림

이 이벤트는 readable 스트림에서 [`unpipe()`][] 메서드를 호출할 때마다 해당 writable
스트림을 목적지에서 제거하면서 발생한다.

```javascript
var writer = getWritableStreamSomehow();
var reader = getReadableStreamSomehow();
writer.on('unpipe', function(src) {
  console.error('something has stopped piping into the writer');
  assert.equal(src, reader);
});
reader.pipe(writer);
reader.unpipe(writer);
```

### Class: stream.Duplex

Duplex 스트림은 [Readable][] 인터페이스와 [Writable][] 인터페이스를 모두 구현한
스트림이다. 사용방법은 윗 부분을 봐라.

Duplex 스트림의 예제는 다음을 포함하고 있다.

* [tcp sockets][]
* [zlib streams][]
* [crypto streams][]


### Class: stream.Transform

Transform 스트림은 입력을 어떤 방법으로 계산해서 출력하는 [Duplex][] 스트림이다.
Transform은 [Readable][] 인터페이스와 [Writable][] 인터페이스를 모두 구현한다.
사용방법은 윗 부분을 봐라.

Transform 스트림 예제는 다음을 포함하고 있다.

* [zlib streams][]
* [crypto streams][]


## API for Stream Implementors

<!--type=misc-->

어떤 종류의 스트림이든지 스트림을 구현하기 위한 패턴은 동일하다.

1. 자신만의 하위클래스에 적절한 부모클래스를 상속받는다. ([`util.inherits`][]
   메서드는 이 부분에 유용하다.)
2. 생성자에서 적절한 부모 클래스의 생성자를 호출해서 내부적인 메카니즘이 적절하게
   설정되도록 해라.
2. 하나 이상의 특정 메서드를 구현해라.(자세한 내용은 아래에 나온다.)

작성하는 스트림 클래스의 종류에 따라 상속받을 클래스와 구현할 메서드가 다르다.

<table>
  <thead>
    <tr>
      <th>
        <p>사용처</p>
      </th>
      <th>
        <p>클래스</p>
      </th>
      <th>
        <p>구현할 메서드</p>
      </th>
    </tr>
  </thead>
  <tr>
    <td>
      <p>읽기 전용</p>
    </td>
    <td>
      <p>[Readable](#stream_class_stream_readable_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>쓰기 전용</p>
    </td>
    <td>
      <p>[Writable](#stream_class_stream_writable_1)</p>
    </td>
    <td>
      <p><code>[_write][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>읽기/쓰기</p>
    </td>
    <td>
      <p>[Duplex](#stream_class_stream_duplex_1)</p>
    </td>
    <td>
      <p><code>[_read][]</code>, <code>[_write][]</code></p>
    </td>
  </tr>
  <tr>
    <td>
      <p>작성한 데이터를 처리 후 결과를 읽어들인다</p>
    </td>
    <td>
      <p>[Transform](#stream_class_stream_transform_1)</p>
    </td>
    <td>
      <p><code>_transform</code>, <code>_flush</code></p>
    </td>
  </tr>
</table>

구현 코드에서는 위의 [API for Stream Consumers][]에서 설명한 메서드를 호출하지 않는
것이 아주 중요하다. 호출한다면 스트리밍 인터페이스를 사용하는 프로그렘에서 부작용이 발생할
잠재성을 갖게 된다.

### Class: stream.Readable

<!--type=class-->

`stream.Readable`은 [`_read(size)`][] 메서드의 의존 구현체를 확장도록 설계된
추상 클래스다.

프로그램에서 스트림을 사용하는 방법은 위의 [API for Stream Consumers][]를 참고해라.
이어서 프로그램에서 Readable 스트림을 어떻게 구현하는지 설명한다.

#### Example: A Counting Stream

<!--type=example-->

다음은 Readable 스트림의 기본적인 예제다. 이 예제는 순차적으로 1부터
1,000,000까지 숫자를 발생시키고 끝난다.

```javascript
var Readable = require('stream').Readable;
var util = require('util');
util.inherits(Counter, Readable);

function Counter(opt) {
  Readable.call(this, opt);
  this._max = 1000000;
  this._index = 1;
}

Counter.prototype._read = function() {
  var i = this._index++;
  if (i > this._max)
    this.push(null);
  else {
    var str = '' + i;
    var buf = new Buffer(str, 'ascii');
    this.push(buf);
  }
};
```

#### Example: SimpleProtocol v1 (Sub-optimal)

이는 위에서 설명한 `parseHeader` 함수와 비슷하지만 커스텀 스트림으로 구현되었다.
그리고 이 구현체는 들어오는 데이터를 문자열로 변환하지 않는다.

하지만 이는 [Transform][] 스트림으로 구현하는 것이 더 좋다.
더 좋은 구현체는 아래 부분을 참고해라.

```javascript
// 간단한 데이터 프로토콜에 대한 파서.
// "header"는 JSON 객체이고 이어서 두개의 \n 문자가 오로고 이어서 메시지 바디가 온다.
//
// Note: 이는 Transform 스트림으로 훨씬 간단히 할 수 있다!
// 이를 위해 Readable를 직접 사용하는 것이 차선책이다. 이에 대한 예제은 아래의
// Transform 부분을 참고해라.

var Readable = require('stream').Readable;
var util = require('util');

util.inherits(SimpleProtocol, Readable);

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
// var parser = new SimpleProtocol(source);
// 이제 파서는 파싱된 헤더 데이터를 가진 'header'를 발생시킬
// readable 스트림이다.
```


#### new stream.Readable([options])

* `options` {Object}
  * `highWaterMark` {Number} 사용하는 리소스에서 읽기를 중단하기 전에
    내부 버퍼에 저장할 최대 바이트 수. 기본값=16kb
  * `encoding` {String} 이 값을 지정하면 지정한 인코딩으로 버퍼를 문자열로
    디코드한다. 기본값=null
  * `objectMode` {Boolean} 이 스트림이 객체의 스트림처럼 동작해야 하는지 여부.
    즉, stream.read(n)이 n 크기의 Buffer 대신 문자열 값을 반환한다는 것을 의미한다.

Readable 클래스를 확장하는 클래스에서 버퍼링 설정을 적절하게 초기화할 수 있도록
Readable 생성자를 꼭 호출해라.

#### readable.\_read(size)

* `size` {Number} 비동기적으로 읽을 바이트 수

Note: **이 함수를 구현하되 직접 호출하지는 마라.**

이 함수는 직접 호출하지 말아야 한다. 이 함수는 자식 클래스에서 구현해야 하고
Readable 클래스 내부 함수에서만 호출해야 한다.

모든 Readable 스트림 구현체는 사용하는 리소스에서 데이터를 가져오는 `_read` 메서드를
반드시 제공해야 한다.

이 함수는 클래스는 내부에서만 사용할 목적이므로 언더스코어 접두사가 붙어있고 사용자
프로그램에서 직접 호출하지 않아야 한다. 하지만 자신만의 확장 클래스에서 이 메서드를
오버라이드하기 **원할 수도 있다.**

데이터를 사용할 수 있을 때 `readable.push(chunk)`를 호출해서 읽기 큐에 데이터를 넣는다.
`push`가 false를 반환하면 읽기를 멈춰야 한다. `_read`가 다시 호출됐을 때 추가적인
데이터를 큐에 넣기 시작해야 한다.

`size` 아규먼트는 권고사항이다. "read"는 데이터를 반환하는 하나의 호출인
구현체에서 얼마나 많은 데이터를 가여와야 하는지 지정하는데 이 값을 사용한다. TCP나 TLS처럼
관련이 없는 구현체는 이 인자를 무시하고 데이터를 이용할 수 있을때마다 제공할 것이다. 예를 들면
[`stream.push(chunk)`][]를 호출하기 전에 `size` 바이트가 사용가능할 때까지
"기다릴" 필요가 없다.

#### readable.push(chunk, [encoding])

* `chunk` {Buffer | null | String} 읽기 큐에 데이터의 청크를 넣는다
* `encoding` {String} 문자열 청크의 인코딩. `'utf8'`나 `'ascii'`같은
  유효한 Buffer 인코딩이어야 한다.
* return {Boolean} push를 계속해야 하는지 여부

Note: **이 함수는 Readable 구현체에서 호출해야 지 Readable 스트림를 소비하는 쪽에서
호출하면 안된다.**

`push(chunk)`가 최소 한번 실행될 때까지는 `_read()` 함수는 다시 호출되지
않을 것이다.

나중에 `'readable'` 이벤트가 발생했을 때 `read()` 메서드를 호출해서 데이터를 꺼내도록
`Readable` 클래스는 읽기 큐에 데이터를 넣어서 동작한다.

`push()` 메서드는 데이터를 읽기 큐에 명시적으로 넣을 것이다. 이 함수를 `null`로 호출하면
데이터가 끝났다는(EOF) 신호가 올 것이다.

이 API는 최대한 유연하게 설계되었다. 예를 들어 어떤 종류의 pause/resume 메카니즘과 데이터
콜백을 가진 저수준 소스를 감싸고 있다고 해보자. 이러한 경우 다음과 같이 해서 저수준 소스를
감쌀 수 있다.

```javascript
// source는 readStop(), readStart() 메서드와
// source가 데이터를 가졌을 때 호출되는 `ondata` 멤버,
// 데이터가 끝났을 때 호출되는 `onend` 멤버를 가진 객체이다.

util.inherits(SourceWrapper, Readable);

function SourceWrapper(options) {
  Readable.call(this, options);

  this._source = getLowlevelSourceObject();
  var self = this;

  // 데이터가 있을 때마다 내부 버퍼에 데이터를 넣는다.
  this._source.ondata = function(chunk) {
    // push()가 false를 반환하면 source에서 읽기를 멈춰야 한다
    if (!self.push(chunk))
      self._source.readStop();
  };

  // source가 끝났을 때 EOF를 알리는 `null` 청크를 넣는다.
  this._source.onend = function() {
    self.push(null);
  };
}

// 이 경우에 권고사항인 size 인자를 무시하고 스트림이 추가적인 데이터를
// 가져오도록 하고 싶을 때 _read를 호출할 것이다.
SourceWrapper.prototype._read = function(size) {
  this._source.readStart();
};
```


### Class: stream.Writable

<!--type=class-->

`stream.Writable`는 [`_write(chunk, encoding, callback)`][] 메서드의
의존 구현체로 확장하도록 설계된 추상 클래스다.

프로그램에서 writable 스트림을 사용하는 방법은 위의 [API for Stream Consumers][]
부분을 참고해라. 여기서는 프로그램에서 Writable 스트림을 구현하는 방법을 설명한다.

#### new stream.Writable([options])

* `options` {Object}
  * `highWaterMark` {Number} [`write()`][]가 false를 반환하기 시작했을 때
    Buffer 레벨. 기본값=16kb
  * `decodeStrings` {Boolean} 문자열을 [`_write()`][]에 전달하기 전에
    Buffer로 디코딩할 것인지 여부.  기본값=true

Writable 클래스를 확장한 클래스에서는 버퍼링 설정이 적절하게 초기화될 수 있도록
꼭 생성자를 호출해라.

#### writable.\_write(chunk, encoding, callback)

* `chunk` {Buffer | String} 작성할 청크. `decodeStrings` 옵션이 `false`로
  설정하지 않았다면 항상 버퍼가 될 것이다.
* `encoding` {String} 청크가 문자열인경우 인코딩 종류. 청크가 버퍼이면 무시한다.
  `decodeStrings` 옵션을 명시적으로 `false`로 설정하지 않았다면 청크는
  **항상** 버퍼가 될 것이다.
* `callback` {Function} 전달된 청크를 모두 처리했을 때
  이 함수(선택적으로 오류 인자와 함께)를 호출한다.

모든 Writable 스트림 구현체는 기반 리소스에 데이터를 전송하기 위해서
[`_write()`][] 메서드를 제공해야 한다.

Note: **이 함수는 직접 호출하지 말아야 한다.** 이는 자식 클래스에서 구현되어야 하고
Writable 클래스 내부 메서드만 호출해야 한다.

작성이 성공적으로 완료되었거나 오류가 발생했다는 신호로 표준 `callback(error)`
패턴을 사용해서 콜백을 호출한다.

생성자 옵션에서 `decodeStrings` 플래그를 설정했다면 `chunk`는 버퍼가 아니라
문자열이 되고 `encoding`은 문자열의 종류를 나타내게 된다. 이는 특정 문자열 데이터
인코딩에 대한 최적화 처리를 하는 구현체를 지원하기 위함이다. 명시적으로 `decodeStrings`
옵션을 `false`로 설정하지 않았다면 `encoding` 인자를 무시하고 `chunk`가 항상
Buffer라고 생각할 수 있다.

이 메서드는 정의된 클래스 내부용이므로 언더스코어 접두사가 붙어있다. 그래서 사용자
프로그램이 직접 호출하지 말아야 한다. 하지만 자신의 확장 클래스에서 이 함수를
오버라이드하기 **원할 수도 있다**.


### Class: stream.Duplex

<!--type=class-->

"duplex" 스트림은 TCP 소켓 연결처럼 Readable이면서 Writable인 스트림이다.

`stream.Duplex`는 Readable 스트림 클래스나 Writable 스트림 클래스에서 했듯이
`_read(size)`와 [`_write(chunk, encoding, callback)`][] 메서드의
의존 구현체로 확장하도록 설계된 추상 클래스다.

자바스크립트가 프로토타입 다중 상속을 지원하지 않으므로 이 클래스는 Readable을
프로토타입으로 상속받은 뒤 Writable이 기생하는 식으로 동작한다. 그러므로 확장 duplex
클래스에서 저수준 [`_write(chunk, encoding, callback)`][] 메서드와
저수준 `_read(n)`를 사용자가 모두 구현했는가에 달려있다.

#### new stream.Duplex(options)

* `options` {Object} Writable 생성자와 Readable 생성자에 전달되는 옵션.
  다음의 값도 가진다.
  * `allowHalfOpen` {Boolean} 기본값=true.  `false`로 설정했다면 스트림은
    writable 쪽이 종료되었을 때 readable쪽도 자동으로 종료하고 그 반대의 경우도
    마찬가지다.

Duplex 클래스를 확장한 클래스에서는 버퍼링 설정을 적절히 초기화할 수 있도록
생성자를 꼭 호출해야 한다.


### Class: stream.Transform

"transform" 스트림은 [zlib][] 스트림이나 [crypto][] 스트림처럼 출력이
어떤 식으로든 입력에 연결되는 duplex 스트림이다.

출력이 입력과 같은 크기이거나 청크의 수가 같거나 동시에 도착해야 한다는 등의 요구사항은 없다.
예를 들면 Hash 스트림은 입력이 종료되었을 때 제공된 출력의 단일 청크만을 가질 것이다.
zlib 스트림은 입력코다 아주 작거나 아주 큰 출력을 보낼 것이다.

Transform 클래스는 [`_read()`][]와 [`_write()`][] 메서드를 구현하기 보다는
`_transform()` 메서드를 구현해야 하고 선택적으로 `_flush()` 메서드를
구현할 수도 있다.(하단 참조)

#### new stream.Transform([options])

* `options` {Object} Writable과 Readable 생성자에 모두 전달된다.

Transform 클래스를 확장한 클래스에서는 버퍼링 설정을 적절하게 초기화할 수 있도록
생성자를 호출해야 한다.

#### transform.\_transform(chunk, encoding, callback)

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

#### transform.\_flush(callback)

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

#### Example: `SimpleProtocol` parser v2

위의 간단한 프로토콜 파서 예제를 고수준의 [Transform][] 스트림 클래스를 사용해서
간단하게 구현할 수 있고 이는 위의 `parseHeader`와 `SimpleProtocol v1` 예제와
비슷하다.

이 예제에서는 인자로 입력을 제공하기 보다는 더 이상적인 Node 스트림 접근으로
파서에 파이프를 연결한다.

```javascript
var util = require('util');
var Transform = require('stream').Transform;
util.inherits(SimpleProtocol, Transform);

function SimpleProtocol(options) {
  if (!(this instanceof SimpleProtocol))
    return new SimpleProtocol(options);

  Transform.call(this, options);
  this._inBody = false;
  this._sawFirstCr = false;
  this._rawHeader = [];
  this.header = null;
}

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
      // 헤더 파싱이 끝났음을 알려준다.
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

// 사용방법:
// var parser = new SimpleProtocol();
// source.pipe(parser)
// 이제 파서는 파싱된 헤더 데이터를 가진 'header'를 발생시킬
// readable 스트림이다.
```


### Class: stream.PassThrough

이는 입력 바이트를 출력으로 단순히 전달하는 [Transform][] 스트림의 별로 중요치 않은 구현체이다.
이 클래스의 목적은 주로 예제나 테스트이지만 새로운 스트림에 대한 블럭을 구성하는 경우처럼
가끔은 유용한 경우가 있다.


## Streams: Under the Hood

<!--type=misc-->

### Buffering

<!--type=misc-->

Writable 스트림과 Readable 스트림은 `_writableState.buffer`나
`_readableState.buffer`를 각각 호출해서 내부 객체에 데이터를 버퍼링할 것이다.

버퍼될 데이터의 양은 생성자에 전달한 `highWaterMark` 옵션에 따라 다르다.

Readable 스트림에서는 구현체가 [`stream.push(chunk)`][]를 호출했을 때 버퍼링이 일어난다.
스트림의 컨슈머가 `stream.read()`를 호출하지 않은 경우 데이터가 소비될 때까지 데이터는
내부 큐에 있을 것이다.

Writable 스트림에서는 사용자가 [`stream.write(chunk)`][]를 반복적으로
호출했을 때(`write()`가 `false`를 반환하더라도) 버퍼링이 일어난다.

스트림의 목적은(특히 `pipe()` 메서드) 데이터 버퍼링을 수긍할만한 수준으로 제한해서 속도가
제각각인 소스와 목적지가 사용가능한 메모리를 넘어서지 않게 하기 위함이다.

### `stream.read(0)`

데이터는 실제로 소비하지 않으면서 사용하는 readable 스트림의 메카니즘을 갱신하고자 하는
경우가 있다. 이러한 경우에는 항상 null을 반환하는 `stream.read(0)`를 호출할 수 있다.

내부 읽기 버퍼가 `highWaterMark`보다 작고 스트림이 읽고 있는 중이 아닌 경우 `read(0)`를
호출하면 저수준의 `_read` 호출이 발생할 것이다.

이렇게 해야하는 경우는 거의 없지만 Node 내부에서 이러한 작업을 하는 경우를
볼 수 있을 것이다.(특히 Readable 스트림 클래스의 내부에서)

### `stream.push('')`

0 바이트 문자열이나 버퍼를 밀어넣을 때([Object mode][]가 아닌 경우) 흥미로운 부가작용이
일어난다. 이는 [`stream.push()`][]를 호출한 *것이므로* `reading` 과정을 종료할 것이다.
하지만 읽기 가능한 버퍼에는 어떤 데이터도 추가하지 *않으므로* 유저가 사용할 수 있는 데이터는
전혀 없다.

아주 드물게 지금은 제공할 데이터가 없지만 스트림의 컨슈머(또는 자신의 코드의 다른 곳에서)는
`stream.read(0)`를 호출해서 언제 다시 확인해 봐야 하는지 알아야 하는 경우가 있다.
이러한 경우 `stream.push('')`를 호출할 *것이다*.

지금까지는 이 기능을 유일하게 사용한 곳이 Node v0.12에서는 폐기된 [tls.CryptoStream][]
클래스다. `stream.push('')`를 사용해야 한다면 무언가 크게 잘못될 가능성이 아주 크므로
다른 접근이 없는지 생각해 보기를 바란다.

### Compatibility with Older Node Versions

<!--type=misc-->

Node v0.10 이전 버전에서는 Readable 스트림 인터페이스가 더 간단하지만 동시에
덜 강력하고 덜 유용하다.

* `read()` 메서드를 호출하기를 기다리기 보다는 `'data'` 이벤트가 바로 발생하기 시작할
  것이다. 데이터를 다루는 방법을 결정하기 위해서 I/O를 사용해야 한다면 데이터를 잃어버리지
  않기 위해 어떤 종류든 버퍼에 청크를 저장해야 한다.
* `pause()` 메서드는 보장된다기 보다는 권고사항이다. 즉, 스트림이 멈춰진 상태에 있더라도
  `'data'` 이벤트를 받을 준비를 하고 있어야 한다.

Node v0.10에는 아래에서 설명한 Readable 클래스가 추가되었다. 오래된 Node 프로그램과의
하위 호환성때문에 `'data'` 이벤트 핸들러가 추가되거나 `pause()` 혹은 `resume()`
메서드가 호출되었을 때 Readable 스트림은 "flowing mode"로 바뀐다. 그래서 새로운
`read()` 메서드와 `'readable'` 이벤트를 사용하지 않더라도 `'data'` 청크를
잃어버릴 걱정을 하지 않아도 된다.

대부분의 프로그램은 정상적으로 잘 동작할 것이지만 이 기능은 다음과 같은 상황에
대처한다.

* `'data'` 이벤트 핸들러가 추가되지 않았을 때.
* `pause()`와 `resume()` 메서드가 호출되지 않았을 때.

에를 들면 다음 코드를 생각해 보자.

```javascript
// 주의! 동작하지 않음!
net.createServer(function(socket) {

  // 'end' 메서드를 추가했지만 데이터를 소비하지 않는다
  socket.on('end', function() {
    // 이 코드는 실행되지 않는다.
    socket.end('I got your message (but didnt read it)\n');
  });

}).listen(1337);
```

Node v0.10 이전 버전에서 들어오는 메시지 데이터는 그냥 버려진다.
하지만 Node v0.10 이상에서는 소켓이 영원히 멈춰진 상태로 남게 된다.

이 상황을 해결하려면 "old mode"로 동작하도록 `resume()`을 호출해야 한다.

```javascript
// 해결 방법
net.createServer(function(socket) {

  socket.on('end', function() {
    socket.end('I got your message (but didnt read it)\n');
  });

  // 데이터의 흐름이 시작되고 데이터를 버린다.
  socket.resume();

}).listen(1337);
```

flowing-mode로 변경되는 새로운 Readable 스트림에 추가적으로 v0.10 이전 방식의 스트림은
`wrap()` 메서드를 사용해서 Readable 클래스로 감쌀 수 있다.


### Object Mode

<!--type=misc-->

일반적으로 스트림은 문자열이나 버퍼에서만 동작한다.

**object 모드**의 스트림은 버퍼나 문자열이 아닌 일반적인 자바스크립트 값을
발생시킬 수 있다.

object 모드의 Readable 스트림은 size 인자의 크기에 상관없이
`stream.read(size)` 호출에서 항상 하나의 아이템을 반환할 것이다.

object 모드의 Writable 스트림은 `stream.write(data, encoding)`의
`encoding` 인자를 항상 무시할 것이다.

특별한 값인 `null`은 object 모드 스트림에서도 여전히 특별한 값이다. 즉, object 모드의
readable 스트림에서 `stream.read()`이 반환한 `null`은 더이상 데이터가 없음을
의미하고 [`stream.push(null)`][]은 스트림 데이터가 끝났다(`EOF`)는 신호로 사용한다.

Node 코어에서 object 모드인 스트림은 없다.
이 방식은 사용자의 스트리밍 라이브러리에서만 사용한다.

스트 자식 클래스 생성자에서 옵션 객체에 `objectMode`를 설정해야 했다.
중간 스트림에 `objectMode`를 설정하는 것은 안전하지 않다.

### State Objects

[Readable][] 스트림은 `_readableState`라는 멤버 객체를 가진다.
[Writable][] 스트림은 `_writableState`라는 멤버 객체를 가진다.
[Duplex][] 스트림은 위의 두 멤버 객체를 모두 가진다.

**이러한 객체는 일반적으로 자식 클래스에서 수정하면 안된다.**
하지만 읽는 쪽이 `objectMode`이고 쓰는 쪽은 `objectMode`가 아닌 Duplex나
Transform 스트림이라면 적절한 상태 객체에 명시적으로 플래그를 설정해서 생성자에서
수정해야 한다.

```javascript
var util = require('util');
var StringDecoder = require('string_decoder').StringDecoder;
var Transform = require('stream').Transform;
util.inherits(JSONParseStream, Transform);

// \n으로 구분된 JSON 문자열 데이터를 가져오고 파싱된 객체를 보낸다.
function JSONParseStream(options) {
  if (!(this instanceof JSONParseStream))
    return new JSONParseStream(options);

  Transform.call(this, options);
  this._writableState.objectMode = false;
  this._readableState.objectMode = true;
  this._buffer = '';
  this._decoder = new StringDecoder('utf8');
}

JSONParseStream.prototype._transform = function(chunk, encoding, cb) {
  this._buffer += this._decoder.write(chunk);
  // newline으로 분리한다
  var lines = this._buffer.split(/\r?\n/);
  // 버퍼링된 마지막 부분의 라인을 유지한다
  this._buffer = lines.pop();
  for (var l = 0; l < lines.length; l++) {
    var line = lines[l];
    try {
      var obj = JSON.parse(line);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // 읽을 수 있는 컨슈머로 파싱된 객체를 밀어넣는다
    this.push(obj);
  }
  cb();
};

JSONParseStream.prototype._flush = function(cb) {
  // 남은 부분을 처리한다
  var rem = this._buffer.trim();
  if (rem) {
    try {
      var obj = JSON.parse(rem);
    } catch (er) {
      this.emit('error', er);
      return;
    }
    // 읽을 수 있는 컨슈머로 파싱된 객체를 밀어넣는다
    this.push(obj);
  }
  cb();
};
```

상태 객체에는 프로그램에서 스트림의 상태를 디버깅할 수 있는 유용한 정보가 담겨있다. 이 객체를
살펴보는 건 안전하지만 생성자에서 옵션 플래스를 설정하는 것을 넘어서서 객체를 수정하는 것은
안전하지 **않다**.


[EventEmitter]: events.html#events_class_events_eventemitter
[Object mode]: #stream_object_mode
[`stream.push(chunk)`]: #stream_readable_push_chunk_encoding
[`stream.push(null)`]: #stream_readable_push_chunk_encoding
[`stream.push()`]: #stream_readable_push_chunk_encoding
[`unpipe()`]: #stream_readable_unpipe_destination
[unpiped]: #stream_readable_unpipe_destination
[tcp sockets]: net.html#net_class_net_socket
[zlib streams]: zlib.html
[zlib]: zlib.html
[crypto streams]: crypto.html
[crypto]: crypto.html
[tls.CryptoStream]: tls.html#tls_class_cryptostream
[process.stdin]: process.html#process_process_stdin
[stdout]: process.html#process_process_stdout
[process.stdout]: process.html#process_process_stdout
[process.stderr]: process.html#process_process_stderr
[child process stdout and stderr]: child_process.html#child_process_child_stdout
[API for Stream Consumers]: #stream_api_for_stream_consumers
[API for Stream Implementors]: #stream_api_for_stream_implementors
[Readable]: #stream_class_stream_readable
[Writable]: #stream_class_stream_writable
[Duplex]: #stream_class_stream_duplex
[Transform]: #stream_class_stream_transform
[`_read(size)`]: #stream_readable_read_size_1
[`_read()`]: #stream_readable_read_size_1
[_read]: #stream_readable_read_size_1
[`writable.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback
[`write()`]: #stream_writable_write_chunk_encoding_callback
[`stream.write(chunk)`]: #stream_writable_write_chunk_encoding_callback
[`_write(chunk, encoding, callback)`]: #stream_writable_write_chunk_encoding_callback_1
[`_write()`]: #stream_writable_write_chunk_encoding_callback_1
[_write]: #stream_writable_write_chunk_encoding_callback_1
[`util.inherits`]: util.html#util_util_inherits_constructor_superconstructor
[`end()`]: #stream_writable_end_chunk_encoding_callback
