# net

    Stability: 3 - Stable

`net` 모듈은 비동기적인 네트워크 래퍼다. 서버와 클라이언트(스트림이라고 부른다.)를 모두
생성하는 메서드들이 포함되어 있다. `require('net');`로 이 모듈을 사용할 수 있다.

## net.createServer([options], [connectionListener])

새로운 TCP 서버를 생성한다. `connectionListener` 아규먼트는 자동으로
['connection'][] 이벤트의 리스너로 설정한다.

`options`은 다음 객체가 기본값이다.

    { allowHalfOpen: false
    }

`allowHalfOpen`가 `true`이면 소켓은 반대쪽 소켓이 FIN 패킷을 보냈을 때 자동으로
FIN 패킷을 보내지 않는다. 소켓은 읽을 수 없는 상태가 되지만 여전히 쓰기는 가능하다.
명시적으로 `end()` 메서드를 호출해야 한다. 더 자세한 내용은 ['end'][]
이벤트를 봐라.

다음은 8124 포트에 대한 연결을 받는 에코 서버의 예제다.

    var net = require('net');
    var server = net.createServer(function(c) { //'connection' listener
      console.log('client connected');
      c.on('end', function() {
        console.log('client disconnected');
      });
      c.write('hello\r\n');
      c.pipe(c);
    });
    server.listen(8124, function() { //'listening' listener
      console.log('server bound');
    });

이 예제는 `telnet`를 사용해서 테스트한다.

    telnet localhost 8124

`/tmp/echo.sock` 소켓에서 응답을 받으려면 마지막에서 세 번째 줄을 다음과 같이
바꾼다.

    server.listen('/tmp/echo.sock', function() { //'listening' listener

UNIX 도메인 소켓 서버에 접속하려면 `nc`를 사용해라.

    nc -U /tmp/echo.sock

## net.connect(options, [connectionListener])
## net.createConnection(options, [connectionListener])

새로운 ['net.Socket'](#net_class_net_socket)를 반환하는 팩토리 메서드로
제공된 주소와 포트로 연결한다.

소켓 구성이 완료되었을 때 ['connect'][] 이벤트가 발생할 것이다.

['net.Socket'](#net_class_net_socket)와 같은 이벤트를 가진다.

TCP 소켓에서 `options` 아규먼트는 다음을 지정하는 객체여야 한다.

  - `port`: 클라이언트가 접속할 포트 (필수).

  - `host`: 클라이언트가 접속할 호스트. 기본값은 `'localhost'`다.

  - `localAddress`: 네트워크 연결에 바인딩할 로컬 인터페이스.

UNIX계열 소켓에서는 `options` 아규먼트는 다음을 지정하는 객체여야 한다.

  - `path`: 클라이언트가 접속할 경로. (필수)

공통 옵션은 다음과 같다.

  - `allowHalfOpen`: 이 값이 `true`이면 소켓의 반대쪽에서 FIN 패킷을
    보냈을 때 자동으로 FIN 패킷을 보내지 않는다. 더 자세한 내용은 ['end'][]를
    봐라.

`connectListener` 파라미터를 ['connect'][] 이벤트의
리스터로 추가할 것이다.

다음은 이전에 설명한 에코 서버의 클라이언트 예제다.

    var net = require('net');
    var client = net.connect({port: 8124},
        function() { //'connect' listener
      console.log('connected to server!');
      client.write('world!\r\n');
    });
    client.on('data', function(data) {
      console.log(data.toString());
      client.end();
    });
    client.on('end', function() {
      console.log('disconnected from server');
    });

`/tmp/echo.sock` 소켓에 연결하려면 두번째 줄을 다음과 같이 변경한다.

    var client = net.connect({path: '/tmp/echo.sock'};

## net.connect(port, [host], [connectListener])
## net.createConnection(port, [host], [connectListener])

`host`의 `port`로 TCP 연결을 생성한다. `host`를 생략하면 `'localhost'`라고
가정한다.
`connectListener` 파라미터는 ['connect'][] 이벤트의 리스너로 추가될 것이다.

새로운 ['net.Socket'](#net_class_net_socket)를 반환하는 팩토리 메서드다.

## net.connect(path, [connectListener])
## net.createConnection(path, [connectListener])

`path`로 유닉스 소켓 연결을 생성한다.
`connectListener` 파라미터는 ['connect'][] 이벤트의 리스너로 추가될 것이다.

새로운 ['net.Socket'](#net_class_net_socket)를 반환하는 팩토리 메서드다.

## Class: net.Server

이 클래스는 TCP나 UNIX 서버를 생성하는 데 사용한다.

### server.listen(port, [host], [backlog], [callback])

지정한 `port`와 `host`에서 연결을 받아들이기 시작한다. `host`를 생략하면
모든 IPv4 주소(`INADDR_ANY`)에서 직접 들어오는 연결을 받아들일 것이다.
포트를 0으로 지정하면 임의의 포트를 사용할 것이다.

백로그는 대기하는 연결의 큐의 최대 길이이다.
실제 길이는 리눅스의 `tcp_max_syn_backlog`와 `somaxconn`같은 sysctl 설정으로
OS가 결정한다. 백로그의 기본값은 511이다.(512가 아니다)

이 함수는 비동기함수이다. 서버가 바인딩 되었을 때
['listening'][] 이벤트가 발생할 것이다.
마지막 파라미터 `callback`는 ['listening'][] 이벤트의
리스터로 추가할 것이다.

몇몇 사용자는 실행했을 때 `EADDRINUSE` 오류가 발생하는 이슈가 있다. 이는 다른 서버가
이미 요청한 포트에서 동작하고 있다는 것을 의미한다. 이 오류를 다루는 한 가지 방법은
약간 기다린 후에 다시 시도하는 것이다. 다음과 같은 코드로 처리할 수 있다.

    server.on('error', function (e) {
      if (e.code == 'EADDRINUSE') {
        console.log('Address in use, retrying...');
        setTimeout(function () {
          server.close();
          server.listen(PORT, HOST);
        }, 1000);
      }
    });

(주의: Node의 모든 소켓은 이미 `SO_REUSEADDR`를 설정한다.)


### server.listen(path, [callback])

전달한 `path`에서 연결을 받아들이는 UNIX 소켓서버를 시작한다.

이 함수는 비동기 함수이다. 서버가 바인딩되었을 때
['listening'][] 이벤트가 발생할 것이다.
마지막 파라미터 `callback`는 ['listening'][] 이벤트의
리스터로 추가될 것이다.

### server.listen(handle, [callback])

* `handle` {Object}
* `callback` {Function}

`handle` 객체는 서버나 소켓으로 설정되거나(의존하는 `_handle` 멤버를 가진
어떤 것이든) `{fd: <n>}` 객체가 될 수 있다.

이 함수는 서버가 지정한 핸들에서 연결을 받아드리도록 하지만 파일 디스크립터나
핸들이 이미 포트나 도메인 소켓에 바인딩 되었다고 가정한다.

Windows에서는 파일스크립터에서 연결을 받아들이는 것을 지원하지 않는다.

이 함수는 비동기 함수이다. 서버가 바인딩되었을 때
['listening'](#event_listening_) 이벤트가 발생할 것이다.
마지막 파라미터 `callback`는 ['listening'](#event_listening_) 이벤트의
리스터로 추가될 것이다.

### server.close([callback])

서버가 새로운 연결을 받아들이는 것과 현재 존재하는 연결을 유지하는 것을 멈춘다.
이 함수는 비동기 함수이고 모든 연결이 종료되고 서버에서 `'close'` 이벤트가
발생했을 때 완전히 닫힌다. 선택적으로 `'close'` 이벤트를 받는 콜백을
전달할 수 있다.

### server.address()

운영체제가 보고한 대로 서버에 바인딩 된 주소와 주소 패밀리 이름과 포트를 반환한다.
운영체제에 할당된 주소를 사용했을 때 할당된 포트를 찾는 데 유용하다.
세 개의 프로퍼티를 가진 객체를 반환한다. 예시:
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

예제:

    var server = net.createServer(function (socket) {
      socket.end("goodbye\n");
    });

    // grab a random port.
    server.listen(function() {
      address = server.address();
      console.log("opened server on %j", address);
    });

`'listening'` 이벤트가 발생할 때까지 `server.address()`를 호출하지 마라.

### server.unref()

서버에 `unref`를 호출하는 것은 해당 서버가 이벤트 시스템에서 유일하게 활성화된 서버인 경우
프로그램이 종료되도록 할 것이다. 서버에 이미 `unref`를 호출했다면 다시 `unref`를 호출해도
아무런 영향이 없다.

### server.ref()

`unref`와는 반대로 이미 `unref`된 서버에 `ref`를 호출하는 것은 해당 서버가 유일하게 남은
서버인 경우 프로그램이 종료되지 *않도록* 할 것이다.(기본 동작이다) 서버에 이미 `ref`를 호출했다면
다시 `ref`를 호출해도 아무런 영향이 없을 것이다.

### server.maxConnections

서버의 연결 수가 많아졌을 때 연결을 거절하려면 이 프로퍼티를 설정해라.

`child_process.fork()`로 자식에 소켓을 보냈을 때 이 옵션을 사용하는 것은
권하지 않는다.

### server.connections

이 함수는 **폐기되었다**. 대신 [server.getConnections()][]를 사용해라.
서버의 동시접속 수.

`child_process.fork()`로 자식 프로세스에 소켓을 보냈을 때 이 값은 `null`이 된다.
현재 활성화된 연결의 수를 얻거나 포크 하려면 대신 비동기 `server.getConnections`를 사용해라.

### server.getConnections(callback)

비동기적으로 서버의 현재 연결 수를 가져온다. 포크(fork)하려보 소켓을 보냈을 때 동작한다.

콜백은 두 개의 인자 `err`와 `count`를 받아야 한다.

`net.Server`는 다음 이벤트를 가진 [EventEmitter][]이다.

### Event: 'listening'

`server.listen`가 호출된 후 서버가 바인딩 되었을 때 발생한다.

### Event: 'connection'

* {Socket object} 연결 객체

새로운 연결이 생성되었을 때 발생한다. `socket`은 `net.Socket`의
인스턴스다.

### Event: 'close'

서버를 닫을 때 발생한다. 연결이 존재하는 경우
모든 연결이 종료될 때까지 이 이벤트틑 발생하지 않는다.

### Event: 'error'

* {Error Object}

오류가 생겼을 때 발생한다. 이 이벤트 후에 곧바로 `'close'` 이벤트가 호출될 것이다.
`server.listen` 설명에서의 예제를 봐라.

## Class: net.Socket

이 객체는 TCP나 UNIX 소켓의 추상화 객체다. `net.Socket` 인스턴스는 이중 Stream
인스턴스를 구현한다. `net.Socket` 인스턴스는 사용자가 생성해서 클라이언트처럼
(`connect()`를 사용해서) 사용하거나 Node가 생성해서 서버의 `'connection'` 이벤트로
사용자에게 전달할 수도 있다.

### new net.Socket([options])

새로운 소켓 객체를 생성한다.

`options`은 다음의 기본값을 가진 객체다.

    { fd: null
      allowHalfOpen: false,
      readable: false,
      writable: false
    }

`fd`로 이미 존재하는 소켓의 파일 디스크립터를 지정할 수 있다. 해당 소켓을 읽고 쓰기 가능하게
하려면 `readable`이나 `writable`를 각각 `true`로 설정해라.(주의: `fd`를 전달했을
때만 동작한다.)
`allowHalfOpen`에 대해서는 `createServer()`와 `'end'` 이벤트를 참고해라.

### socket.connect(port, [host], [connectListener])
### socket.connect(path, [connectListener])

주어진 소켓에 대한 연결을 오픈한다. `port`와 `host`를 전달하면 소켓은 TCP 소켓으로 열릴
것이고 `host`를 생략하면 `localhost`를 사용할 것이다. `path`를 전달하면 소켓은
해당 경로로의 유닉스 소켓으로 열릴 것이다.

`net.createConnection`이 소켓을 여는 한 이 메서드는 보통 필요하지 않다. 커스텀 Sokcet을
구현했을 때만 이 메서드를 사용해라.

이 함수는 비동기 함수이다. ['connect'][] 이벤트가 발생하면 소켓이 구성된
것이다. 연결하는 데 문제가 있다면 `'connect'` 이벤트는 발생하지 않고 `'error'` 이벤트가
예외와 함께 발생할 것이다.

`connectListener` 파라미터는 ['connect'][] 이벤트의 리스터로 추가된다.


### socket.bufferSize

`net.Socket`는 `socket.write()`가 항상 동작하는 프로퍼티를 가지고 있다. 이 프로퍼티는
사용자가 빠르게 준비해서 실행하도록 도와준다. 컴퓨터는 소켓에 작성된 데이터의 양을 항상
유지할 수는 없다. - 네트워크 연결은 쉽게 엄청나게 느려질 수도 있다. Node는 내부적으로 소켓에
작성된 데이터를 큐에 넣고 가능해 졌을 때 데이터를 보낼 것이다. (내부적으로 쓰기 가능한
상태를 유지하기 위해 소켓의 파일 디스크립트에서 폴링한다.)

이러한 내부적 버퍼링으로 인해서 메모리가 커질 수 있다. 이 프로퍼티는 작성하려는
버퍼 된 글자 수를 보여준다. (글자의 수는 대략 쓰일 바이트의 수와 같지만, 버퍼는
문자열을 담고 있을 것이고 문자열은 지연 인코딩된다. 그래서 바이트의 정확한 수는 알지
못한다.)

엄청나게 크거나 커지는 `bufferSize`를 경험한 사용자들은 프로그램에서 `pause()`와
`resume()`로 데이터의 흐름을 "조절하는(throttle)" 시도를 해야 한다.


### socket.setEncoding([encoding])

읽을 수 있는 스트림인 소켓의 인코딩을 설정한다. 더 자세한 내용은
[stream.setEncoding()][]를 봐라.

### socket.write(data, [encoding], [callback])

소켓에 데이터를 보낸다. 두 번째 파라미터는 문자열인 경우에 인코딩을 지정한다.
기본 인코딩은 UTF8 인코딩이다.

전체 데이터를 커널 버퍼에 성공적으로 플러시(flush)했다면 `true`를 반환한다.
데이터 일부나 전체가 사용자 메모리에 큐되었다면 `false`를 반환한다.
버퍼가 다시 비워졌을 때 `'drain'`이 발생할 것이다.

선택사항인 `callback` 파라미터는 데이터가 최종적으로 작성되었을 때 실행될 것이다.
이는 즉시 실행되는 것은 아니다.

### socket.end([data], [encoding])

소켓을 절반만 닫는다. 예를 들어 이 메서드는 FIN 패킷을 보낸다. 서버는 여전히 약간의
데이터를 보낼 수 있을 것이다.

`data`를 지정하면 `socket.end()`에 이어서 `socket.write(data, encoding)`를
호출한 것과 같다.

### socket.destroy()

이 소켓에는 더는 I/O 작업이 없다는 것은 보증한다. 오류가 발생한 경우에만 필요하다.
(오류를 파싱하는 등)

### socket.pause()

데이터를 읽는 것을 멈춘다. 즉, `'data'` 이벤트가 발생하지 않는다.
업로드를 차단하는 데 유용하다.

### socket.resume()

`pause()`를 호출한 후에 다시 읽기 시작한다.

### socket.setTimeout(timeout, [callback])

소켓에서 활동이 없는 `timeout` 밀리 초 후의 타임아웃에 소켓을 설정한다.
기본적으로 `net.Socket`에는 타임아웃이 없다.

유휴시간(idle timeout)이 실행되었을 때 소켓은 `'timeout'` 이벤트를 받지만
연결은 끊기지 않는다. 사용자는 소켓을 수동으로 `end()`하거나 `destroy()`해야 한다.

`timeout`이 0이면 존재하는 유휴시간이 사용할 수 없게 된다.

선택사항인 `callback` 파라미터는 `'timeout'` 이벤트의 일회성 리스터로 추가될 것이다.

### socket.setNoDelay([noDelay])

Nagle 알고리즘을 사용하지 않는다. 기본적으로 TCP 연결은 Nagle 알고리즘을 사용해서
데이터를 보내기 전에 데이터를 버퍼에 넣는다. `noDelay`를 `true`로 설정하면
`socket.write()`를 호출할 때마다 데이터를 즉시 보낼 것이다.
`noDelay`의 기본값은 `true`이다.

### socket.setKeepAlive([enable], [initialDelay])

keep-alive 기능을 사용하거나 사용하지 않고 비어있는 소켓에 첫 keepalive 조사를
하기 전에 초기화 지연을 선택적으로 설정한다.
`enable`의 기본값은 `false`다.

받은 마지막 데이터 패킷과 첫 keepalive 검사 사이에 지연시간을 설정하는데
`initialDelay`를(밀리 초로) 설정한다. initialDelay에 0을 설정하면 기본(혹은 이전의)
설정 값을 변경하지 않고 놔둘 것이다. 기본값은 `0`이다.


### socket.address()

운영체제가 보고한 대로 소켓에 바인딩 된 주소와 주소 패밀리 이름과 포트를 반환한다.
세 개의 프로퍼티를 가진 객체를 반환한다.
예시: `{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### socket.unref()

소켓에 `unref`를 호출하는 것은 해당 소켓이 이벤트 시스템에서 유일하게 활성화된 소켓일 때 프로그램이
종료되도록 할 것이다. 소켓에 이미 `unref`를 호출했다면 다시 `unref`를 호출해도 아무런 영향이
없을 것이다.

### socket.ref()

`unref`와는 반대로 이전에 `unref`된 소켓에 `ref`를 호출하는 것은 해당 소켓이 유일하게 남은
소켓일 때 프로그램이 종료되지 *않도록* 할 것이다.(기본 동작이다) 소켓에 이미 `ref`를 호출했다면
다시 `ref`를 호출해도 아무런 영향이 없을 것이다.

### socket.remoteAddress

원격 IP 주소의 문자열 표현이다. 예를 들면 `'74.125.127.100'`나
`'2001:4860:a005::68'`와 같은 식이다.

### socket.remotePort

원격 포트의 숫자 표현이다. 예를 들면 `80`이나 `21`이다.

### socket.localAddress

원격 클라이언트가 접속한 로컬 IP 주소의 문자열 표현이다. 예를 들어 `'0.0.0.0'`를 리스닝하고 있고
클라이언트가 `'192.168.1.1'`에 접속했다면 이 값은 `'192.168.1.1'`가 될 것이다.

### socket.localPort

로컬 포트의 숫자 표현이다. 예를 들면 `80`이나 `21`이다.

### socket.bytesRead

받은 바이트의 양.

### socket.bytesWritten

보낸 바이트의 양.


`net.Socket` 인스턴스는 다음의 이벤트를 가진 [EventEmitter][]이다.

### Event: 'connect'

소켓 연결이 성공적으로 이뤄졌을 때 발생한다.
`connect()`를 봐라.

### Event: 'data'

* {Buffer object}

데이터를 받았을 때 발생한다. `data` 아규먼트는 `Buffer`나 `String`이 될 것이다.
데이터의 인코딩은 `socket.setEncoding()`로 설정한다.
(더 자세한 내용은 [Readable Stream][]를 봐라.)

`Socket`이 `'data'` 이벤트를 발생시켰을 때 리스너가 없으면 __데이터를 잃어버릴 것이다.__

### Event: 'end'

소켓의 반대쪽 끝에서 FIN 패킷을 보냈을 때 발생한다.

기본적으로 (`allowHalfOpen == false`) 소켓은 미뤄두었던 작성 큐를 다 소비하면
자신의 파일 디스크립터를 파괴할 것이다.
하지만 `allowHalfOpen == true`로 설정하면 소켓은 이제 반대쪽에 `end()`할 필요가
있다는 경고와 함께 사용자가 임의의 양의 데이터를 작성할 수 있도록 반대쪽에 `end()`를
자동으로 호출하지 않을 것이다.


### Event: 'timeout'

유휴상태에서 소켓의 시간이 만료되었을 때 발생한다. 이는 소켓이 아무 일도 하지 않을 때만
알려준다. 사용자는 수동으로 연결을 닫아야 한다.

`socket.setTimeout()`도 봐라.


### Event: 'drain'

작성 버퍼가 비워졌을 때 발생한다. 업로드를 조절하는 데 사용할 수 있다.

`socket.write()`의 반환 값도 참고해라.

### Event: 'error'

* {Error object}

오류가 생겼을 때 발생한다. 이 이벤트에 이어서 곧바로 `'close'` 이벤트가 호출될 것이다.

### Event: 'close'

* `had_error` {Boolean} 소켓에 전송 오류가 있을 때 true 이다

소켓이 완전히 닫혔을 때 발생한다. `had_error` 아규먼트는 소켓이 전송 오류 때문에 닫혔는지를
알려주는 불리언 값이다.

## net.isIP(input)

input이 IP 주소인지 검사한다. 유효하지 않은 문자열이면 0을 반환하고 IP 버전 4 주소이면 4를
반환하고 IP 버전 6 주소이면 6을 반환한다.


## net.isIPv4(input)

input이 IP 버전 4 주소이면 true를 반환하고 IP 버전 4 주소가 아니면 false를 반환한다.


## net.isIPv6(input)

input이 IP 버전 6 주소이면 true를 반환하고 IP 버전 6 주소가 아니면 false를 반환한다.

['connect']: #net_event_connect
['connection']: #net_event_connection
['end']: #net_event_end
[EventEmitter]: events.html#events_class_events_eventemitter
['listening']: #net_event_listening
[server.getConnections()]: #net_server_getconnections_callback
[Readable Stream]: stream.html#stream_readable_stream
[stream.setEncoding()]: stream.html#stream_stream_setencoding_encoding
