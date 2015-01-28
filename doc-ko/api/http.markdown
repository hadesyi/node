# HTTP

    Stability: 3 - Stable

HTTP 서버와 클라이언트를 사용하려면 `require('http')`를 사용해라.

Node의 HTTP 인터페이스는 전통적으로 다루기 어려웠던 프로토콜의 많은 기능을
지원하려고 디자인되었다. 특히 크고 청크로 인코딩 될 수 있는 메시지들이다.
인터페이스는 전체 요청이나 응답을 버퍼에 넣지 않는다. 사용자는 스트림 데이터를
버퍼에 넣을 수 있다.

HTTP 메시지 헤더는 다음과 같은 객체로 표현된다.

    { 'content-length': '123',
      'content-type': 'text/plain',
      'connection': 'keep-alive',
      'accept': '*/*' }

키는 소문자로 쓰고 값은 수정되지 않는다.

HTTP 애플리케이션이 가능한 전체 범위를 다 지원하기 위해서 Node의 HTTP API는 상당히
저수준의 API이다. API는 스트림 핸들링과 메시지 파싱만을 다룬다. 메시지를 헤더와 바디로
파싱하지만 실제 헤더와 바디는 파싱하지 않는다.


## http.STATUS_CODES

* {Object}

모든 표준 HTTP 응답 상태코드와 짧은 설명의 모음이다.
예를 들어 `http.STATUS_CODES[404] === 'Not Found'`와 같이 할 수 있다.

## http.createServer([requestListener])

새로운 웹서버 객체를 반환한다.

`requestListener`는 자동으로 `'request'` 이벤트에 추가되는
함수다.

## http.createClient([port], [host])

이 함수는 **폐기되었다**. 대신에 [http.request()][]를 사용해라.
새로운 HTTP 클라이언트를 구성한다. `port`와 `host`는 연결할 서버를 가리킨다.

## Class: http.Server

이 클래스는 다음의 이벤트를 가진 [EventEmitter][]다.

### Event: 'request'

`function (request, response) { }`

요청이 있을 때마다 발생한다. 연결마다 여러 번의 요청이 있을 수 있다.(keep-alive
연결인 경우)
 `request`는 [http.IncomingMessage][]의 인스턴스이고 `response`는
 `http.ServerResponse`의 인스턴스다.

### Event: 'connection'

`function (socket) { }`

 새로운 TCP 스트림이 생성되었을 때 발생한다. `socket`은 `net.Socket` 타입의
 객체다. 보통 사용자들은 이 이벤트에 접근하지 않을 것이다. 특히 프로토콜 파서가 소켓에 연결되는
 방법 때문에 소켓이 `readable` 이벤트를 발생시키지 않을 것이다. `socket`도
 `request.connection`에서 접근할 수 있다.

### Event: 'close'

`function () { }`

 서버가 닫혔을 때 발생한다.

### Event: 'checkContinue'

`function (request, response) { }`

http Expect: 100-continue 헤더를 가진 요청을 받을 때마다 발생한다.
이 이벤트가 바인딩 되지 않았다면 서버는 자동으로 적절한 100 Continue로 응답할
것이다.

이 이벤트를 다루면 클라이언트가 계속해서 요청 바디를 보내야 한다면
[response.writeContinue()][] 호출하고 클라이언트가 요청 바디를 계속 보내지 않는다면
적절한 HTTP 응답(예시: 400 Bad Request)을 생성한다.

### Event: 'connect'

`function (request, socket, head) { }`

클라이언트가 http CONNECT 메서드를 요청할 때마다 발생한다. 이 이벤트에 등록된
리스너가 없으면 CONNECT 메서드를 요청한 클라이언트의 연결이 닫힐 것이다.

* `request`는 request 이벤트와 같이 http 요청의 아규먼트다.
* `socket`는 서버와 클라이언트 간의 네트워크 소켓이다.
* `head`는 터널링 스트림의 첫 패킷인 Buffer의 인스턴스다. 이 값은 비어있을 것이다.

이 이벤트가 발생한 후 요청의 소켓은 `data` 이벤트 리스너를 가지지 않을 것이다.
즉, 해당 소켓으로 서버에 보낸 데이터를 다루려면 리스너에 바인딩해야 한다.

### Event: 'upgrade'

`function (request, socket, head) { }`

클라이언트가 http 업그레이드를 요청할 때마다 발생한다. 이 이벤트가 바인딩 되지 않았다면
업그레이드를 요청하는 클라이언트는 닫힌 연결을 할 것이다.

* `request`은 요청이벤트에 있는 것처럼 http 요청의 아규먼트다.
* `socket`은 서버와 클라이언트 사이의 네트워크 소켓이다.
* `head`는 Buffer의 인스턴스다. 업그레이드된 스트림의 첫 패킷이고 비어있을 수도 있다.

이 이벤트가 발생한 후에 요청의 소켓은 `data` 이벤트 리스너를 갖지 않을 것이다. 이는
해당 소켓으로 서버에 보내는 메시지를 다루기 위해서는 소켓에 바인딩할 필요가 있다는 의미이다.

`function (request, socket, head) { }`

### Event: 'clientError'

`function (exception, socket) { }`

클라이언트 연결에서 'error' 이벤트가 발생하면 이 이벤트가 진행된다.

`socket`은 오류가 발생한 `net.Socket` 객체이다.

### server.listen(port, [hostname], [backlog], [callback])

지정한 hostname과 port에서 연결을 받아들이기 시작한다. hostname을 생략하면 서버는
IPv4 주소(`INADDR_ANY`)에서 들어오는 연결을 모두 받아들일 것이다.

유닉스 소켓에 바인딩하려면 port와 hostname 대신에 파일명을 전달한다.

백로그는 지연되는 연결 큐의 최대길이이다.
실제 길이는 리눅스의 `tcp_max_syn_backlog`와 `somaxconn`같은 sysctl 설정을 통해
OS가 결정한다. backlog의 기본값은 511이다.(512가 아니다)

이 함수는 비동기 함수다. 마지막 파라미터 `callback`은
['listening'][] 이벤트의 리스터로 추가될 것이다.
[net.Server.listen(port)][]도 참고해 봐라.


### server.listen(path, [callback])

전달한 `path`에서 연결을 받아들이는 UNIX 소켓 서버를 시작한다.

이 함수는 비동기 함수다. 마지막 파라미터 `callback`은
['listening'][] 이벤트의 리스너로 추가될 것이다.
[net.Server.listen(path)][]도 참고해라.


### server.listen(handle, [callback])

* `handle` {Object}
* `callback` {Function}

`handle` 객체는 서버나 소켓(의존하는 `_handle` 멤버를 가진 어떤 것이든)으로
설정하거나 `{fd: <n>}` 객체로 설정할 수 있다.

이 함수는 서버가 지정한 핸들에서 연결을 받아들이도록 하지만 파일 디스크립터나
핸들이 이미 포트나 도메인 소켓에 바인딩 되어 있다고 가정한다.

윈도우는 파일 디스크립터에서 요청을 받아들이는 것을 지원하지 않는다.

이 함수는 비동기 함수다. 마지막 파라미터 `callback`은
['listening'](net.html#event_listening_) 이벤트의 리스너로 추가될 것이다.
[net.Server.listen()](net.html#net_server_listen_handle_callback)도 참고해라.

### server.close([callback])

Stops the server from accepting new connections.  See [net.Server.close()][].
서버가 새로운 연결을 받아들이는 것을 멈춘다. [net.Server.close()][]를 참고해라.


### server.maxHeadersCount

들어오는 헤더의 최대 수를 제한하고 기본값은 1000이다. 이 값을 0으로 설정하면
제한을 두지 않는다.

### server.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

소켓의 타임아웃 값을 설정하고 타임아웃이 발생하면 인자로 소켓을 넘기면서 Server 객체에
`'timeout'` 이벤트를 발생시킨다.

Server 객체에 `'timeout'` 이벤트 리스너가 있으면 타임아웃이 발생한 소켓을 인자로 전달하면서
이벤트 리스너를 호출할 것이다.

기본적으로 Server의 타임아웃 값은 2분이고 타임아웃이 발생했을 때 소켓을 자동으로 없앤다.
하지만 Server의 `'timeout'` 이벤트에 콜백을 할당하면 소켓 타임아웃을 처리하는 책임은
개발자에게 있다.

### server.timeout

* {Number} Default = 120000 (2 minutes)

소켓이 타임아웃 되었다고 판단하기 전에 활동하지 않는 밀리 초 시간.

소켓 타임아웃 로직이 연결에 설정되므로 이 값을 변경해도 서버에 이미 존재하는 연결이 아닌
*새로운* 연결에만 영향을 끼칠 것이다.

들어오는 요청에 자동 타임아웃 동작을 모두 비활성화하려면 0으로 설정해라.

## Class: http.ServerResponse

사용자가 아니라 HTTP 서버가 내부적으로 생성하는 객체다. `'request'` 이벤트의
두 번째 파라미터로 전달된다.

응답은 [Writable Stream][] 인터페이스를 구현했다.
이 클래스는 다음의 이벤트를 가지고 있는 [EventEmitter][]이다.

### Event: 'close'

`function () { }`

[response.end()][]가 호출되기 전이나 플러시할 수 있을 때 의존하는 연결을
종료했다는 것을 나타낸다.

### Event: 'finish'

`function () { }`

응답을 보냈을 때 발생한다. 조금 더 자세히 얘기하면 응답 헤더와 바디의 마지막 부분을 네트워크를
통해서 전송하기 위해서 운영체제에 전달했을 때 이 이벤트가 발생한다. 이는 클라이언트는
아직 아무것도 받지 않았다는 것을 의미하지 않는다.

이 이벤트 뒤에는 응답 객체에서 어떤 이벤트도 더는 발생하지 않을 것이다.

### response.writeContinue()

클라이언트에 요청 바디가 보내지리라는 것을 나타내는 HTTP/1.1 100 Continue
메시지를 보낸다. `Server`의 ['checkContinue'][] 이벤트를 봐라.

### response.writeHead(statusCode, [reasonPhrase], [headers])

요청에 응답 헤더를 보낸다. 상태코드는 `404`같은 3자리 수의 HTTP 상태코드이다.
마지막 파라미터인 `headers`는 응답 헤더다. 선택적으로 두 번째 파라미터에 사람이 읽을 수
있는 `reasonPhrase`를 전달할 수 있다.

예제:

    var body = 'hello world';
    response.writeHead(200, {
      'Content-Length': body.length,
      'Content-Type': 'text/plain' });

이 메서드는 한 메시지에서 딱 한 번만 호출되어야 하고 [response.end()][]가 호출되기 전에
호출되어야 한다.

이 메서드가 호출되기 전에 [response.write()][]나 [response.end()][]를 호출한다면
암묵적이고 변할 가능성이 있는 헤더가 계산해서 이 함수를 호출할 것이다.

Note: 해당 Content-Length는 문자가 아니라 바이트로 주어진다. 문자열 `'hello world'`는
단일 바이트의 문자만 가지고 있기 때문에 위의 예제는 동작한다. 바디에 더 높은 코드의 문자가
있다면 주어진 인코딩으로 바이트의 수를 결정하는데 `Buffer.byteLength()`를 사용할 것이다.
그리고 Node는 전송된 바디의 길이와 Content-Length가 같은지 같지 않은지 확인하지 않는다.

### response.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

Socket의 타임아웃값을 `msecs`로 설정한다. 콜백을 지정하면 응답 객체의
`'timeout'` 이벤트 리스너로 추가한다.

요청, 응답, 서버에 `'timeout'` 리스너를 추가하지 않으면 타임아웃 되었을 때 소켓을 제거한다.
요청, 응답, 서버의 `'timeout'` 이벤트에 핸들러를 할당하면 타임아웃된 소켓을 처리하는 것은
개발자의 몫이다.

### response.statusCode

(명시적으로 [response.writeHead()][]를 호출하지 않고) 암묵적인 헤더를 사용하는 경우
헤더가 플러시 됐을 때 클라이언트에 보낼 상태코드를 이 프로퍼티가 제어한다.

예제:

    response.statusCode = 404;

응답헤더가 클라이언트에 보내진 후 이 프로퍼티는 보낸 상태코드를 나타낸다.

### response.setHeader(name, value)

암묵적인 헤더에 단일 헤더 값을 설정한다. 전송할 헤더에 이미 이 헤더가 존재한다면 해당 값은
덮어써 질 것이다. 같은 이름을 가진 여러 헤더를 전송해야 한다면 여기에 문자열 배열을
사용해라.

예제:

    response.setHeader("Content-Type", "text/html");

또는

    response.setHeader("Set-Cookie", ["type=ninja", "language=javascript"]);

### response.headersSent

불리언 값(읽기 전용). 헤더를 보냈으면 true이고 보내지 않았으면 false이다.

### response.sendDate

이 값이 ture이면 이미 헤더에 Date 헤더가 존재하지 않으며 자동으로 Date 헤더를
생성해서 응답에 보낼 것이다. 기본값은 true이다.

이 값은 테스트할 때만 사용하지 않도록 해야 한다. HTTP는 응답에 Date 헤더를 필요로
한다.

### response.getHeader(name)

이미 큐에 들어갔지만, 아직 클라이언트에는 보내지 않은 헤더를 읽는다. 이름이 대소문자를
구분한다는 점에 주의해라. 이 함수는 헤더가 암묵적으로 플러시 되기 전에만 호출할 수 있다.

예제:

    var contentType = response.getHeader('content-type');

### response.removeHeader(name)

암묵적으로 보내려고 큐에 있는 헤더를 제거한다.

예제:

    var contentType = response.getHeader('content-type');


### response.write(chunk, [encoding])

이 메서드는 호출하고 [response.writeHead()][]는 호출하지 않았다면 암묵적인 헤더 모드로
바꾸고 암묵적인 헤더를 플러시할 것이다.

이 메서드는 응답 바디의 청크를 전송한다. 바디의 연속적인 부분을 제공하기 위해 이 함수를
여러번 호출할 수 있다.

`chunk`는 문자열이나 버퍼가 될 수 있다. `chunk`는 문자열이면 두 번째 파라미터로 `chunk`를
어떻게 바이트 스트림으로 인코딩할 것인지 지정한다. `encoding`의 기본값은  `'utf8'`이다.

**Note**: 이 메서드는 로우(raw) HTTP 바디이고 사용될 수도 있는 고수준의 multi-part 바디
인코딩에서는 아무것도 하지 않는다.

처음 `response.write()`를 호출하면 버퍼 되어 있는 헤더 정보와 첫 바디를 클라이언트에 보낼
것이다. 두 번째로 `response.write()`를 호출하면 Node는 데이터를 스트리밍 할 것이라고
가정하고 나눠서 전송한다. 즉, 응답은 바디의 첫 번째 청크에 버퍼링된다.

전체 데이터가 성공적으로 커널 버퍼에 플러시 되면 `true`를 반환한다. 전체 혹은 일부의 데이터가
사용자 메모리에 큐로 들어가면 `false`를 반환한다.
버퍼가 다시 여유가 생기면 `'drain'`가 발생할 것이다.

### response.addTrailers(headers)

이 메서드는 HTTP trailing headers(헤더이지만 메시지 끝에 오는 헤더)를 응답에
추가한다.

응답에 chunked 인코딩을 사용한 **경우에만** Trailers가 발생할 것이다. chunked 인코딩이
아니라면(요청이 HTTP/1.0인 등등) 이 헤더는 경고 없이 버려질 것이다.

헤더의 값에 헤더 필드들의 리스트로 trailers를 발생시키려면 HTTP는 `Trailer`를 필요로 한다.

    response.writeHead(200, { 'Content-Type': 'text/plain',
                              'Trailer': 'Content-MD5' });
    response.write(fileData);
    response.addTrailers({'Content-MD5': "7895bf4b8828b55ceaf47747b4bca667"});
    response.end();


### response.end([data], [encoding])

이 메서드는 모든 응답 헤더와 바디를 보냈다고 서버에 신호를 보낸다. 해당 서버는 이 메시지를
완료된 것으로 간주해야 한다. `response.end()` 메서드는 반드시 응답마다 호출되어야
한다.

`data`를 지정하면 `response.write(data, encoding)`를 호출한 다음에
`response.end()`를 호출한 것과 같다.


## http.request(options, [callback])

Node는 HTTP 요청에 대한 연결을 서버마다 여러 개 유지하고 있다. 이 함수는 투명하게
요청을 진행한다.

`options`은 객체나 문자열이 될 수 있다. `options`이 문자열이면 자동으로
[url.parse()][]를 사용해서 파싱한다.

옵션:

- `host`: 요청을 보낼 서버의 도메인 명이나 IP 주소. 기본값은 `'localhost'`이다.
- `hostname`: `url.parse()`를 지원하려면 `host`보다 `hostname`가 낫다.
- `port`: 원격 서버의 포트. 기본 포트는 80 포트이다.
- `localAddress`: 네트워크 연결에 바인딩할 로컬 인터페이스.
- `socketPath`: Unix 도메인 소켓 (host:port나 socketPath 중 하나를 사용한다.)
- `method`: HTTP 요청 메서드를 지정하는 문자열. 기본값은 `'GET'`이다.
- `path`: 요청 경로. 기본값은 `'/'`이다. 필요하다면 쿼리스트링도 포함한다.
  예시. `'/index.html?page=12'`
- `headers`: 요청 헤더를 담고 있는 객체.
- `auth`: 기본 인증이다. 예를 들면 Authorization 헤더를 처리하는 `'user:password'`이다.
- `agent`: [Agent][] 동작을 제어한다. 에이전트를 사용했을 때 요청은
  기본적으로 `Connection: keep-alive`가 될 것이다. 가능한 값은 다음과 같다.
 - `undefined` (default): 이 호스트와 포트에 대한 [global Agent][]를 사용한다.
 - `Agent` object: 명시적으로 `Agent`에 전달된 객체를 사용한다.
 - `false`: Agent와 함께 연결 풀을 사용하지 않는다. 기본값은
   `Connection: close`에 요청한다.

선택사항인 `callback` 파라미터는 ['response'][] 이벤트의 일회성 리스터로 추가될 것이다.

`http.request()`는 [http.ClientRequest][] 클래스의 인스턴스를 반환한다.
`ClientRequest` 인스턴스는 쓰기가 가능한 스트림이다. POST 요청으로 파일을
업로드해야 한다면 `ClientRequest` 객체에 작성한다.

예제:

    var options = {
      hostname: 'www.google.com',
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

    // 요청 바디에 데이터를 쓴다
    req.write('data\n');
    req.write('data\n');
    req.end();

예제에서 `req.end()`를 호출했다. 요청 바디에 쓰인 데이터가 없다고 하더라도 요청이
완료되었다는 의미로 `http.request()`에서 항상 `req.end()`를 반드시 호출해야 한다.

요청 중에 어떤 오류가 있다면(DNS 처리나 TCP 레벨의 오류, 실제 HTTP 파싱 오류 등등)
반환된 요청 객체에서 `'error'` 이벤트가 발생한다.

알아야 할 몇 가지 특별한 헤더가 있다.

* 'Connection: keep-alive'를 전송하면 서버와의 연결을 다음 요청까지 유지해야 한다는
  것을 Node에 알려줄 것이다.

* 'Content-length' 헤더를 전송하면 기본 chunked 인코딩을 사용하지 않을 것이다.

* 'Expect'헤더를 전송하면 즉시 요청 헤더를 보낼 것이다. 'Expect: 100-continue'를 보낼
  때는 보통 타임아웃과 `continue` 이벤트에 대한 linten을 둘 다 설정해야 한다. 더
  자세한 내용은 RFC2616의 섹션 8.2.3을 봐라.

* 인증 헤더를 전송하면 기본 인증을 계산하는 `auth` 옵션을 덮어쓸 것이다.

## http.get(options, callback)

대부분의 요청은 바디가 없는 GET 요청이기 때문에 Node는 이 편리한 메서드를 제공한다.
이 메서드와 `http.request()`간의 유일한 차이점은 자동으로 메서드를 GET으로 설정하고
`req.end()`를 호출한다는 점이다.

예제:

    http.get("http://www.google.com/index.html", function(res) {
      console.log("Got response: " + res.statusCode);
    }).on('error', function(e) {
      console.log("Got error: " + e.message);
    });


## Class: http.Agent

node 0.5.3 이상부터는 HTTP 클라이언트 요청의 풀링 소켓에 사용하는 HTTP 에이전트의
새로운 구현체가 있다.

그 이전에 단일 에이전트 인스턴스는 하나의 host+port에 대한 풀을 도왔다. 지금의 구현체는
많은 수의 호스트에 대한 소켓을 가지고 있다.

현재 HTTP 에이전트도 기본 클라이언트 요청은 Connection:keep-alive 를 사용한다. 소켓에서
대기하고 있는 지연된 HTTP 요청이 없다면 해당 소켓을 닫는다. 즉 node의 풀은 부하가 있을 때
keep-alive 이점을 가지지만 여전히 keep-alive를 사용하는 HTTP 클라이언트를 개발자가
수동으로 닫을 필요가 없다.

소켓이 "close" 이벤트나 특수한 "agentRemove" 이벤트를 발생시켰을 때 에이전트의 풀에서
소켓을 제거한다. 다시 말하면 HTTP 요청을 오랫동안 열어놓고 유지하고 HTTP 요청이 풀에
유지되기를 원하지 않는다면 다음과 같이 할 수 있다.

    http.get(options, function(res) {
      // Do stuff
    }).on("socket", function (socket) {
      socket.emit("agentRemove");
    });

대신 `agent:false`를 사용해서 완전히 풀링을 사용하지 않을 수도 있다.

    http.get({hostname:'localhost', port:80, path:'/', agent:false}, function (res) {
      // Do stuff
    })

### agent.maxSockets

기본적으로 5로 설정되어 있다. 에이전트가 얼마나 많은 동시 소켓을 출처(origin)마다 열 수 있는지를
결정한다. 출처는 'host:port'나 'host:port:localAddress' 조합이다.

### agent.sockets

Agent가 현재 사용하고 있는 소켓의 배열을 담고 있는 객체다. 수정하면 안 된다.

### agent.requests

소켓에 아직 할당되지 않은 요청의 큐를 담고 있는 객체다. 수정하면 안 된다.

## http.globalAgent

모든 HTTP 클라이언트 요청에 기본적으로 사용되는 Agent의 전역 인스턴스다.


## Class: http.ClientRequest

내부적으로 생성하고 `http.request()`가 반환하는 객체다. 헤더는 이미 큐에 들어간
_처리 중인_ 요청을 나타낸다. `setHeader(name, value)`, `getHeader(name)`,
`removeHeader(name)` API를 사용해서 헤더를 여전히 변경할 수 있다. 실제 헤더는
첫 데이터 청크와 함께 보내거나 연결이 닫힐 때 보낼 것이다.

응답을 받으려면 응답 객체에 `'response'`에 대한 리스너를 추가해라. `'response'`는
응답 헤더를 받았을 때 요청 객체에서 발생할 것이다. `'response'` 이벤트는
[http.IncomingMessage][] 인스턴스를 아규먼트로 받아서 실행된다.

`'response'` 이벤트 가운데 응답 객체에 리스너들을 추가할 수 있다. 특히 `'data'`
이벤트를 받기 위해 추가할 수 있다.

`'response'` 핸들러를 추가하지 않았다면 응답을 완전히 버릴 것이다. 하지만 `'response'` 이벤트
핸들러를 추가했다면 `'readable'` 이벤트가 발생할 때마다 `response.read()`를 호출하거나
`'data'` 핸들러를 추가하거나 `.resume()` 메서드를 호출해서 응답객체의 데이터를 **반드시**
소비해야 한다. 데이터가 소비될 때까지 `'end'` 이벤트는 발생하지 않을 것이다. 또한, 데이터를 읽는
동안 메모리를 소비하므로 'process out of memory' 오류가 발생할 수도 있다.

Note: Node는 Content-Length와 전송된 바디의 길이가 같은지 같지 않은지 확인하지
않는다.

요청은 [Writable Stream][] 인터페이스를 구현했다.
이는 다음 이벤트를 가진 [EventEmitter][]이다.

### Event 'response'

`function (response) { }`

해당 요청에 대한 응답을 받았을 때 발생한다. 이 이벤트는 딱 한 번만 발생한다. `response` 아규먼트는
[http.IncomingMessage][]의 인스턴스가 될 것이다.

옵션:

- `host`: 요청을 보낼 서버의 도메인 명이나 IP 주소다.
- `port`: 원격 서버의 포트.
- `socketPath`: Unix 도메인 소켓 (host:port나 socketPath 중 하나를 사용한다.)

### Event: 'socket'

`function (socket) { }`

해당 요청에 소켓이 할당된 후에 발생한다.

### Event: 'connect'

`function (response, socket, head) { }`

CONNECT 메서드의 요청에 서버가 응답할 때마다 발생한다. 이 이벤트에 등록된 리스너가 없으면
CONNECT 메서드를 받는 클라이언트의 연결을 닫힐 것이다.

클라이언트와 서버가 어떻게 `connect` 이벤트를 받는지 보여준다.

    var http = require('http');
    var net = require('net');
    var url = require('url');

    // HTTP 터널링 프록시를 생성한다
    var proxy = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    proxy.on('connect', function(req, cltSocket, head) {
      // 원래의 서버로 연결한다
      var srvUrl = url.parse('http://' + req.url);
      var srvSocket = net.connect(srvUrl.port, srvUrl.hostname, function() {
        cltSocket.write('HTTP/1.1 200 Connection Established\r\n' +
                        'Proxy-agent: Node-Proxy\r\n' +
                        '\r\n');
        srvSocket.write(head);
        srvSocket.pipe(cltSocket);
        cltSocket.pipe(srvSocket);
      });
    });

    // 이제 프록시서버가 동작한다
    proxy.listen(1337, '127.0.0.1', function() {

      // 터널링 프록시에 대한 요청을 만든다
      var options = {
        port: 1337,
        hostname: '127.0.0.1',
        method: 'CONNECT',
        path: 'www.google.com:80'
      };

      var req = http.request(options);
      req.end();

      req.on('connect', function(res, socket, head) {
        console.log('got connected!');

        // HTTP 터널을 통해 요청을 만든다
        socket.write('GET / HTTP/1.1\r\n' +
                     'Host: www.google.com:80\r\n' +
                     'Connection: close\r\n' +
                     '\r\n');
        socket.on('data', function(chunk) {
          console.log(chunk.toString());
        });
        socket.on('end', function() {
          proxy.close();
        });
      });
    });

### Event: 'upgrade'

`function (response, socket, head) { }`

업그레이드 요청에 서버가 응답할 때마다 발생한다. 이 이벤트가 바인딩 되어 있지 않으면
업그레이드 헤더를 받는 클라이언트는 연결이 닫힐 것이다.

`upgrade` 이벤트를 어떻게 바인딩하는지 보여주는 클라이언트와 서버 쌍의 예제다.

    var http = require('http');

    // HTTP 서버 생성
    var srv = http.createServer(function (req, res) {
      res.writeHead(200, {'Content-Type': 'text/plain'});
      res.end('okay');
    });
    srv.on('upgrade', function(req, socket, head) {
      socket.write('HTTP/1.1 101 Web Socket Protocol Handshake\r\n' +
                   'Upgrade: WebSocket\r\n' +
                   'Connection: Upgrade\r\n' +
                   '\r\n');

      socket.pipe(socket); // echo back
    });

    // 이제 서버가 동작한다
    srv.listen(1337, '127.0.0.1', function() {

      // 요청 생성
      var options = {
        port: 1337,
        hostname: '127.0.0.1',
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

보통 요청이 'Expect: 100-continue'를 담고 있기 때문에 서버가 '100 Continue' HTTP
응답을 보냈을 때 발생한다. 이는 클라이언트가 요청 바디를 보내야 한다는 것을 알려준다.

### request.write(chunk, [encoding])

바디의 청크를 전송한다. 이 메서드를 여러 번 호출해서
사용자는 요청 바디를 서버에 스트리밍 할 수 있다. -
이 경우 요청을 생성할 때
`['Transfer-Encoding', 'chunked']` 헤더를
사용하기를 제안한다.

`chunk` 아규먼트는 [Buffer][]나 문자열이 되어야 한다.

`encoding` 아규먼트는 선택사항이고 `chunk`가 문자열인 경우에만 적용된다.
기본값은 `'utf8'`이다.


### request.end([data], [encoding])

요청 전송을 종료한다. 바디의 일부를 보내지 않았다면 스트림으로 플러시할
것이다. 요청이 청크라면 종료하는 `'0\r\n\r\n'`를 보낼 것이다.

`data`를 지정하면 `request.end()` 다음에
`request.write(data, encoding)`를 호출한 것과 같다.

### request.abort()

요청을 중단한다. (v0.3.8부터 추가되었다.)

### request.setTimeout(timeout, [callback])

해당 요청에 소켓이 바인딩 되고 소켓이 연결되면
[socket.setTimeout()][]이 호출될 것이다.

### request.setNoDelay([noDelay])

해당 요청에 소켓이 바인딩 되고 소켓이 연결되면
[socket.setNoDelay()][]이 호출될 것이다.

### request.setSocketKeepAlive([enable], [initialDelay])

해당 요청에 소켓이 바인딩 되고 소켓이 연결되면
[socket.setKeepAlive()][]이 호출될 것이다.

## http.IncomingMessage

`IncomingMessage` 객체는 [http.Server][]나 [http.ClientRequest][]가 생성하고
`'request'`와 `'response'` 이벤트에 각각 첫 번째 인자로 전달된다. 응답 상태,
헤더, 데이터에 접근할 때 사용한다.

이는 [Readable Stream][] 인터페이스를 구현했고 다음의 추가적인 이벤트, 메서드,
프로퍼티를 구현했다.

### Event: 'close'

`function () { }`

의존하는 연결이 닫혔는지를 나타낸다.
`'end'`처럼 이 이벤트는 응답마다 딱 한 번만 발생한다.

### message.httpVersion

서버 요청의 경우 HTTP 버전을 클라이언트가 보낸다. 클라이언트 응답의 경우 서버에 연결된
HTTP 버전이다. 아마 `'1.1'`나 `'1.0'` 둘 중 하나일 것이다.

`response.httpVersionMajor`는 첫 번째 정수이고
`response.httpVersionMinor`는 두 번째 정수이다.

### message.headers

요청/응답 헤더 객체.

읽기 전용인 헤더 이름과 값의 맵이다. 헤더 이름은 소문자이다.
예제:

    // Prints something like:
    //
    // { 'user-agent': 'curl/7.22.0',
    //   host: '127.0.0.1:8000',
    //   accept: '*/*' }
    console.log(request.headers);

### message.trailers

요청/응답의 trailers 객체다. 'end' 이벤트 후에만 존재한다.

### message.setTimeout(msecs, callback)

* `msecs` {Number}
* `callback` {Function}

`message.connection.setTimeout(msecs, callback)`를 호출한다.

### message.method

**[http.Server][]에서 얻은 요청에만 유효하다.**

요청 메서드의 문자열 표현. 읽기 전용이다.
예제: `'GET'`, `'DELETE'`.

### message.url

**[http.Server][]에서 얻은 요청에만 유효하다..**

요청 URL 문자열. 이 값은 실제 HTTP 요청에 있는 URL만 담고 있다.
요청이 다음과 같다면

    GET /status?name=ryan HTTP/1.1\r\n
    Accept: text/plain\r\n
    \r\n

`request.url`는 다음과 같을 것이다.

    '/status?name=ryan'

URL을 각 부분으로 파싱하고 싶다면 `require('url').parse(request.url)`를
사용할 수 있다. 예제:

    node> require('url').parse('/status?name=ryan')
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: 'name=ryan',
      pathname: '/status' }

쿼리스트링에서 파라미터를 추출하고 싶다면 `require('querystring').parse` 함수를
사용하거나 `require('url').parse`에 두 번째 인자로 `true`를 전달할 수 있다.
예제:

    node> require('url').parse('/status?name=ryan', true)
    { href: '/status?name=ryan',
      search: '?name=ryan',
      query: { name: 'ryan' },
      pathname: '/status' }

### message.statusCode

**`http.ClientRequest`에서 얻은 응답에만 유효하다.**

3자리 숫자의 HTTP 응답 상태코드. 예를 들면 `404`등이다.

### message.socket

연결과 관련된 `net.Socket` 객체이다.

HTTPS를 지원한다면 클라이언트의 자세한 인증을 얻을 때
request.connection.verifyPeer()와 request.connection.getPeerCertificate()를
사용해라.


['checkContinue']: #http_event_checkcontinue
['listening']: net.html#net_event_listening
['response']: #http_event_response
[Agent]: #http_class_http_agent
[Buffer]: buffer.html#buffer_buffer
[EventEmitter]: events.html#events_class_events_eventemitter
[Readable Stream]: stream.html#stream_class_stream_readable
[Writable Stream]: stream.html#stream_class_stream_writable
[global Agent]: #http_http_globalagent
[http.ClientRequest]: #http_class_http_clientrequest
[http.IncomingMessage]: #http_http_incomingmessage
[http.ServerResponse]: #http_class_http_serverresponse
[http.Server]: #http_class_http_server
[http.request()]: #http_http_request_options_callback
[http.request()]: #http_http_request_options_callback
[net.Server.close()]: net.html#net_server_close_callback
[net.Server.listen(path)]: net.html#net_server_listen_path_callback
[net.Server.listen(port)]: net.html#net_server_listen_port_host_backlog_callback
[response.end()]: #http_response_end_data_encoding
[response.write()]: #http_response_write_chunk_encoding
[response.writeContinue()]: #http_response_writecontinue
[response.writeHead()]: #http_response_writehead_statuscode_reasonphrase_headers
[socket.setKeepAlive()]: net.html#net_socket_setkeepalive_enable_initialdelay
[socket.setNoDelay()]: net.html#net_socket_setnodelay_nodelay
[socket.setTimeout()]: net.html#net_socket_settimeout_timeout_callback
[stream.setEncoding()]: stream.html#stream_stream_setencoding_encoding
[url.parse()]: url.html#url_url_parse_urlstr_parsequerystring_slashesdenotehost
