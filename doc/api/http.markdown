# HTTP

    Stability: 3 - Stable

To use the HTTP server and client one must `require('http')`.

The HTTP interfaces in Node are designed to support many features
of the protocol which have been traditionally difficult to use.
In particular, large, possibly chunk-encoded, messages. The interface is
careful to never buffer entire requests or responses--the
user is able to stream data.

HTTP message headers are represented by an object like this:

    { 'content-length': '123',
      'content-type': 'text/plain',
      'connection': 'keep-alive',
      'accept': '*/*' }

Keys are lowercased. Values are not modified.

In order to support the full spectrum of possible HTTP applications, Node's
HTTP API is very low-level. It deals with stream handling and message
parsing only. It parses a message into headers and body but it does not
parse the actual headers or the body.

    안정성: 3 - Stable

HTTP 서버와 클라이언트를 사용하려면 `require('http')`를 사용해라.

Node의 HTTP 인터페이스는 전통적으로 다루기 어려웠던 프로토콜의 많은 기능들을 
지원하려고 디자인되었다. 특히 크고 청크로 인코딩 될수 있는 메시지들이다.
인터페이스는 전체 요청이나 응답을 버퍼에 넣지 않는다. 사용자는 스트림 데이터를
버퍼에 넣을 수 있다.

HTTP 메시지 헤더는 다음과 같은 객체로 표현된다.

    { 'content-length': '123',
      'content-type': 'text/plain',
      'connection': 'keep-alive',
      'accept': '*/*' }

키는 소문자로 쓰고 값은 수정되지 않는다.

HTTP 어플리케이션이 가능한 전체 범위를 다 지원하기 위해서 Node의 HTTP API는 상당히 
저수준의 API이다. API는 스트림 핸들링과 메시지 파싱만을 다룬다. 메시지를 헤더와 바디로 
파싱하지만 실제 헤더와 바디는 파싱하지 않는다.


## http.createServer([requestListener])

Returns a new web server object.

The `requestListener` is a function which is automatically
added to the `'request'` event.

새로운 웹서버 객체를 반환한다.

`requestListener`는 자동으로 `'request'` 이벤트에 추가되는 
함수다.

## Class: http.Server

This is an `EventEmitter` with the following events:

이 클래스는 다음의 이벤트를 가진 `EventEmitter`다.

### Event: 'request'

`function (request, response) { }`

Emitted each time there is a request. Note that there may be multiple requests
per connection (in the case of keep-alive connections).
 `request` is an instance of `http.ServerRequest` and `response` is
 an instance of `http.ServerResponse`

`function (request, response) { }`

요청이 있을 때마다 발생한다. 연결마다 여러번의 요청이 있을 수 있다.(keep-alive 
연결인 경우) 
 `request`는 `http.ServerRequest`의 인스턴스이고 `response`는 
 `http.ServerResponse`의 인스턴스다.

### Event: 'connection'

`function (socket) { }`

 When a new TCP stream is established. `socket` is an object of type
 `net.Socket`. Usually users will not want to access this event. The
 `socket` can also be accessed at `request.connection`.

`function (socket) { }`

 새로운 TCP 스트림이 생성되었을 때 발생한다. `socket`은 `net.Socket` 타입의 
 객체다. 보통 사용자들은 이 이벤트에 접근하지 않을 것이다. `socket`은 
 `request.connection`에서도 접근할 수 있다.

### Event: 'close'

`function () { }`

 Emitted when the server closes.

`function () { }`

 서버가 닫혔을 때 발생한다.

### Event: 'checkContinue'

`function (request, response) { }`

Emitted each time a request with an http Expect: 100-continue is received.
If this event isn't listened for, the server will automatically respond
with a 100 Continue as appropriate.

Handling this event involves calling `response.writeContinue` if the client
should continue to send the request body, or generating an appropriate HTTP
response (e.g., 400 Bad Request) if the client should not continue to send the
request body.

Note that when this event is emitted and handled, the `request` event will
not be emitted.

`function (request, response) { }`

http Expect: 100-continue 헤더를 가진 요청을 받을 때마다 발생한다.
이 이벤트가 바인딩되지 않았다면 서버는 자동적으로 적절한 100 Continue로 응답할 
것이다.

이 이벤트를 다루면 클라이언트가 계속해서 요청바디를 보내야 한다면 
`response.writeContinue` 호출하고 클라이언트가 요청 바디를 계속 보내지 않는다면 
적절한 HTTP 응답(예시: 400 Bad Request)을 생성한다.

### Event: 'upgrade'

`function (request, socket, head) { }`

Emitted each time a client requests a http upgrade. If this event isn't
listened for, then clients requesting an upgrade will have their connections
closed.

* `request` is the arguments for the http request, as it is in the request event.
* `socket` is the network socket between the server and client.
* `head` is an instance of Buffer, the first packet of the upgraded stream, this may be empty.

After this event is emitted, the request's socket will not have a `data`
event listener, meaning you will need to bind to it in order to handle data
sent to the server on that socket.

`function (request, socket, head) { }`

클라이언트가 http 업그래이드를 요청할 때마다 발생한다. 이 이벤트가 바인딩되지 않았다면 
업그래이드를 요청하는 클라이언트는 닫힌 연결을 가질 것이다.

* `request`은 요청이벤트에 있는 것처럼 http 요청의 아규먼트다. 
* `socket`은 서버와 클라이언트 사이의 네트워크 소켓이다.
* `head`는 Buffer의 인스턴스다. 업그래이드된 스트림의 첫 패킷이고 비어있을 수도 있다.

이 이벤트가 발생한 후에 요청의 소켓은 `data` 이벤트 리스너를 갖지 않을 것이다. 이는 
해당 소켓으로 서버에 보내는 메시지를 다루기 위해서는 소켓에 바인딩할 필요가 있다는 의미이다.

`function (request, socket, head) { }`

### Event: 'clientError'

`function (exception) { }`

If a client connection emits an 'error' event - it will forwarded here.

`function (exception) { }`

클라이언트 연결에서 'error' 이벤트가 발생하면 이 이벤트가 진행된다.

### server.listen(port, [hostname], [callback])

Begin accepting connections on the specified port and hostname.  If the
hostname is omitted, the server will accept connections directed to any
IPv4 address (`INADDR_ANY`).

To listen to a unix socket, supply a filename instead of port and hostname.

This function is asynchronous. The last parameter `callback` will be added as
a listener for the ['listening'](net.html#event_listening_) event.
See also [net.Server.listen()](net.html#server.listen).

지정한 hostname과 port에서 열결을 받아들이기 시작한다. hostname을 생락하면 서버는 
IPv4 주소(`INADDR_ANY`)에서 들어오는 연결을 모두 받아들일 것이다.

유닉스 소켓에 바인딩하려면 port와 hostname 대신에 파일명을 전달한다.

이 함수는 비동기 함수다. 마지막 파라미터 `callback`은 
['listening'](net.html#event_listening_) 이벤트의 리스터로 추가될 것이다.
[net.Server.listen()](net.html#server.listen)도 참고해 봐라.


### server.listen(path, [callback])

Start a UNIX socket server listening for connections on the given `path`.

This function is asynchronous. The last parameter `callback` will be added as
a listener for the ['listening'](net.html#event_listening_) event.
See also [net.Server.listen()](net.html#server.listen).

전달한 `path`에서 연결을 받아들이는 UNIX 소켓 서버를 시작한다.

이 함수는 비동기 함수다. 마지막 파라미터 `callback`은 
['listening'](net.html#event_listening_) 이벤트의 리스너로 추가될 것이다.
[net.Server.listen()](net.html#server.listen)도 참고해라.


### server.close()

Stops the server from accepting new connections.
See [net.Server.close()](net.html#server.close).

서버가 새로운 연결을 받아들이는 것을 멈춘다.
[net.Server.close()](net.html#server.close)를 봐라.


## Class: http.ServerRequest

This object is created internally by a HTTP server -- not by
the user -- and passed as the first argument to a `'request'` listener.

The request implements the [Readable Stream](stream.html#readable_stream)
interface. This is an `EventEmitter` with the following events:

사용자가 아니라 HTTP 서버 내부적으로 생성되는 객체다. 
`'request'` 리스너의 첫번째 아규먼트로 전달한다.

### Event: 'data'

`function (chunk) { }`

Emitted when a piece of the message body is received. The chunk is a string if
an encoding has been set with `request.setEncoding()`, otherwise it's a
[Buffer](buffer.html).

Note that the __data will be lost__ if there is no listener when a
`ServerRequest` emits a `'data'` event.

`function (chunk) { }`

메시지 바디의 일부를 받았을 때 발생한다. `request.setEncoding()`로 인코딩을 
설정한 경우 청크는 문자열이고 설정하지 않았으면 [Buffer](buffer.html)다.

`ServerRequest`가 `'data'` 이벤트를 했을 때 리스터가 없다면 
__데이터를 읽어버릴 것이다.__

### Event: 'end'

`function () { }`

Emitted exactly once for each request. After that, no more `'data'` events
will be emitted on the request.

`function () { }`

각 요청마다 정확히 한번씩만 발생한다. 이 이벤트 후에는 해당 요청에서 `'data'` 이벤트가 
더이상 발생하지 않을 것이다.

### Event: 'close'

`function () { }`

Indicates that the underlaying connection was terminated before
`response.end()` was called or able to flush.

Just like `'end'`, this event occurs only once per request, and no more `'data'`
events will fire afterwards.

Note: `'close'` can fire after `'end'`, but not vice versa.

`function () { }`

`response.end()`가 호출되기 전이나 플러시할 수 있을 때 의존하는 연결을 
종료했다는 것을 나타낸다. 

`'end'`처럼 이 이벤트는 요청당 딱 한번만 발생하고 그 후에는 더 이상 
`'data'` 이벤트가 발생하지 않을 것이다.

Note: `'end'`후에 `'close'`가 발생할 수 있지만 그 반대로는 안된다.

### request.method

The request method as a string. Read only. Example:
`'GET'`, `'DELETE'`.

요청 메서드의 문자열 표현. 읽기 전용이다. 예시: `'GET'`, `'DELETE'`


### request.url

Request URL string. This contains only the URL that is
present in the actual HTTP request. If the request is:

    GET /status?name=ryan HTTP/1.1\r\n
    Accept: text/plain\r\n
    \r\n

Then `request.url` will be:

    '/status?name=ryan'

If you would like to parse the URL into its parts, you can use
`require('url').parse(request.url)`.  Example:

    node> require('url').parse('/status?name=ryan')
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: 'name=ryan',
      pathname: '/status' }

If you would like to extract the params from the query string,
you can use the `require('querystring').parse` function, or pass
`true` as the second argument to `require('url').parse`.  Example:

    node> require('url').parse('/status?name=ryan', true)
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: { name: 'ryan' },
      pathname: '/status' }

요청 URL 문자열. 이 값은 실제 HTTP 요청에 있는 URL만 담고 있다. 
요청이 다음과 같다면,

    GET /status?name=ryan HTTP/1.1\r\n
    Accept: text/plain\r\n
    \r\n

`request.url`는 다음의 값이 될 것이다.

    '/status?name=ryan'

URL을 부분별로 파싱하고 싶다면 `require('url').parse(request.url)`를 
사용할 수 있다. 예제:

    node> require('url').parse('/status?name=ryan')
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: 'name=ryan',
      pathname: '/status' }

쿼리스트링에서 파라미터를 추출하려면 `require('querystring').parse` 
함수를 사용하거나 `require('url').parse`의 두번째 아규먼트로 
`true`를 전달할 수 있다. 예제:

    node> require('url').parse('/status?name=ryan', true)
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: { name: 'ryan' },
      pathname: '/status' }



### request.headers

Read only.

읽기 전용.

### request.trailers

Read only; HTTP trailers (if present). Only populated after the 'end' event.

읽기 전용. (존재하는 경우) HTTP trailers이다. 'end' 이벤트 후에만 존재한다.

### request.httpVersion

The HTTP protocol version as a string. Read only. Examples:
`'1.1'`, `'1.0'`.
Also `request.httpVersionMajor` is the first integer and
`request.httpVersionMinor` is the second.

HTTP 프로토콜 버전의 문자열 표현이다. 읽기 전용. 
예시: `'1.1'`, `'1.0'` 


### request.setEncoding([encoding])

Set the encoding for the request body. Either `'utf8'` or `'binary'`. Defaults
to `null`, which means that the `'data'` event will emit a `Buffer` object..

요청 바디의 인코딩을 설정하고 `'utf8'`나 `'binary'`가 가능하다. 기본값은 `'data'` 
이벤트가 `Buffer` 객체를 발생시킨다는 것을 나타내는 `null`이다.


### request.pause()

Pauses request from emitting events.  Useful to throttle back an upload.

요청이 이벤트를 발생키시는 것을 멈춘다. 흐름을 제어하는 데 유용하다.


### request.resume()

Resumes a paused request.

멈췄던 요청을 복구한다.

### request.connection

The `net.Socket` object associated with the connection.


With HTTPS support, use request.connection.verifyPeer() and
request.connection.getPeerCertificate() to obtain the client's
authentication details.

요청과 연결된 `net.Socket` 객체다.


HTTPS 지원과 함께 클라이언트의 인증 세부사항을 얻으려면 
request.connection.verifyPeer()와 
request.connection.getPeerCertificate()를 사용해라. 



## Class: http.ServerResponse

This object is created internally by a HTTP server--not by the user. It is
passed as the second parameter to the `'request'` event.

The response implements the [Writable  Stream](stream.html#writable_stream)
interface. This is an `EventEmitter` with the following events:

사용자가 아니라 HTTP 서버가 내부적으로 생성하는 객체다. `'request'` 이벤트의 
두번째 파라미터로 전달된다.

응답은 [Writable  Stream](stream.html#writable_stream) 인터페이스를 구현했다.
이 클래스는 다음의 이벤트를 가지고 있는 `EventEmitter`이다.

### Event: 'close'

`function () { }`

Indicates that the underlaying connection was terminated before
`response.end()` was called or able to flush.

`function () { }`

`response.end()`가 호출되기 전이나 플러시할 수 있을 때 의존하는 연결을 
종료했다는 것을 나타낸다. 

### response.writeContinue()

Sends a HTTP/1.1 100 Continue message to the client, indicating that
the request body should be sent. See the [checkContinue](#event_checkContinue_) event on
`Server`.

클라이언트에 요청 바디가 보내질 것이라는 것을 나타내는 HTTP/1.1 100 Continue 
메시지를 보낸다. `Server`의 [checkContinue](#event_checkContinue_) 이벤트를 봐라.


### response.writeHead(statusCode, [reasonPhrase], [headers])

Sends a response header to the request. The status code is a 3-digit HTTP
status code, like `404`. The last argument, `headers`, are the response headers.
Optionally one can give a human-readable `reasonPhrase` as the second
argument.

Example:

    var body = 'hello world';
    response.writeHead(200, {
      'Content-Length': body.length,
      'Content-Type': 'text/plain' });

This method must only be called once on a message and it must
be called before `response.end()` is called.

If you call `response.write()` or `response.end()` before calling this, the
implicit/mutable headers will be calculated and call this function for you.

Note: that Content-Length is given in bytes not characters. The above example
works because the string `'hello world'` contains only single byte characters.
If the body contains higher coded characters then `Buffer.byteLength()`
should be used to determine the number of bytes in a given encoding.
And Node does not check whether Content-Length and the length of the body
which has been transmitted are equal or not.

요청에 응답 헤더를 보낸다. 상태코드는 `404`같은 3자리 수의 HTTP 상태코드이다. 
마지막 파리미터인 `headers`는 응답 헤더다. 선택적으로 두번째 파라미터에 사람이 읽을 수 
있는 `reasonPhrase`를 전달할 수 있다. 

예제:

    var body = 'hello world';
    response.writeHead(200, {
      'Content-Length': body.length,
      'Content-Type': 'text/plain' });

이 메서드는 한 메시지에서 딱 한번만 호출되어야 하고 `response.end()`가 호출되기 전에 
호출되어야 한다.

이 메서드가 호출되기 전에 `response.write()`나 `response.end()`를 호출한다면 
암묵적이고 변할 가능성이 있는 헤더가 계산해서 이 함수를 호출할 것이다. 

Note: 해당 Content-Length는 문자가 아니라 바이트로 주어진다. 문자열 `'hello world'`는 
단일 바이트의 문자만 가지고 있기 때문에 위의 예제는 동작한다. 바디에 더 높은 코드의 문자가 
있다면 주어진 인코딩으로 바이트의 수를 결정하는데 `Buffer.byteLength()`를 사용할 것이다. 
그리고 Node는 전송된 바디의 길이와 Content-Length가 같은지 같지 않은지 확인하지 않는다.

### response.statusCode

When using implicit headers (not calling `response.writeHead()` explicitly), this property
controls the status code that will be send to the client when the headers get
flushed.

Example:

    response.statusCode = 404;

After response header was sent to the client, this property indicates the
status code which was sent out.

(명시적으로 `response.writeHead()`를 호출하지 않고) 암묵적인 헤더를 사용하는 경우 헤더가 플러시됐을 
때 클라이언트에 보낼 상태코드를 이 프로퍼티가 제어한다. 

예제:

    response.statusCode = 404;

응답해더가 클라이언트에 보내진 후 이 프로퍼티는 보낸 상태코드를 나타낸다.

### response.setHeader(name, value)

Sets a single header value for implicit headers.  If this header already exists
in the to-be-sent headers, its value will be replaced.  Use an array of strings
here if you need to send multiple headers with the same name.

Example:

    response.setHeader("Content-Type", "text/html");

or

    response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);

암묵적인 헤더에 단일 헤더값을 설정한다. 전송할 헤더에 이미 이 헤더가 존재한다면 해당 값은 
덮어써질 것이다. 같은 이름을 가진 여러 헤더를 전송해야 한다면 여기에 문자열 배열을 
사용해라.

예제:

    response.setHeader("Content-Type", "text/html");

또는

    response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);


### response.getHeader(name)

Reads out a header that's already been queued but not sent to the client.  Note
that the name is case insensitive.  This can only be called before headers get
implicitly flushed.

Example:

    var contentType = response.getHeader('content-type');

이미 큐에 들어갔지만 아직 클라이언트에는 보내지 않은 헤더를 읽는다. 이름이 대소문자를 
구분한다는 점에 주의해라. 이 함수는 헤더가 암묵적으로 플러시 되기 전에만 호출할 수 있다.

예제:

    var contentType = response.getHeader('content-type');

### response.removeHeader(name)

Removes a header that's queued for implicit sending.

Example:

    response.removeHeader("Content-Encoding");

암묵적으로 보내려고 큐에 있는 헤더르르 제거한다.

예제:

    var contentType = response.getHeader('content-type');


### response.write(chunk, [encoding])

If this method is called and `response.writeHead()` has not been called, it will
switch to implicit header mode and flush the implicit headers.

This sends a chunk of the response body. This method may
be called multiple times to provide successive parts of the body.

`chunk` can be a string or a buffer. If `chunk` is a string,
the second parameter specifies how to encode it into a byte stream.
By default the `encoding` is `'utf8'`.

**Note**: This is the raw HTTP body and has nothing to do with
higher-level multi-part body encodings that may be used.

The first time `response.write()` is called, it will send the buffered
header information and the first body to the client. The second time
`response.write()` is called, Node assumes you're going to be streaming
data, and sends that separately. That is, the response is buffered up to the
first chunk of body.

이 메서드는 호출하고 `response.writeHead()`는 호출하지 않았다면 암묵적인 헤더 모드로 
바꾸고 암묵적인 헤더를 플러시할 것이다.

이 메서드는 응답 바디의 청크를 전송한다. 바디의 연속적인 부분을 제공하기 위해 이 함수를 
여러번 호출할 수 있다.

`chunk`는 문자열이나 버퍼가 될 수 있다. `chunk`는 문자열이면 두번째 파라미터로 `chunk`를 
어떻게 바이트 스트림으로 인코딩할 것인지 지정한다. `encoding`의 기본값은  `'utf8'`이다.

**Note**: 이 메서드는 로우(raw) HTTP 바디이고 사용될 수도 있는 고수준의 multi-part 바디 
인코딩에서는 아무것도 하지 않는다.

처음 `response.write()`를 호출하면 버퍼되어 있는 헤더 정보와 첫 바디를 클라이언트에 보낼 
것이다. 두번째로 `response.write()`를 호출하면 Node는 데이터를 스트리밍 할 것이라고 
가정하고 나눠서 전송한다. 즉, 응답은 바디의 첫번재 청크에 버퍼된다.

### response.addTrailers(headers)

This method adds HTTP trailing headers (a header but at the end of the
message) to the response.

Trailers will **only** be emitted if chunked encoding is used for the
response; if it is not (e.g., if the request was HTTP/1.0), they will
be silently discarded.

Note that HTTP requires the `Trailer` header to be sent if you intend to
emit trailers, with a list of the header fields in its value. E.g.,

    response.writeHead(200, { 'Content-Type': 'text/plain',
                              'Trailer': 'Content-MD5' });
    response.write(fileData);
    response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
    response.end();

이 메서드는 HTTP trailing headers(헤더이지만 메시지 끝에 오는 헤더)를 응답에 
추가한다. 

응답에 chunked 인코딩을 사용한 **경우에만** Trailers가 발생할 것이다. chunked 인코딩이 
아니라면(요청이 HTTP/1.0인 등등) 이 헤더는 경고없이 버려질 것이다.

헤더의 값에 헤더 필드들의 리스트로 trailers를 발생시키려면 HTTP는 `Trailer`를 필요로 한다.

    response.writeHead(200, { 'Content-Type': 'text/plain',
                              'Trailer': 'Content-MD5' });
    response.write(fileData);
    response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
    response.end();


### response.end([data], [encoding])

This method signals to the server that all of the response headers and body
has been sent; that server should consider this message complete.
The method, `response.end()`, MUST be called on each
response.

If `data` is specified, it is equivalent to calling `response.write(data, encoding)`
followed by `response.end()`.

이 메서드는 모든 응답 헤더와 바디를 보냈다고 서버에 신호를 보낸다. 해당 서버는 이 메시지를 
완료된 것으로 간주해야 한다. `response.end()` 메서드는 반드시 각 응답마다 호출되어야 
한다.

`data`를 지정하면 `response.write(data, encoding)`를 호출한 다음에 `response.end()`를 
호출한 것과 같다.


## http.request(options, callback)

Node maintains several connections per server to make HTTP requests.
This function allows one to transparently issue requests.  `options` align
with [url.parse()](url.html#url.parse).

Options:

- `host`: A domain name or IP address of the server to issue the request to.
  Defaults to `'localhost'`.
- `hostname`: To support `url.parse()` `hostname` is preferred over `host`
- `port`: Port of remote server. Defaults to 80.
- `socketPath`: Unix Domain Socket (use one of host:port or socketPath)
- `method`: A string specifying the HTTP request method. Defaults to `'GET'`.
- `path`: Request path. Defaults to `'/'`. Should include query string if any.
  E.G. `'/index.html?page=12'`
- `headers`: An object containing request headers.
- `auth`: Basic authentication i.e. `'user:password'` to compute an
  Authorization header.
- `agent`: Controls [Agent](#http.Agent) behavior. When an Agent is used
  request will default to `Connection: keep-alive`. Possible values:
 - `undefined` (default): use [global Agent](#http.globalAgent) for this host
   and port.
 - `Agent` object: explicitly use the passed in `Agent`.
 - `false`: opts out of connection pooling with an Agent, defaults request to
   `Connection: close`.

`http.request()` returns an instance of the `http.ClientRequest`
class. The `ClientRequest` instance is a writable stream. If one needs to
upload a file with a POST request, then write to the `ClientRequest` object.

Example:

    var options = {
      host: 'www.google.com',
      port: 80,
      path: '/upload',
      method: 'POST'
    };

    var req = http.request(options, function(res) {
      console.log('STATUS: ' + res.statusCode);
      console.log('HEADERS: ' + JSON.stringify(res.headers));
      res.setEncoding('utf8');
      res.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    req.on('error', function(e) {
      console.log('problem with request: ' + e.message);
    });

    // write data to request body
    req.write('data\n');
    req.write('data\n');
    req.end();

Note that in the example `req.end()` was called. With `http.request()` one
must always call `req.end()` to signify that you're done with the request -
even if there is no data being written to the request body.

If any error is encountered during the request (be that with DNS resolution,
TCP level errors, or actual HTTP parse errors) an `'error'` event is emitted
on the returned request object.

There are a few special headers that should be noted.

* Sending a 'Connection: keep-alive' will notify Node that the connection to
  the server should be persisted until the next request.

* Sending a 'Content-length' header will disable the default chunked encoding.

* Sending an 'Expect' header will immediately send the request headers.
  Usually, when sending 'Expect: 100-continue', you should both set a timeout
  and listen for the `continue` event. See RFC2616 Section 8.2.3 for more
  information.

* Sending an Authorization header will override using the `auth` option
  to compute basic authentication.

Node는 HTTP 요청에 대한 연결을 서버당 여러 개 유지하고 있다. 이 함수는 투명하게 
요청을 진행한다. `options`은 [url.parse()](url.html#url.parse)를 지원한다.
Node maintains several connections per server to make HTTP requests.ㅏ

옵션:

- `host`: 요청을 보낼 서버의 도메인 명이나 IP 주소. 기본값은 `'localhost'`이다. 
- `hostname`: `url.parse()`를 지원하려면 `host`보다 `hostname`가 낫다.
- `port`: 원격서버의 포트. 기본포트는 80포트이다.
- `socketPath`: Unix 도메인 소켓 (host:port난 socketPath 중 하나를 사용한다)
- `method`: HTTP 요청 메서드를 지정하는 문자열. 기본값은 `'GET'`이다.
- `path`: 요청 경로. 기본값은 `'/'`이다. 필요하다면 쿼리스트링도 포함시킨다.
  예시. `'/index.html?page=12'`
- `headers`: 요청 헤더를 담고 있는 객체.
- `auth`: 기본 인증으롤 예를 들면 인증헤더를 계산하는 `'user:password'`이다.
- `agent`: [Agent](#http.Agent) 동작을 제어한다. 에이전트를 사용했을 때 요청은 
  기본적으로 `Connection: keep-alive`가 될 것이다. 가능한 값은 다음과 같다.
 - `undefined` (default): 이 호스트와 포트에 대한 [global Agent](#http.globalAgent)
   를 사용한다.
 - `Agent` object: 명시적으로 `Agent`에 전달된 객체를 사용한다.
 - `false`: Agent와 함께 연결 풀을 사용하지 않는다. 기본값은 
   `Connection: close`에 요청한다.

`http.request()`는 `http.ClientRequest` 클래스의 인스턴스를 리턴한다. 
`ClientRequest` 인스턴스는 쓰기가 가능한 스트림이다. POST 요청으로 파일을 
업로드해야 한다면 `ClientRequest` 객체에 작성한다.

예제:

    var options = {
      host: 'www.google.com',
      port: 80,
      path: '/upload',
      method: 'POST'
    };

    var req = http.request(options, function(res) {
      console.log('STATUS: ' + res.statusCode);
      console.log('HEADERS: ' + JSON.stringify(res.headers));
      res.setEncoding('utf8');
      res.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    req.on('error', function(e) {
      console.log('problem with request: ' + e.message);
    });

    // write data to request body
    req.write('data\n');
    req.write('data\n');
    req.end();

예제에서 `req.end()`를 호출했다. 요청 바디에 쓰여진 데이터가 없다고 하더라도 요청이 
완료되었다는 의미로 `http.request()`에서 항상 `req.end()`를 반드시 호출해야 한다. 

요청중에 어떤 오류가 있다면(DNS 처리나 TCP 레벨의 오류, 실제 HTTP 파싱 오류 등등) 
반환된 요청 객체에서 `'error'` 이벤트가 발생한다.

알아야 할 몇가지 특별한 헤더가 있다.

* 'Connection: keep-alive'를 전송하면 서버와의 연결을 다음 요청까지 유지해야 한다는 
  것을 Node에 알려줄 것이다.

* 'Content-length' 헤더를 전송하면 기본 chunked 인코딩을 사용하지 않을 것이다.

* 'Expect'헤더를 전송하면 즉시 요청헤더를 보낼 것이다. 'Expect: 100-continue'를 보낼 
  때는 보통 타임아웃과 `continue` 이벤트에 대한 linten을 둘 다 설정해야 한다. 더 
  자세한 내용은 RFC2616의 섹션 8.2.3을 봐라.

* 인증헤더를 전송하면 기본 인증을 계산하는 `auth` 옵션을 덮어쓸 것이다.

## http.get(options, callback)

Since most requests are GET requests without bodies, Node provides this
convenience method. The only difference between this method and `http.request()` is
that it sets the method to GET and calls `req.end()` automatically.

Example:

    var options = {
      host: 'www.google.com',
      port: 80,
      path: '/index.html'
    };

    http.get(options, function(res) {
      console.log("Got response: " + res.statusCode);
    }).on('error', function(e) {
      console.log("Got error: " + e.message);
    });

대부분의 요청은 바디가 없는 GET 요청이기 때문에 Node는 이 편리한 메서드를 제공한다. 
이 메서드와 `http.request()`간의 유일한 차이점은 자동적으로 메서드를 GET으로 설정하고 
`req.end()`를 호출한다는 점이다.

예제:

    var options = {
      host: 'www.google.com',
      port: 80,
      path: '/index.html'
    };

    http.get(options, function(res) {
      console.log("Got response: " + res.statusCode);
    }).on('error', function(e) {
      console.log("Got error: " + e.message);
    });


## Class: http.Agent

In node 0.5.3+ there is a new implementation of the HTTP Agent which is used
for pooling sockets used in HTTP client requests.

Previously, a single agent instance help the pool for single host+port. The
current implementation now holds sockets for any number of hosts.

The current HTTP Agent also defaults client requests to using
Connection:keep-alive. If no pending HTTP requests are waiting on a socket
to become free the socket is closed. This means that node's pool has the
benefit of keep-alive when under load but still does not require developers
to manually close the HTTP clients using keep-alive.

Sockets are removed from the agent's pool when the socket emits either a
"close" event or a special "agentRemove" event. This means that if you intend
to keep one HTTP request open for a long time and don't want it to stay in the
pool you can do something along the lines of:

    http.get(options, function(res) {
      // Do stuff
    }).on("socket", function (socket) {
      socket.emit("agentRemove");
    });

Alternatively, you could just opt out of pooling entirely using `agent:false`:

    http.get({host:'localhost', port:80, path:'/', agent:false}, function (res) {
      // Do stuff
    })

node 0.5.3이상부터는 HTTP 클라이언트 요청의 풀링 소켓에 사용하는 HTTP 에이전트의 
새로운 구현체가 있다. 

그 이전에 단일 에이전트 인스턴스는 하나의 host+port에 대한 풀을 도왔다. 지금의 구현체는 
많은 수의 호스트에 대한 소켓을 가지고 있다.

현재 HTTP 에이전트도 기본 클라이언트 요청은 Connection:keep-alive 를 사용한다. 소켓에서 
대기하고 있는 지연된 HTTP 요청이 없다면 해당 소켓을 닫는다. 즉 node의 풀은 부하가 있을 때 
keep-alive 이점을 가지지만 여전히 keep-alive를 사용하는 HTTP 클라이언트를 개발자가 
수동으로 닫을 필요가 없다.

소켓이 "close" 이벤트나 특수한 "agentRemove" 이벤트를 발생시켰을 때 이에전트의 풀에서 
소켓을 제거한다. 다시 말하면 HTTP 요청을 오랫동안 열어놓고 유지하고 HTTP 요청이 풀에 
유지되기를 원하지 않는다면 다음과 같이 할 수 있다.

    http.get(options, function(res) {
      // Do stuff
    }).on("socket", function (socket) {
      socket.emit("agentRemove");
    });

대신 `agent:false`를 사용해서 완전히 풀링을 사용하지 않을 수도 있다.

    http.get({host:'localhost', port:80, path:'/', agent:false}, function (res) {
      // Do stuff
    })

### agent.maxSockets

By default set to 5. Determines how many concurrent sockets the agent can have 
open per host.

기본적으로 5로 설정되어 있다. 에이전트가 얼마나 많은 동시 소켓을 호스트당 열 수 있는지를 
결정한다.

### agent.sockets

An object which contains arrays of sockets currently in use by the Agent. Do not 
modify.

Agent가 현재 사용하고 있는 소켓의 배열을 담고 있는 객체다. 수정하면 안된다.

### agent.requests

An object which contains queues of requests that have not yet been assigned to 
sockets. Do not modify.

소켓에 아직 할당되지 않은 요청의 큐를 담고 있는 객체다. 수정하면 안된다.

## http.globalAgent

Global instance of Agent which is used as the default for all http client
requests.

모든 HTTP 클라이언트 요청에 기본적으로 사용되는 Agent의 전역 인스턴스다. 


## Class: http.ClientRequest

This object is created internally and returned from `http.request()`.  It
represents an _in-progress_ request whose header has already been queued.  The
header is still mutable using the `setHeader(name, value)`, `getHeader(name)`,
`removeHeader(name)` API.  The actual header will be sent along with the first
data chunk or when closing the connection.

To get the response, add a listener for `'response'` to the request object.
`'response'` will be emitted from the request object when the response
headers have been received.  The `'response'` event is executed with one
argument which is an instance of `http.ClientResponse`.

During the `'response'` event, one can add listeners to the
response object; particularly to listen for the `'data'` event. Note that
the `'response'` event is called before any part of the response body is received,
so there is no need to worry about racing to catch the first part of the
body. As long as a listener for `'data'` is added during the `'response'`
event, the entire body will be caught.


    // Good
    request.on('response', function (response) {
      response.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    // Bad - misses all or part of the body
    request.on('response', function (response) {
      setTimeout(function () {
        response.on('data', function (chunk) {
          console.log('BODY: ' + chunk);
        });
      }, 10);
    });

Note: Node does not check whether Content-Length and the length of the body
which has been transmitted are equal or not.

The request implements the [Writable  Stream](stream.html#writable_stream)
interface. This is an `EventEmitter` with the following events:

내부적으로 생성하고 `http.request()`가 반환하는 객체다. 헤더는 이미 큐에 들어간 
_처리중인_ 요청을 나타낸다. `setHeader(name, value)`, `getHeader(name)`,
`removeHeader(name)` API를 사용해서 헤더를 여전히 변경할 수 있다. 실제 헤더는 
첫 데이터 청크와 함게 보내거나 연결이 닫힐 때 보낼 것이다. 

응답을 받으려면 응답 객체에 `'response'`에 대한 리스너를 추가해라. `'response'`는 
응답 헤더를 받았을 때 요청 객체에서 발생할 것이다. `'response'` 이벤트는 
`http.ClientResponse` 인스턴스를 아규먼트로 받아서 실행된다.

`'response'` 이벤트 가운데 응답 객체에 리스너들을 추가할 수 있다. 특히 `'data'` 
이벤트를 받기 위해 추가할 수 있다. `'response'` 이벤트는 응답 바디를 받기 전에 
실행되므로 바디의 첫 부분을 받기 위해 경쟁하는 것은 걱정할 필요가 없다. `'response'` 
이벤트 가운데 `'data'`에 대한 리스너가 추가되면 전체 바디를 받을 것이다.


    // 좋은 사례
    request.on('response', function (response) {
      response.on('data', function (chunk) {
        console.log('BODY: ' + chunk);
      });
    });

    // 나쁜 사례 - 바디 전체나 일부를 놓친다
    request.on('response', function (response) {
      setTimeout(function () {
        response.on('data', function (chunk) {
          console.log('BODY: ' + chunk);
        });
      }, 10);
    });

Note: Node는 Content-Length와 전송된 바디의 길이가 같은지 같지 않은지 확인하지 
않는다.

요청은 [Writable  Stream](stream.html#writable_stream) 인터페이스를 구현했다.
이는 다음 이벤트를 가진 `EventEmitter`이다.

### Event 'response'

`function (response) { }`

Emitted when a response is received to this request. This event is emitted only once. The
`response` argument will be an instance of `http.ClientResponse`.

Options:

- `host`: A domain name or IP address of the server to issue the request to.
- `port`: Port of remote server.
- `socketPath`: Unix Domain Socket (use one of host:port or socketPath)

`function (response) { }`

해당 요청에 대한 응답을 받았을 때 발생한다. 이 이벤트는 딱 한번만 발생한다. `response` 아규먼트는 
`http.ClientResponse`의 인스턴드가 될 것이다.

옵션:

- `host`: 요청을 보낼 서버의 도메인명이나 IP 주소다.
- `port`: 원격 서버의 포트.
- `socketPath`: Unix 도메인 소켓 (host:port나 socketPath 중 하나를 사용한다)

### Event: 'socket'

`function (socket) { }`

Emitted after a socket is assigned to this request.

`function (socket) { }`

해당 요청에 소켓이 할당된 후에 발생한다.

### Event: 'upgrade'

`function (response, socket, head) { }`

Emitted each time a server responds to a request with an upgrade. If this
event isn't being listened for, clients receiving an upgrade header will have
their connections closed.

A client server pair that show you how to listen for the `upgrade` event using `http.getAgent`:

    var http = require('http');
    var net = require('net');

    // Create an HTTP server
    var srv = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    srv.on('upgrade', function(req, socket, upgradeHead) {
      socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
                   'Upgrade: WebSocket\r\n' +
                   'Connection: Upgrade\r\n' +
                   '\r\n\r\n');

      socket.ondata = function(data, start, end) {
        socket.write(data.toString('utf8', start, end), 'utf8'); // echo back
      };
    });

    // now that server is running
    srv.listen(1337, '127.0.0.1', function() {

      // make a request
      var options = {
        port: 1337,
        host: '127.0.0.1',
        headers: {
          'Connection': 'Upgrade',
          'Upgrade': 'websocket'
        }
      };

      var req = http.request(options);
      req.end();

      req.on('upgrade', function(res, socket, upgradeHead) {
        console.log('got upgraded!');
        socket.end();
        process.exit(0);
      });
    });

`function (response, socket, head) { }`

업그레이드 요청에 서버가 응답할 때마다 발생한다. 이 이벤트가 바인딩되어 있지 않으면 
업그레이드 헤더를 받는 클라이언트는 연결이 닫힐 것이다.

`http.getAgent`를 사용해서 `upgrade` 이벤트를 어떻게 바인딩하는 지 보여주는 클라이언트와 서버 쌍의 예제다:

    var http = require('http');
    var net = require('net');

    // HTTP 서버 생성
    var srv = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    srv.on('upgrade', function(req, socket, upgradeHead) {
      socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
                   'Upgrade: WebSocket\r\n' +
                   'Connection: Upgrade\r\n' +
                   '\r\n\r\n');

      socket.ondata = function(data, start, end) {
        socket.write(data.toString('utf8', start, end), 'utf8'); // echo back
      };
    });

    // 이제 서버가 동작한다
    srv.listen(1337, '127.0.0.1', function() {

      // 요청 생성
      var options = {
        port: 1337,
        host: '127.0.0.1',
        headers: {
          'Connection': 'Upgrade',
          'Upgrade': 'websocket'
        }
      };

      var req = http.request(options);
      req.end();

      req.on('upgrade', function(res, socket, upgradeHead) {
        console.log('got upgraded!');
        socket.end();
        process.exit(0);
      });
    });

### Event: 'continue'

`function () { }`

Emitted when the server sends a '100 Continue' HTTP response, usually because
the request contained 'Expect: 100-continue'. This is an instruction that
the client should send the request body.

`function () { }`

보통 요청이 'Expect: 100-continue'를 답고 있기 때문에서버가 '100 Continue' HTTP 
응답을 보냈을 때 발생한다. 이는 클라이언트가 요청 바디를 보내야 한다는 것을 알려준다.

### request.write(chunk, [encoding])

Sends a chunk of the body.  By calling this method
many times, the user can stream a request body to a
server--in that case it is suggested to use the
`['Transfer-Encoding', 'chunked']` header line when
creating the request.

The `chunk` argument should be a [buffer](buffer.html) or a string.

The `encoding` argument is optional and only applies when `chunk` is a string.
Defaults to `'utf8'`.

바디의 청크를 전송한다. 이 메서드를 여러번 호출해서 
사용자는 요청 바디를 서버에 스트리밍할 수 있다. - 
이 경우 요청을 생성할 때 
`['Transfer-Encoding', 'chunked']` 헤더를 
사용하기를 제안한다.

`chunk` 아규먼트는 [buffer](buffer.html)나 문자열이 되어야 한다.

`encoding` 아규먼트는 선택사항이고 `chunk`가 문자열인 경우에만 적용된다. 
기본값은 `'utf8'`이다.


### request.end([data], [encoding])

Finishes sending the request. If any parts of the body are
unsent, it will flush them to the stream. If the request is
chunked, this will send the terminating `'0\r\n\r\n'`.

If `data` is specified, it is equivalent to calling
`request.write(data, encoding)` followed by `request.end()`.

요청 전송을 종료한다. 바디의 일부를 보내지 않았다면 스트림으로 플러시할 
것이다. 요청이 청크라면 종료하는 `'0\r\n\r\n'`를 보낼 것이다.

`data`를 지정하면 `request.end()` 다음에 
`request.write(data, encoding)`를 호출한 것과 같다.

### request.abort()

Aborts a request.  (New since v0.3.8.)

요청을 중단한다. (v0.3.8부터 추가되었다.)

### request.setTimeout(timeout, [callback])

Once a socket is assigned to this request and is connected 
[socket.setTimeout(timeout, [callback])](net.html#socket.setTimeout)
will be called.

해당 요청에 소켓이 바인딩되고 소켓이 연결되면 
[socket.setTimeout(timeout, [callback])](net.html#socket.setTimeout)
이 호출될 것이다.

### request.setNoDelay([noDelay])

Once a socket is assigned to this request and is connected 
[socket.setNoDelay(noDelay)](net.html#socket.setNoDelay)
will be called.

해당 요청에 소켓이 바인딩되고 소켓이 연결되면 
[socket.setNoDelay(noDelay)](net.html#socket.setNoDelay)
이 호출될 것이다.

### request.setSocketKeepAlive([enable], [initialDelay])

Once a socket is assigned to this request and is connected 
[socket.setKeepAlive(enable, [initialDelay])](net.html#socket.setKeepAlive)
will be called.

해당 요청에 소켓이 바인딩되고 소켓이 연결되면 
[socket.setKeepAlive(enable, [initialDelay])](net.html#socket.setKeepAlive)
이 호출될 것이다.

## http.ClientResponse

This object is created when making a request with `http.request()`. It is
passed to the `'response'` event of the request object.

The response implements the [Readable Stream](stream.html#readable_stream)
interface. This is an `EventEmitter` with the following events:

`http.request()`로 요청했을 때 생성되는 객체다. 요청 객체의 `'response'` 이벤트에 
전달된다.

응답은 [Readable Stream](stream.html#readable_stream) 인터페이스를 구현한다. 
이 객체는 다음 이벤트를 가지는 `EventEmitter`이다.


### Event: 'data'

`function (chunk) { }`

Emitted when a piece of the message body is received.

Note that the __data will be lost__ if there is no listener when a
`ClientResponse` emits a `'data'` event.

`function (chunk) { }`

메시지 바디의 일부를 받았을 때 발생한다.

`ClientResponse`가 `'data'` 이벤트를 발생시켰을 때 리스너가 없으면 
__데이터를 잃을 수 있다__는 것을 명심해라.


### Event: 'end'

`function () { }`

Emitted exactly once for each message. No arguments. After
emitted no other events will be emitted on the response.

`function () { }`

각 메시지마다 정확히 한번만 발생한다. 아규먼트는 없다. 
이 이벤트가 발생한 후에는 어떤 메시지도 응답에 발생시키지 않을 것이다.

### Event: 'close'

`function (err) { }`

Indicates that the underlaying connection was terminated before
`end` event was emitted.
See [http.ServerRequest](#http.ServerRequest)'s `'close'` event for more
information.

`function (err) { }`

`end` 이벤트가 발생하기 전에 의존하는 연결이 종료되었다는 것을 나타낸다. 

더 자세한 내용은 [http.ServerRequest](#http.ServerRequest)의 `'close'` 
이벤트를 봐라.

### response.statusCode

The 3-digit HTTP response status code. E.G. `404`.

3자리 수의 HTTP 응답 상태코드다. 예시: `404`.

### response.httpVersion

The HTTP version of the connected-to server. Probably either
`'1.1'` or `'1.0'`.
Also `response.httpVersionMajor` is the first integer and
`response.httpVersionMinor` is the second.

서버에 연결된 HTTP 버전이다. 아마 `'1.1'`나 `'1.0'`일 것이다.
`response.httpVersionMajor`는 첫 숫자이고 
`response.httpVersionMinor`는 두번째 숫자이다.

### response.headers

The response headers object.

응답 헤더 객체다.

### response.trailers

The response trailers object. Only populated after the 'end' event.

응답의 trailers 객체다. 'end' 이벤트 후에만 존재한다.

### response.setEncoding([encoding])

Set the encoding for the response body. Either `'utf8'`, `'ascii'`, or
`'base64'`. Defaults to `null`, which means that the `'data'` event will emit
a `Buffer` object.

응답 바디의 인코딩을 설정한다. `'utf8'`, `'ascii'`, `'base64'`가 될 수 있다.
기본값은 `'data'` 이벤트라 `Buffer` 객체를 발생시킨다는 의미로 `null`이다.

### response.pause()

Pauses response from emitting events.  Useful to throttle back a download.

발생하는 이벤트에서 응답을 멈춘다. 다운로드 트래픽을 조절하는데 유용하다.

### response.resume()

Resumes a paused response.

멈춘 응답을 복구한다.
