# UDP / Datagram Sockets

    Stability: 3 - Stable

<!-- name=dgram -->

데이터그램 소켓은 `require('dgram')`로 사용할 수 있다.

중요사항: `dgram.Socket#bind()`의 동작은 v0.10에서 변경되어서 이제는 항상 비동기로 동작한다.
다음과 같이 작성했다면:

    var s = dgram.createSocket('udp4');
    s.bind(1234);
    s.addMembership('224.0.0.114');

다음과 같이 변경해야 한다.

    var s = dgram.createSocket('udp4');
    s.bind(1234, function() {
      s.addMembership('224.0.0.114');
    });


## dgram.createSocket(type, [callback])

* `type` 문자열. 'udp4'나 'udp6'
* `callback` 함수. `message` 이벤트에 리스너로 추가된다.
  선택사항
* 반환값: Socket 객체

지정한 타입의 데이터그램 소켓을 생성한다. 유효한 타입은 `udp4`와 `udp6`이다.

`message` 이벤트의 리스터로 추가되는 선택사항인 콜백을 받는다.

데이터그램을 받으려면 `socket.bind`를 호출해라. `socket.bind()`는 임의의 포트에서
"모든 인터페이스" 주소에 바인딩한다. (`udp4`와 `udp6` 소켓 둘다에 대해 맞는 것이다.)
그 다음 `socket.address().address`와 `socket.address().port`로 주소와 포트를
획득할 수 있다.

## Class: dgram.Socket

dgram Socket 클래스는 데이터그램 기능을 은닉화한다. 이 클래스는
`dgram.createSocket(type, [callback])`를 통해서 생성되어야 한다.

### Event: 'message'

* `msg` Buffer 객체. 메시지
* `rinfo` 객체. 원격 주소 정보

소켓에서 새로운 데이터그램을 사용할 수 있게 되었을 때 발생한다. `msg`는 `Buffer`이고 `rinfo`는
보내는 쪽의 주소정보와 데이터그램의 바이트 수를 담고 있는 객체다.

### Event: 'listening'

소켓이 데이트그램을 받기 시작했을 때 발생한다. 이는 UDP 소켓이 생성되자마자 발생한다.

### Event: 'close'

소켓이 `close()`로 닫혔을 때 발생한다. 소켓에서 더이상 새로운 `message` 이벤트가 발생하지 않는다.

### Event: 'error'

* `exception` Error 객체

오류가 생겼을 때 발생한다.

### socket.send(buf, offset, length, port, address, [callback])

* `buf` Buffer 객체.  보내는 메시지다
* `offset` 정수. 버퍼에서 메시지가 시작되는 오프셋.
* `length` 정수. 메시지의 바이트 수
* `port` 정수. 목적지 포트
* `address` 문자열. 목적지 IP
* `callback` 함수. 메시지가 배달완료되었을 때 호출될 콜백. 선택사항.

UDP 소켓에서 목적지 포트와 IP 주소는 반드시 지정해야 한다. `address` 파라미터에 문자열을
전달하고 이는 DNS로 처리될 것이다. 선택사항인 콜백은 DNS 오류와 `buf`가 다시 사용될 때를
탐지하기 위해 지정할 것이다. DNS 검색은 최소한 다음 tick까지 통신이 이뤄지는 시간동안 연기될
것이다. 통신이 콜백을 사용한다는 것을 확실하게 아는 유일한 방법이다.

소켓이 `bind` 호출로 이전에 바인딩되지 않았다면 임의의 포트번호를 할당받고
"모든 인터페이스" 주소에 바인딩 될 것이다.
(`udp4`에는 0.0.0.0, `udp6` 소켓에는 ::0)

`localhost`의 임의의 포트로 UDP 패킷을 보내는 예제;

    var dgram = require('dgram');
    var message = new Buffer("Some bytes");
    var client = dgram.createSocket("udp4");
    client.send(message, 0, message.length, 41234, "localhost", function(err, bytes) {
      client.close();
    });

**UDP 데이터그램 크기에 대한 내용**

`IPv4/v6` 데이터그램의 최대 크기는 `MTU` (_Maximum Transmission Unit_)와 `Payload Length` 필드
사이즈에 달려 있다.

- `Payload Length` 필드는 `16 bits` 크기이다. 이는 일반적인 탑재량(payload)은 인터넷
  헤더와 데이터를 포함해서 64K 옥텟보다 클 수가 없다는 의미이다
  (65,507 bytes = 65,535 − 8 bytes UDP header − 20 bytes IP header);
  이는 loopback 인터페이스에서 보통 true이지만 긴 데이터그램같은 것은 대부분의 호스트와
  네트워크에서 비현실적이다.

- `MTU`는 주어진 링크 계층 기술(link layer technology)이 데이터그램에 지원할 수 있는
  최대 사이즈이다. 전체를 받든지 부분만 받든지 IPv4에 대해 추천되는 `MTU`이 `576`인 한
  어떤 링크에서도 `IPv4`는 `68` 옥텟의 최소 `MTU`를 지정한다. (보통 다이얼업(dial-up)
  타입의 어플리케이션에 대해 `MTU`로 추천된다.)

  `IPv6`에서는 최소 `MTU`가 `1280` 옥텟이지만 강제적인 최소 단편 재조합 버퍼의 크기는
  `1500` 옥텟이다.
  가장 현대적인 링크계층 기술은 `1500`의 최소 `MTU`를 가지기 때문에(이더넷처럼)
  `68` 옥텟의 값은 아주 작다.

패킷이 이동하는 각 링크의 MTU를 미리 아는 것은 불가능하고 보통 (수신자) `MTU`보다 큰
데이터그램을 전송하는 것은 동작하지 않는다. (데이터가 의도된 수신자에게 도달하지 않았다는
것을 소스에 알리지 않고 패킷을 경고없이 버린다.)

### socket.bind(port, [address], [callback])

* `port` 정수
* `address` 문자열, 선택사항

UDP 소켓에서 `port`와 선택사항인 `address`에서 데이터그랩을 받는다. `address`를 지정하지
않으면 운영체제는 모든 주소에서 받을 것이다.

`callback`을 제공하는 경우 `callback`은 1회성 `'listening'` 이벤트 리스너로 추가된다.

UDP 서버가 41234 포트에서 받는 예제:

    var dgram = require("dgram");

    var server = dgram.createSocket("udp4");

    server.on("message", function (msg, rinfo) {
      console.log("server got: " + msg + " from " +
        rinfo.address + ":" + rinfo.port);
    });

    server.on("listening", function () {
      var address = server.address();
      console.log("server listening " +
          address.address + ":" + address.port);
    });

    server.bind(41234);
    // server listening 0.0.0.0:41234


### socket.close()

의존하는 소켓을 닫고 소켓에서 데이터를 받는 것을 멈춘다.

### socket.address()

소켓에 대한 주소 정보를 담고 있는 객체를 반환한다. UDP 소켓에서 이 객체는
`address`와 `family`와 `port`를 담고 있을 것이다.

### socket.setBroadcast(flag)

* `flag` 불리언

`SO_BROADCAST` 소켓 옵션을 설정하거나 없앤다. 이 옵션을 설정하면 로컬 인터페이스의 브로드캐스트
주소로 UDP 패킷을 보낼 것이다.

### socket.setTTL(ttl)

* `ttl` 정수

`IP_TTL` 소켓 옵션을 설정한다. TTL은 "Time to Live"를 의미하지만 이 상황에서 TTL은 패킷이
거쳐가도 되는 IP 홉(hop)의 수를 지정한다. 패킷이 지나쳐가는 각 라우터나 게이트웨이는 TTL을
감소시킨다. TTL이 라우터에 의해서 0까지 줄어들면 더이상 전송되지 않을 것이다. TTL 값을
변경하는 것은 보통 네트워크 검사나 멀티캐스팅 될 때 이뤄진다.

`setTTL()`의 아규먼트는 1부터 255사이의 홉 수이다. 대부분의 시스템에서 기본값은 64이다.

### socket.setMulticastTTL(ttl)

* `ttl` 정수

`IP_MULTICAST_TTL` 소켓 옵션을 설정한다. TTL은 "Time to Live"를 의미하지만 이 상황에서 TTL은
특히 멀티캐스트 트래픽에 대해서 패킷이 거쳐가도 되는 IP 홉(hop)의 수를 지정한다. 패킷이 지나쳐가는
각 라우터나 게이트웨이는 TTL을 감소시킨다. TTL이 라우터에 의해서 0까지 줄어들면 더이상 전송되지
않을 것이다.

`setMulticastTTL()`의 아규먼트는 0부터 255사이의 홉(hop) 수이다. 대부분의 시스템에서 기본값은 1이다.

### socket.setMulticastLoopback(flag)

* `flag` 불리언

`IP_MULTICAST_LOOP` 소켓 옵션을 설정하거나 제거한다. 이 옵셩을 설정하면 멀티캐스트 패킷도 로컬
인터페이스에서 받을 것이다.

### socket.addMembership(multicastAddress, [multicastInterface])

* `multicastAddress` 문자열
* `multicastInterface` 문자열, 선택사항

`IP_ADD_MEMBERSHIP` 소켓 옵션과 함께 멀티캐스트 그룹에 합류하도록 커널에 알려준다.

`multicastInterface`를 지정하지 않으면 운영체제는 회원을 유효한 모든 인터페이스에 추가하려고
할 것이다.

### socket.dropMembership(multicastAddress, [multicastInterface])

* `multicastAddress` 문자열
* `multicastInterface` 문자열, 선택사항

`addMembership`와는 반대로 `IP_DROP_MEMBERSHIP` 소켓 옵션과 함께 멀티캐스트 그룹에서
나오도록 커널에 명령한다. 이는 소캣이 닫히거나 진행이 종료되었을 때 커널이 자동적으로 호출할
것이므로 대부분의 어플리케이션은 이 함수를 호출할 필요가 없을 것이다.

`multicastInterface`를 지정하지 않으면 운영체제는 회원을 유효한 모든 인터페이스에서 버리려고
할 것이다.

### socket.unref()

소켓에서 `unref`를 호출하는 것은 해당 소켓이 이벤트 시스템에서 유일하게 활성화된 소켓인 경우
프로그램이 종료될 수 있게 할 것이다. 소켓에 이미 `unref`가 호출됐다면 다시 `unref`를 호출해도
아무런 영향이 없을 것이다.

### socket.ref()

`unref`과는 반대로 이전에 `unref`된 소켓에 `ref`를 호출하면 해당 소켓이 유일하게 남아있는
소켓인 경우 프로그램이 종료되지 *않도록* 할 것이다.(기본 동작) 소켓에 이미 `ref`를 호출했다면
`ref`를 다시 호출하는 것은 아무런 영향이 없을 것이다.
