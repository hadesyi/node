# TLS (SSL)

    Stability: 3 - Stable

이 모듈에 접근하려면 `require('tls')`를 사용해라.

`tls` 모듈은 TLS(Transport Layer Security)나 SSL(Secure Socket Layer)을
제공하는데 OpenSSL을 사용한다: 암호화된 스트림 통신

TLS/SSL는 공개키/개인 키 기반이다. 각 클라이언트와 서버는 개인 키를 반드시 가지고
있어야 한다. 개인 키는 다음과 같이 생성한다.

    openssl genrsa -out ryans-key.pem 1024

모든 서버와 몇몇 클라이언트는 인증서를 가질 필요가 있다. 인증서는 인증기관(Certificate
Authority)이나 자체서명으로 서명된 공개키이다. "인증서 서명 요청(Certificate Signing
Request)" (CSR) 파일을 생성하는 것이 인증서를 얻는 첫 단계이다. 다음과 같이 실행한다.

    openssl req -new -key ryans-key.pem -out ryans-csr.pem

CSR로 자체서명 인증서를 생성하려면 다음과 같이 한다.

    openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem

아니면 서명을 위해서 인증기관(Certificate Authority)에 CSR을 보낼 수 있다.

(TODO: CA를 생성하는 문서에 관심이 있는 사용자들은 Node 소스코드의
`test/fixtures/keys/Makefile`를 봐야 한다.)

다음과 같이 실행해서 .pfx나 .p12를 생성한다.

    openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
        -certfile ca-cert.pem -out agent5.pfx

  - `in`:  인증서
  - `inkey`: 개인키
  - `certfile`: `cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`와 같이 모든
    CA 인증서를 하나의 파일로 연결한다.


## Client-initiated renegotiation attack mitigation

<!-- type=misc -->

TLS 프로토콜은 클라이언트가 TLS 세션의 어떤 관점을 재협상하게 한다. 불행히도 세션 재협상은
DoS(denial-of-service) 공격의 잠재적인 요소인 서버 측 리소스의 양의 불균형을 일으킨다.

이를 완화하려면 재현상을 10분 내에 3번으로 제한해야 한다. 이 한계를 넘어서면
[CleartextStream][] 인스턴스에서 오류가 발생한다. 이 제한은
설정할 수 있다.

  - `tls.CLIENT_RENEG_LIMIT`: 재협상 제한, 기본값은 3이다.

  - `tls.CLIENT_RENEG_WINDOW`: 초 단위의 재협상 시간, 기본값은
                               10분이다.

무엇을 하는지 알지 못하면 기본값을 바꾸지 마라.

서버를 테스트하려면 `openssl s_client -connect address:port`로 서버에 연결한 뒤
`R<CR>` (`R`문자 뒤에 캐리지리턴)을 몇 번 입력한다.


## NPN and SNI

<!-- type=misc -->

NPN (Next Protocol Negotiation)와 SNI (Server Name Indication)는 TLS
핸드쉐이크 확장이다.

  * NPN - 다중 프로토콜(HTTP, SPDY)의 TLS 서버에 사용한다
  * SNI - 다른 SSL 인증서의 다중 호스트명의 TLS 서버에 사용한다.


## tls.getCiphers()

지원하는 SSL 암호 이름의 배열을 반환한다.

예제:

    var ciphers = tls.getCiphers();
    console.log(ciphers); // ['AES128-SHA', 'AES256-SHA', ...]


## tls.createServer(options, [secureConnectionListener])

새로운 [tls.Server][]를 생성한다.
`connectionListener` 아규먼트는 자동으로 [secureConnection][]
이벤트의 리스너로 설정된다.
`options` 객체는 다음과 같은 선택사항이 있다.

  - `pfx`: PFX나 PKCS12 형식으로 개인 키, 인증서, 서버의 CA 인증서를 담고 있는
    문자열이나 `Buffer`다. (`key`, `cert`, `ca` 옵션은 상호배타적이다.)

  - `key`: PEM 형식의 서버 개인 키를 담고 있는 문자열이나 `Buffer`다. (필수사항)

  - `passphrase`: 개인 키나 pfx에 대한 암호문의 문자열이다.

  - `cert`: PEM 형식의 서버 인증키를 담고 있는 문자열이나 `Buffer`다. (필수사항)

  - `ca`: 신뢰할 수 있는 인증서의 문자열이나 `Buffer` 배열로 PEM 형식이다. 이
    옵션을 생략하면 각각 VeriSign처럼 잘 알려진 "루트" CA를 사용할 것이다. 연결에
    권한을 부여하는 데 이것들을 사용한다.

  - `crl` : PEM으로 인코딩된 CRL(Certificate Revocation List)의 문자열이나 문자열의
    리스트

  - `ciphers`: 사용하거나 제외할 암호(cipher)를 설명하는 문자열이다.

    [BEAST attacks]을 완화하려면 CBC가 아닌 암호문을 우선시하도록 아래에서 설명할
    `honorCipherOrder`을 이 옵션과 함께 사용하기를 권장한다.

    기본값은
    `AES128-GCM-SHA256:RC4:HIGH:!MD5:!aNULL:!EDH`이다.
    형식에 대해 자세히 알고 싶으면 [OpenSSL cipher list format documentation]를 참고해라.
    ECDH (Elliptic Curve Diffie-Hellman) 암호화는 아직 지원하지 않는다.

    node.js가 OpenSSL 1.0.1이나 그 상위 버전과 연결되었을 때 `AES128-GCM-SHA256`를
    사용하고 클라이언트는 TLS 1.2, RC4을 안전한 폴백(fallback)으로 사용한다.

    **NOTE**: 이 섹션의 이전 버전에서는 `AES256-SHA`를 괜찮은 암호문으로 제안했었다. 하지만
    `AES256-SHA`는 CBC 암호문이므로 BEAST attacks을 받기 쉽다. `AES256-SHA`를
    사용하지 *말아라*.

  - `handshakeTimeout`: SSL/TLS 핸드쉐이크가 이 밀리 초 내에 끝나지 않으면 연결을 거부한다.
    기본값은 120초이다.

    핸드쉐이크가 타임아웃될 때마다 `tls.Server` 객체에서 `'clientError'`가 발생한다.

  - `honorCipherOrder` : 암호문을 선택할 때 클라이언트의 설정 대신에 서버의 설정을 사용해라.

    SSLv2을 사용한다면 서버는 클라이언트로 설정리스트를 보낼 것이고 클라이언트가 암호문을
    선택한다.

    이 옵션은 기본적으로는 사용안함으로 되어 있지만, BEAST attacks를 완화하려면 `ciphers`
    옵션과 함께 이 옵션을 사용하기를 *권장한다*.

  - `requestCert`: `true`로 지정하면 연결할 때 서버가 클라이언트한테 인증서를 요청하고
    인증서를 검증하는 데 사용할 것이다. 기본값: `false`.

  - `rejectUnauthorized`: `true`로 지정하면 제공된 CA 목록에서 인가되지 않은
    모든 연결을 서버가 거절할 것이다. 이 옵션은 `requestCert`가 `true`일 때만 유효하다.
    기본값: `false`.

  - `NPNProtocols`: 사용 가능한 NPN 프로토콜의 배열이나 `Buffer`다. (프로토콜은
    우선순위에 따라 정렬되어야 한다.)

  - `SNICallback`: 클라이언트가 SNI TLS 확장을 지원하는 경우 호출될 함수다. 이 함수는
    단 하나의 아규먼트인 `servername`만 받는다. 그리고 `SNICallback`는 SecureContext
    인스턴스를 반환해야 한다.
    (적합한 SecureContext를 얻으려면 `crypto.createCredentials(...).context`를
    사용할 수 있다.) `SNICallback`를 제공하지 않으면 고수준 API와 함께 기본 콜백을
    사용할 것이다.(아래를 참고해라.)

  - `sessionIdContext`: 세션 회수를 위한 불투명한 식별자를 담고 있는 문자열이다.
    `requestCert`가 `true`이면 기본값은 커맨드라인에서 생성한 MD5 해시값이다.
    `requestCert`가 `true`가 아니면 기본값은 제공하지 않는다.

  - `secureProtocol`: 사용할 SSL 방식. 예를 들어 SSL 버전 3을 사용하려면
    `SSLv3_method`이다. 사용 가능한 값은 설치한 OpenSSL에 따라 다르고
    상수 [SSL_METHODS][]에 정의되어 있다.

간단한 에코(echo) 서버의 예제다.

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),

      // 클라이언트 인증서 인증(certificate authentication)을 사용할 때만 필요하다.
      requestCert: true,

      // 클라이언트가 직접 서명한 인증서를 사용할 때만 필요하다.
      ca: [ fs.readFileSync('client-cert.pem') ]
    };

    var server = tls.createServer(options, function(cleartextStream) {
      console.log('server connected',
                  cleartextStream.authorized ? 'authorized' : 'unauthorized');
      cleartextStream.write("welcome!\n");
      cleartextStream.setEncoding('utf8');
      cleartextStream.pipe(cleartextStream);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });

또는

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('server.pfx'),

      // 클라이언트 인증서 인증(certificate authentication)을 사용할 때만 필요하다.
      requestCert: true,

    };

    var server = tls.createServer(options, function(cleartextStream) {
      console.log('server connected',
                  cleartextStream.authorized ? 'authorized' : 'unauthorized');
      cleartextStream.write("welcome!\n");
      cleartextStream.setEncoding('utf8');
      cleartextStream.pipe(cleartextStream);
    });
    server.listen(8000, function() {
      console.log('server bound');
    });
`openssl s_client`로 서버에 접속해서 테스트할 수 있다.


    openssl s_client -connect 127.0.0.1:8000


## tls.SLAB_BUFFER_SIZE

모든 tls 서버와 클라이언트가 사용하는 슬랩(slab) 버퍼의 크기
기본값: `10 * 1024 * 1024`.


무슨 작업을 하는지 알지 못한다면 기본값을 바꾸지 마라.


## tls.connect(options, [callback])
## tls.connect(port, [host], [options], [callback])

전달한 `port`와 `host`로(과거 API) 혹은 `options.port`와 `options.host`로
새로운 클라이언트 연결을 생성한다. (`host`를 지정하지 않으면
기본값은 `localhost`이다.) `options`는 지정된 객체여야 한다.

  - `host`: 클라이언트가 접속할 호스트

  - `port`: 클라이언트가 접속할 포트

  - `socket`: 새로운 소켓을 생성하는 대신 전달한 소켓으로 안전한 연결을 만든다.
    이 옵션을 지정하면 `host`와 `port`은 무시한다.

  - `pfx`: PFX나 PKCS12 형식으로 개인 키, 인증서 서버의 CA 인증서를 담고 있는 문자열이나
   `Buffer`다.

  - `key`: PEM 형식으로 클라이언트의 개인 키를 담고 있는 문자열이나 `Buffer`다.

  - `passphrase`: 개인 키나 pfx에 대한 암호문 문자열이다.

  - `cert`: PEM 형식으로 클라이언트의 인증서 키를 담고 있는 문자열이나 `Buffer`다.

  - `ca`: 신뢰할 수 있는 인증서의 문자열이나 `Buffer` 배열로 PEM 형식이다. 이 옵션을
    생락하면 각각 VeriSign처럼 잘 알려진 "루트" CA를 사용할 것이다. 연결에 권한을
    부여하는 데 이것들을 사용한다.

  - `rejectUnauthorized`: 이 값이 `true`이면 서버 인증서를 제공한 CA 리스트로 검증한다.
    검증에 실패하면 `'error'` 이벤트가 발생한다. 기본값: `true`.

  - `NPNProtocols`: 지원하는 NPN 프로토콜의 배열이나 `Buffer`다. `Buffer`는 다음의
    형식이어야 한다. `0x05hello0x05world`, 첫 바이트는 next protocol name의
    길이이다.(배열을 전달하는 것이 보통 훨씬 간단할 것이다: `['hello', 'world']`)

  - `servername`: SNI (Server Name Indication) TLS 확장에 대한 Servername이다.

  - `secureProtocol`: 사용할 SSL 방식. 예를 들어 SSL 버전 3을 사용하려면
    `SSLv3_method`이다. 사용 가능한 값은 설치한 OpenSSL에 따라 다르고
    상수 [SSL_METHODS][]에 정의되어 있다.

`callback` 파라미터는 ['secureConnect'][]
이벤트의 리스너로 추가할 것이다.

`tls.connect()`는 [CleartextStream][] 객체를 반환한다.

앞에서 설명한 에코 서버의 클라이언트 예제다.

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      // 클라이언트 인증서 인증(certificate authentication)을 사용할 때만 필요하다.
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-cert.pem'),

      // 클라이언트가 직접 서명한 인증서를 사용할 때만 필요하다.
      ca: [ fs.readFileSync('server-cert.pem') ]
    };

    var cleartextStream = tls.connect(8000, options, function() {
      console.log('client connected',
                  cleartextStream.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(cleartextStream);
      process.stdin.resume();
    });
    cleartextStream.setEncoding('utf8');
    cleartextStream.on('data', function(data) {
      console.log(data);
    });
    cleartextStream.on('end', function() {
      server.close();
    });

또는

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('client.pfx')
    };

    var cleartextStream = tls.connect(8000, options, function() {
      console.log('client connected',
                  cleartextStream.authorized ? 'authorized' : 'unauthorized');
      process.stdin.pipe(cleartextStream);
      process.stdin.resume();
    });
    cleartextStream.setEncoding('utf8');
    cleartextStream.on('data', function(data) {
      console.log(data);
    });
    cleartextStream.on('end', function() {
      server.close();
    });

## tls.createSecurePair([credentials], [isServer], [requestCert], [rejectUnauthorized])

두 스트림으로 새 안전한 쌍(pair) 객체를 생성한다. 두 스트림 중 하나는 암호화된 데이터를
읽고 쓰고 다른 하나는 평문 데이터를 읽고 쓴다.
보통 암호화된 쪽은 들어오는 암호화된 데이터 스트림에 파이프로 연결되고 평문 쪽은 초기
암호화 스트림에 대한 대체용으로 사용된다.

 - `credentials`: crypto.createCredentials( ... )의 증명서 객체

 - `isServer`: 이 tls 연결이 서버나 클라이언트로써 열려야 하는지를 나타내는 불리언값이다.

 - `requestCert`: 서버가 연결하는 클라이언트에서 인증서를 요구해야 하는지 나타내는 불리언
   값이다. 서버 연결에만 적용된다.

 - `rejectUnauthorized`: 서버가 유효하지 않은 인증서를 가진 클라이언트를 자동으로 거절할
   것인지 나타내는 불리언 값이다. `requestCert`를 사용하는 서버에만 적용된다.

`tls.createSecurePair()`는 [cleartext][]와 `encrypted` 스트림 프로퍼티로
SecurePair 객체를 반환한다.

## Class: SecurePair

tls.createSecurePair가 반환하는 클래스다.

### Event: 'secure'

쌍이 성공적으로 안전한 연결을 수립했을 때 SecurePair가 발생시키는 이벤트이다.

서버의 'secureConnection' 이벤트를 확인하는 것과 비슷하게 pair.cleartext.authorized는
사용된 인증서가 제대로 권한을 부여받았는지 승인하기 위해 확인되어야 한다.

## Class: tls.Server

`net.Server`의 하위클래스이고 `net.Server`와 같은 메서드를 가진다.
그냥 로우(raw) TCP 연결을 받아들이는 대신 TLS나 SSL을 사용해서 암호화된 연결을
받아들인다.

### Event: 'secureConnection'

`function (cleartextStream) {}`

새로운 연결이 성공적으로 핸드쉐이크를 했을 때 발생하는 이벤트이다. 아규먼트는
[CleartextStream](#tls.CleartextStream) 인스턴스다. 이 인스턴스는 공통 스트림 메서드와
이벤트를 모두 갖고 있다.

`cleartextStream.authorized`는 서버에 대해 제공된 인증서 권한 중 하나로 클라이언트가
검증되었는지 나타내는 불리언 값이다. `cleartextStream.authorized`가 false이면
`cleartextStream.authorizationError`는 어떻게 권한부여가 실패했는지를 알려주기 위해
설정된다. TLS 서버의 설정에 따라 권한이 부여되지 않은 연결을 받아들일 수 있다는 점을 알아두어야
한다. `cleartextStream.npnProtocol`는 선택된 NPN 프로토콜을 담고 있는 문자열이다.
`cleartextStream.servername`은 SNI로 요청된 servername을 담고 있는 문자열이다.


### Event: 'clientError'

`function (exception, securePair) { }`

안전한 연결이 이뤄지기 전에 클라이언트 연결이 'error' 이벤트를 발생시켰을 때 발생하는
이벤트다.

`securePair`는 오류가 발생한 `tls.SecurePair`이다.


### Event: 'newSession'

`function (sessionId, sessionData) { }`

TLS 세션을 생성했을 때 발생한다. 외부 스토리지에 세션을 저장할 때 사용할 것이다.


### Event: 'resumeSession'

`function (sessionId, callback) { }`

클라이언트가 이전의 TLS 세션을 되찾고자 할 때 발생한다. 이벤트 리스너는 주어진 `sessionId`를
사용해서 외부 스토리지에서 검색하고 검색이 완료되면 `callback(null, sessionData)`를 호출할
것이다. 세션을 되찾을 수 없다면(예: 스토리지에 존재하지 않는 경우) `callback(null, null)`를
호출할 것이다. `callback(err)`를 호출하면 들어오는 요청을 종료시키고 소켓을 제거할 것이다.


### server.listen(port, [host], [callback])

지정한 `port`와 `host`로 연결을 받아들이기 시작한다. `host`를 생략하면 서버는 모든
IPv4 주소(`INADDR_ANY`)에서 직접 연결을 받아들일 것이다.

이 함수는 비동기함수이다. 마지막 파라미터인 `callback`는 서버의 바인딩이 완료되었을 때
호출될 것이다.

더 자세한 내용은 `net.Server`를 봐라.


### server.close()

서버가 새로운 연결을 받아들이는 것을 멈춤다. 이 함수는 비동기 함수이고 서버가 `'close'`
이벤트를 발생시켰을 때 결국 닫힌다.

### server.address()

운영체제에서 보고된 서버가 바인딩 된 주소와 주소 패밀리이름과 포트를 반환한다.
더 자세한 내용은 [net.Server.address()][]를 봐라.

### server.addContext(hostname, credentials)

클라이언트 요청의 SNI 호스트이름이 전달한 `hostname`(와일드카드도 사용할 수 있다.)와
일치하는 경우 사용할 안전한 컨텍스트를 추가한다. `credentials`은 `key`, `cert`, `ca`를
포함할 수 있다.

### server.maxConnections

서버의 연결 수가 많아졌을 때 연결을 거절하려면 이 프로퍼티를 설정해라.

### server.connections

서버의 현재 연결 수


## Class: CryptoStream

이는 암호화된 스트림이다.

### cryptoStream.bytesWritten

의존하는 소켓의 bytesWritten 접근자로의 프록시로 이는 소켓에 쓰인 전체
바이트(*TLS 오버헤드를 포함해서*)를 반환할 것이다.

## Class: tls.CleartextStream

평문 데이터와 같이 암호화된 데이터도 읽고 쓸 수 있도록 *암호화된* 스트림에 기반을 둔
스트림이다.

이 인스턴스는 이중 [Stream](stream.html) 인터페이스를 구현한다.
이 인스턴스는 공통 스트림 메서드와 이벤트를 모두 가지고 있다.

ClearTextStream는 SecurePair 객체의 `clear` 멤버이다.

### Event: 'secureConnect'

새로운 연결이 성공적으로 핸드쉐이크를 했을 때 발생하는 이벤트이다.
리스너는 서버의 인증서가 권한 부여를 받았는지에 상관없이 호출될 것이다. 이는 서버 인증서가
지정한 CA 중 하나로 서명되었는지를 보려고 `cleartextStream.authorized`를 검사하는
사용자에게 달려있다. `cleartextStream.authorized === false`인 경우 오류는
`cleartextStream.authorizationError`에 있을 것이다. 또한 NPN이 사용되었다면 협상
프로토콜을 위해 `cleartextStream.npnProtocol`를 확인할 수 있다.

### cleartextStream.authorized

피어(peer) 인증서가 지정한 CA 중에 하나로 서명되었으면 `true` 그렇지 않으면 `false`인
불리언이다.

### cleartextStream.authorizationError

왜 피어(peer)의 인증서가 검증되지 못했는지에 대한 내용이다. 이 프로퍼티는
`cleartextStream.authorized === false`인 경우에만 사용할 수 있다.

### cleartextStream.getPeerCertificate()

피어(peer)의 인증서를 나타내는 객체를 반환한다. 반환된 객체는 인증서의 필드와 대응되는
몇몇 프로퍼티를 가지고 있다.

예제:

    { subject:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      issuer:
       { C: 'UK',
         ST: 'Acknack Ltd',
         L: 'Rhys Jones',
         O: 'node.js',
         OU: 'Test TLS Certificate',
         CN: 'localhost' },
      valid_from: 'Nov 11 09:52:22 2009 GMT',
      valid_to: 'Nov  6 09:52:22 2029 GMT',
      fingerprint: '2A:7A:C2:DD:E5:F9:CC:53:72:35:99:7A:02:5A:71:38:52:EC:8A:DF' }

피어(peer)가 인증서를 제공하지 않는다면 `null`이나 비어있는 객체를 반환한다.

### cleartextStream.getCipher()
현재 연결의 암호문 이름과 SSL/TLS 프로토콜 버전을 나타내는 객체를
반환한다.

예제:
{ name: 'AES256-SHA', version: 'TLSv1/SSLv3' }

자세한 내용은
http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_CIPHERS 에서
SSL_CIPHER_get_name()와 SSL_CIPHER_get_version()를 봐라.

### cleartextStream.address()

운영체제가 보고했듯이 기반을 두는 소켓의 바인딩 된 주소와 주소 패밀리 이름과 포트를 반환한다.
다음과 같이 세 가지 프로퍼티를 가진 객체를 반환한다.
`{ port: 12346, family: 'IPv4', address: '127.0.0.1' }`

### cleartextStream.remoteAddress

원격 IP 주소를 나타내는 문자열이다. 예를 들면 `'74.125.127.100'`나
`'2001:4860:a005::68'`이다.

### cleartextStream.remotePort

원격 포트를 나타내는 숫자다. 예를 들면 `443`.

[OpenSSL cipher list format documentation]: http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT
[BEAST attacks]: http://blog.ivanristic.com/2011/10/mitigating-the-beast-attack-on-tls.html
[CleartextStream]: #tls_class_tls_cleartextstream
[net.Server.address()]: net.html#net_server_address
['secureConnect']: #tls_event_secureconnect
[secureConnection]: #tls_event_secureconnection
[Stream]: stream.html#stream_stream
[SSL_METHODS]: http://www.openssl.org/docs/ssl/ssl.html#DEALING_WITH_PROTOCOL_METHODS
[tls.Server]: #tls_class_tls_server
