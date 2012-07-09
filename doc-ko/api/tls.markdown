# TLS (SSL)

<!--english start-->

    Stability: 3 - Stable

Use `require('tls')` to access this module.

The `tls` module uses OpenSSL to provide Transport Layer Security and/or
Secure Socket Layer: encrypted stream communication.

TLS/SSL is a public/private key infrastructure. Each client and each
server must have a private key. A private key is created like this

    openssl genrsa -out ryans-key.pem 1024

All severs and some clients need to have a certificate. Certificates are public
keys signed by a Certificate Authority or self-signed. The first step to
getting a certificate is to create a "Certificate Signing Request" (CSR)
file. This is done with:

    openssl req -new -key ryans-key.pem -out ryans-csr.pem

To create a self-signed certificate with the CSR, do this:

    openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem

Alternatively you can send the CSR to a Certificate Authority for signing.

(TODO: docs on creating a CA, for now interested users should just look at
`test/fixtures/keys/Makefile` in the Node source code)

To create .pfx or .p12, do this:

    openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
        -certfile ca-cert.pem -out agent5.pfx

  - `in`:  certificate
  - `inkey`: private key
  - `certfile`: all CA certs concatenated in one file like
    `cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`

<!--english end-->

    Stability: 3 - Stable

이 모듈에 접근하려면 `require('tls')`를 사용해라.

`tls` 모듈은 TLS(Transport Layer Security)나 SSL(Secure Socket Layer)을 
제공하는데 OpenSSL을 사용한다: 암호화된 스트림 통신

TLS/SSL는 공개키/개인키 기반이다. 각 클라이언트와 서버는 개인키를 반드시 가지고 
있어야 한다. 개인키는 다음과 같이 생성한다.

    openssl genrsa -out ryans-key.pem 1024

모든 서버와 몇몇 클라이언트는 인증서를 가질 필요가 있다. 인증서는 인증기관(Certificate 
Authority)이나 자체서명으로 서명된 공개키이다. "인증서 서명 요청(Certificate Signing 
Request)" (CSR) 파일을 생성하는 것이 인증서를 얻는 첫 단계이다. 다음과 같이 실행한다.

    openssl req -new -key ryans-key.pem -out ryans-csr.pem

CSR로 자체서명 인증서를 생성하려면 다음과 같이 한다.

    openssl x509 -req -in ryans-csr.pem -signkey ryans-key.pem -out ryans-cert.pem

아니면 서명을 위해서 인증기관(Certificate Authority)에 CSR을 보낼 수 있다.

(TODO: CA를 생성하는 문서에 관심이 있느 사용자들은 Node 소스크도의 
`test/fixtures/keys/Makefile`를 봐야한다.)

다음과 같이 실행해서 .pfx나 .p12를 생성한다.

    openssl pkcs12 -export -in agent5-cert.pem -inkey agent5-key.pem \
        -certfile ca-cert.pem -out agent5.pfx

  - `in`:  인증서
  - `inkey`: 개인키
  - `certfile`: `cat ca1-cert.pem ca2-cert.pem > ca-cert.pem`와 같이 모든 
    CA 인증서를 하나의 파일로 연결한다.


## Client-initiated renegotiation attack mitigation

<!--english start-->

<!-- type=misc -->

The TLS protocol lets the client renegotiate certain aspects of the TLS session.
Unfortunately, session renegotiation requires a disproportional amount of
server-side resources, which makes it a potential vector for denial-of-service
attacks.

To mitigate this, renegotiations are limited to three times every 10 minutes. An
error is emitted on the [CleartextStream](#tls.CleartextStream) instance when
the threshold is exceeded. The limits are configurable:

  - `tls.CLIENT_RENEG_LIMIT`: renegotiation limit, default is 3.

  - `tls.CLIENT_RENEG_WINDOW`: renegotiation window in seconds, default is
                               10 minutes.

Don't change the defaults unless you know what you are doing.

To test your server, connect to it with `openssl s_client -connect address:port`
and tap `R<CR>` (that's the letter `R` followed by a carriage return) a few
times.

<!--english end-->

<!-- type=misc -->

TLS 프로토콜는 클라이언트가 TLS 세션의 어떤 관점을 재협상하게 한다. 불행히도 세션 재협상은 
DoS(denial-of-service) 공격의 잠재적인 요소인 서버측 리소스의 양의 불균형을 야기시킨다.

이를 완화시키려면 재현상을 10분내에 3번으로 제한해야 한다. 이 한계를 넘어서면 
[CleartextStream](#tls.CleartextStream) 인스턴스에서 오류가 발생한다. 이 제한은 
설정할 수 있다.

  - `tls.CLIENT_RENEG_LIMIT`: 재협상 제한, 기본값은 3이다.

  - `tls.CLIENT_RENEG_WINDOW`: 초단위의 재협상 시간, 기본값은
                               10 분이다.

무엇을 하는지 알지 못하면 기본값을 바꾸지 마라.

서버를 테스트하려면 `openssl s_client -connect address:port`로 서버에 연결한 뒤
`R<CR>` (`R`문자 뒤에 캐리지리턴)을 몇번 입력한다.


## NPN and SNI

<!--english start-->

<!-- type=misc -->

NPN (Next Protocol Negotiation) and SNI (Server Name Indication) are TLS
handshake extensions allowing you:

  * NPN - to use one TLS server for multiple protocols (HTTP, SPDY)
  * SNI - to use one TLS server for multiple hostnames with different SSL
    certificates.

<!--english end-->

<!-- type=misc -->

NPN (Next Protocol Negotiation)와 SNI (Server Name Indication)는 TLS
핸드쉐이크 확장이다.

  * NPN - 다중 프로토콜(HTTP, SPDY)의 TLS 서버에 사용한다
  * SNI - 다른 종류의 SSL 인증서의 다중 호스트네임의 TLS 서버에 사용한다.


## tls.createServer(options, [secureConnectionListener])

<!--english start-->

Creates a new [tls.Server](#tls.Server).
The `connectionListener` argument is automatically set as a listener for the
[secureConnection](#event_secureConnection_) event.
The `options` object has these possibilities:

  - `pfx`: A string or `Buffer` containing the private key, certificate and
    CA certs of the server in PFX or PKCS12 format. (Mutually exclusive with
    the `key`, `cert` and `ca` options.)

  - `key`: A string or `Buffer` containing the private key of the server in
    PEM format. (Required)

  - `passphrase`: A string of passphrase for the private key or pfx.

  - `cert`: A string or `Buffer` containing the certificate key of the server in
    PEM format. (Required)

  - `ca`: An array of strings or `Buffer`s of trusted certificates. If this is
    omitted several well known "root" CAs will be used, like VeriSign.
    These are used to authorize connections.

  - `ciphers`: A string describing the ciphers to use or exclude. Consult
    <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT> for
    details on the format.

  - `requestCert`: If `true` the server will request a certificate from
    clients that connect and attempt to verify that certificate. Default:
    `false`.

  - `rejectUnauthorized`: If `true` the server will reject any connection
    which is not authorized with the list of supplied CAs. This option only
    has an effect if `requestCert` is `true`. Default: `false`.

  - `NPNProtocols`: An array or `Buffer` of possible NPN protocols. (Protocols
    should be ordered by their priority).

  - `SNICallback`: A function that will be called if client supports SNI TLS
    extension. Only one argument will be passed to it: `servername`. And
    `SNICallback` should return SecureContext instance.
    (You can use `crypto.createCredentials(...).context` to get proper
    SecureContext). If `SNICallback` wasn't provided - default callback with
    high-level API will be used (see below).

  - `sessionIdContext`: A string containing a opaque identifier for session
    resumption. If `requestCert` is `true`, the default is MD5 hash value
    generated from command-line. Otherwise, the default is not provided.

Here is a simple example echo server:

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

      // This is necessary only if the client uses the self-signed certificate.
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

Or

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('server.pfx'),

      // This is necessary only if using the client certificate authentication.
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
You can test this server by connecting to it with `openssl s_client`:


    openssl s_client -connect 127.0.0.1:8000

<!--english end-->

새로운 [tls.Server](#tls.Server)를 생성한다.
`connectionListener` 아규먼트는 자동적으로 [secureConnection](#event_secureConnection_) 
이벤트의 리스너로 설정된다. 
`options` 객체는 다음과 같은 선택사항이 있다.

  - `pfx`: PFX나 PKCS12 형식으로 개인키, 인증서, 서버의 CA 인증서를 담고 있는 
    문자열이나 `Buffer`다. (`key`, `cert`, `ca` 옵셩은 상호배타적이다.)

  - `key`: PEM 형식의 서버 개인키를 담고 있는 문자열이나 `Buffer`다. (필수사항)

  - `passphrase`: 개인키나 pfx에 대한 암호문의 문자열이다.

  - `cert`: PEM 형식의 서버 인증키를 담고 있는 문자열이나 `Buffer`다. (필수사항)

  - `ca`: 신뢰할 수 있는 인증서의 문자열이나 `Buffer` 배열이다. 이 옵션을 생락하면
    각각 VeriSign처럼 잘 알려진 "루트" CA를 사용할 것이다. 연결에 권한을 부여하는 데
    이것들을 사용한다.

  - `ciphers`: 사용하거나 제외할 암호(cipher)를 설명하는 문자열이다. 자세한 형식은 
    <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>를
    참고해라.

  - `requestCert`: `true`로 지정하면 연결할 때 서버가 클라이언트한테 인증서를 요청하고
    인증서를 검증하는 데 사용할 것이다. 기본값: `false`.

  - `rejectUnauthorized`: `true`로 지정하면 제공된 CA 목록에서 인가되지 않은 
    모든 연결을 서버가 거절할 것이다. 이 옵션은 `requestCert`가 `true`일때만 유효하다.
    기본값: `false`.

  - `NPNProtocols`: 사용가능한 NPN 프로토콜의 배열이나 `Buffer`다. (프로토콜은 
    우선순위에 따라 정렬되어야 한다.)

  - `SNICallback`: 클라이언트가 SNI TLS 확장을 지원하는 경우 호출될 함수다. 이 함수는
    단 하나의 아규먼트인 `servername`만 받는다. 그리고 `SNICallback`는 SecureContext
    인스턴스를 반환해야 한다.
    (적합한 SecureContext를 얻으려면 `crypto.createCredentials(...).context`를 
    사용할 수 있다.) `SNICallback`를 제공하지 않으면 고수준 API와 함께 기본 콜백을 
    사용할 것이다.(아래를 참고해라.)

  - `sessionIdContext`: 세션 회수를 위한 불투명한 식별자를 담고 있는 문자열이다.
    `requestCert`가 `true`이면 기본값은 커맨드라인에서 생성한 MD5 해시값이다. 
    `requestCert`가 `true`가 아니면기반값은 제공하지 않는다.

간단한 에코(echo) 서버의 예제다.

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('server-key.pem'),
      cert: fs.readFileSync('server-cert.pem'),

      // This is necessary only if using the client certificate authentication.
      requestCert: true,

      // This is necessary only if the client uses the self-signed certificate.
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

      // This is necessary only if using the client certificate authentication.
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


## tls.connect(port, [host], [options], [secureConnectListener])

<!--english start-->

Creates a new client connection to the given `port` and `host`. (If `host`
defaults to `localhost`.) `options` should be an object which specifies

  - `pfx`: A string or `Buffer` containing the private key, certificate and
    CA certs of the server in PFX or PKCS12 format.

  - `key`: A string or `Buffer` containing the private key of the client in
    PEM format.

  - `passphrase`: A string of passphrase for the private key or pfx.

  - `cert`: A string or `Buffer` containing the certificate key of the client in
    PEM format.

  - `ca`: An array of strings or `Buffer`s of trusted certificates. If this is
    omitted several well known "root" CAs will be used, like VeriSign.
    These are used to authorize connections.

  - `NPNProtocols`: An array of string or `Buffer` containing supported NPN
    protocols. `Buffer` should have following format: `0x05hello0x05world`,
    where first byte is next protocol name's length. (Passing array should
    usually be much simpler: `['hello', 'world']`.)

  - `servername`: Servername for SNI (Server Name Indication) TLS extension.

  - `socket`: Establish secure connection on a given socket rather than
    creating a new socket. If this option is specified, `host` and `port`
    are ignored.  This is intended FOR INTERNAL USE ONLY.  As with all
    undocumented APIs in Node, they should not be used.

The `secureConnectListener` parameter will be added as a listener for the
['secureConnect'](#event_secureConnect_) event.

`tls.connect()` returns a [CleartextStream](#tls.CleartextStream) object.

Here is an example of a client of echo server as described previously:

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      // These are necessary only if using the client certificate authentication
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-cert.pem'),
    
      // This is necessary only if the server uses the self-signed certificate
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

Or

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

<!--english end-->

전달한 `port`와 `host`로 새로운 클라이언트 연결을 생성한다. (`host`를 지정하지 않으면
기본값은 `localhost`이다.) `options`는 지정된 객체여야 한다.

  - `pfx`: PFX나 PKCS12 형식으로 개인키, 인증서 서버의 CA 인증서를 담고 있는 문자열이나
   `Buffer`다.

  - `key`: PEM 형식으로 클라이언트의 개인키를 담고 있는 문자열이나 `Buffer`다.

  - `passphrase`: 개인키나 pfx에 대한 암호문 문자열이다.

  - `cert`: PEM 형식으로 클라이언트의 인증서 키를 담고 있는 문자열이나 `Buffer`다.

  - `ca`: 신뢰할 수 있는 인증서의 문자열이나 `Buffer` 배열이다. 이 옵션을 생락하면
    각각 VeriSign처럼 잘 알려진 "루트" CA를 사용할 것이다. 연결에 권한을 부여하는 데
    이것들을 사용한다.

  - `NPNProtocols`: 지원하는 NPN 프로토콜의 배열이나 `Buffer`다. `Buffer`는 다음의 
    형식이어야 한다. `0x05hello0x05world`, 첫 바이트는 next protocol name의 
    길이이다.(배열을 전달하는 것이 보통 훨씬 간단할 것이다: `['hello', 'world']`)

  - `servername`: SNI (Server Name Indication) TLS 확장에 대한 Servername이다.

  - `socket`: 새로운 소켓을 생성하는 대신 전달한 소켓으로 안전한 연결을 만든다.
    이 옵션을 지정하면 `host`와 `port`는 무시한다. 이 옵션은 *내부적으로 사용하기
    위해서* 만들어진 것이다. Node에서 문서화되지 않은 모든 API와 같이 사용하지 말아야
    한다.

`secureConnectListener` 파라미터는 ['secureConnect'](#event_secureConnect_) 
이벤트의 리스너로 추가할 것이다.

`tls.connect()`는 [CleartextStream](#tls.CleartextStream) 객체를 반환한다.

앞에서 설명한 에코 서버의 클라이언트 예제다.

    var tls = require('tls');
    var fs = require('fs');

    var options = {
      // These are necessary only if using the client certificate authentication
      key: fs.readFileSync('client-key.pem'),
      cert: fs.readFileSync('client-cert.pem'),
    
      // This is necessary only if the server uses the self-signed certificate
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

<!--english start-->

Creates a new secure pair object with two streams, one of which reads/writes
encrypted data, and one reads/writes cleartext data.
Generally the encrypted one is piped to/from an incoming encrypted data stream,
and the cleartext one is used as a replacement for the initial encrypted stream.

 - `credentials`: A credentials object from crypto.createCredentials( ... )

 - `isServer`: A boolean indicating whether this tls connection should be
   opened as a server or a client.

 - `requestCert`: A boolean indicating whether a server should request a
   certificate from a connecting client. Only applies to server connections.

 - `rejectUnauthorized`: A boolean indicating whether a server should
   automatically reject clients with invalid certificates. Only applies to
   servers with `requestCert` enabled.

`tls.createSecurePair()` returns a SecurePair object with
[cleartext](#tls.CleartextStream) and `encrypted` stream properties.

<!--english end-->

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

`tls.createSecurePair()`는 [cleartext](#tls.CleartextStream)와 `encrypted` 스트림 
프로퍼티로 SecurePair 객체를 반환한다.

## Class: SecurePair

<!--english start-->

Returned by tls.createSecurePair.

<!--english end-->

tls.createSecurePair가 반환하는 클래스다.

### Event: 'secure'

<!--english start-->

The event is emitted from the SecurePair once the pair has successfully
established a secure connection.

Similarly to the checking for the server 'secureConnection' event,
pair.cleartext.authorized should be checked to confirm whether the certificate
used properly authorized.

<!--english end-->

쌍이 성공적으로 안전한 연결을 수립했을 때 SecurePair가 발생시키는 이벤트이다.

서버의 'secureConnection' 이벤트를 확인하는 것과 비슷하게 pair.cleartext.authorized는 
사용된 인증서가 제대로 권한을 부여받았는 지 승인하려기 위해 확인되어야 한다.

## Class: tls.Server

<!--english start-->

This class is a subclass of `net.Server` and has the same methods on it.
Instead of accepting just raw TCP connections, this accepts encrypted
connections using TLS or SSL.

<!--english end-->

`net.Server`의 하위클래스이고 `net.Server`와 같은 메서드를 가진다.
그냥 로우(raw) TCP 연결을 받아들이는 대신 TLS나 SSL을 사용해서 암호화된 연결을
받아들인다.

### Event: 'secureConnection'

<!--english start-->

`function (cleartextStream) {}`

This event is emitted after a new connection has been successfully
handshaked. The argument is a instance of
[CleartextStream](#tls.CleartextStream). It has all the common stream methods
and events.

`cleartextStream.authorized` is a boolean value which indicates if the
client has verified by one of the supplied certificate authorities for the
server. If `cleartextStream.authorized` is false, then
`cleartextStream.authorizationError` is set to describe how authorization
failed. Implied but worth mentioning: depending on the settings of the TLS
server, you unauthorized connections may be accepted.
`cleartextStream.npnProtocol` is a string containing selected NPN protocol.
`cleartextStream.servername` is a string containing servername requested with
SNI.

<!--english end-->

`function (cleartextStream) {}`

새로운 연결이 성공적으로 핸드쉐이크를 했을 때 발생하는 이벤트이다. 아규먼트는 
[CleartextStream](#tls.CleartextStream) 인스턴스다. 이 인스턴스는 공통 스트림 메서드와 
이벤트를 모두 갖고 있다.

`cleartextStream.authorized`는 서버에 대해 제공된 인증서 권한 중 하나로 클라이언트가 
검증되었는지 나타내는 불리언 값이다. `cleartextStream.authorized`가 false이면 
`cleartextStream.authorizationError`는 어떻게 권한부여가 실패했는 지를 알려주기 위해
설정된다. TLS 서버의 설정에 따라 권한이 부여되지 않은 연결을 받아들일 수 있다는 점을 알아두어야
한다. `cleartextStream.npnProtocol`는 선택된 NPN 프로토콜을 담고 있는 문자열이다.
`cleartextStream.servername`은 SNI로 요청된 servername을 담고 있는 문자열이다.


### Event: 'clientError'

<!--english start-->

`function (exception) { }`

When a client connection emits an 'error' event before secure connection is
established - it will be forwarded here.

<!--english end-->

`function (exception) { }`

안전한 연결이 이뤄지기 전에 클라이언트 연결이 'error' 이벤트를 발생시켰을 때 발생하는 
이벤트다.


### server.listen(port, [host], [callback])

<!--english start-->

Begin accepting connections on the specified `port` and `host`.  If the
`host` is omitted, the server will accept connections directed to any
IPv4 address (`INADDR_ANY`).

This function is asynchronous. The last parameter `callback` will be called
when the server has been bound.

See `net.Server` for more information.

<!--english end-->

지정한 `port`와 `host`로 연결을 받아들이기 시작한다. `host`를 생략하면 서버는 모든 
IPv4 주소(`INADDR_ANY`)에서 직접 연결을 받아들일 것이다.

이 함수는 비동기함수이다. 마지막 파라미터인 `callback`는 서버의 바인딩이 완료되었을 때 
호출될 것이다.

더 자세한 내용은 `net.Server`를 봐라.


### server.close()

<!--english start-->

Stops the server from accepting new connections. This function is
asynchronous, the server is finally closed when the server emits a `'close'`
event.

<!--english end-->

서버가 새로운 연결을 받아들이는 것을 멈춤다. 이 함수는 비동기 함수이고 서버가 `'close'`
이벤트를 발생시켰을 때 결국 닫힌다.

### server.address()

<!--english start-->

Returns the bound address and port of the server as reported by the operating
system.
See [net.Server.address()](net.html#server.address) for more information.

<!--english end-->

운영체제에서 보고된 서버가 바인딩된 주소와 포트를 반환한다.
더 자세한 내용은 [net.Server.address()](net.html#server.address)를 봐라.

### server.addContext(hostname, credentials)

<!--english start-->

Add secure context that will be used if client request's SNI hostname is
matching passed `hostname` (wildcards can be used). `credentials` can contain
`key`, `cert` and `ca`.

<!--english end-->

클라이언트 요청의 SNI 호스트이름이 전달한 `hostname`(와일드카드도 사용할 수 있다.)와 
일치하는 경우 사용할 안전한 컨텍스트를 추가한다. `credentials`은 `key`, `cert`, `ca`를 
포함할 수 있다.

### server.maxConnections

<!--english start-->

Set this property to reject connections when the server's connection count
gets high.

<!--english end-->

서버의 연결수가 많아졌을 때 연결을 거절하려면 이 프로퍼티를 설정해라.

### server.connections

<!--english start-->

The number of concurrent connections on the server.

<!--english end-->

서버의 현재 연결 수


## Class: tls.CleartextStream

<!--english start-->

This is a stream on top of the *Encrypted* stream that makes it possible to
read/write an encrypted data as a cleartext data.

This instance implements a duplex [Stream](stream.html) interfaces.
It has all the common stream methods and events.

A ClearTextStream is the `clear` member of a SecurePair object.

<!--english end-->

평문 데이터와 같이 암호화된 데이터도 읽고 쓸 수 있도록 *암호화된* 스트림에 기반을 둔 
스트림이다.

이 인스턴스는 이중 [Stream](stream.html) 인터페이스를 구현한다.
이 인스턴스는 공통 스트림 메서드와 이벤트를 모두 가지고 있다.

ClearTextStream는 SecurePair 객체의 `clear` 멤버이다.

### Event: 'secureConnect'

<!--english start-->

This event is emitted after a new connection has been successfully handshaked. 
The listener will be called no matter if the server's certificate was
authorized or not. It is up to the user to test `cleartextStream.authorized`
to see if the server certificate was signed by one of the specified CAs.
If `cleartextStream.authorized === false` then the error can be found in
`cleartextStream.authorizationError`. Also if NPN was used - you can check
`cleartextStream.npnProtocol` for negotiated protocol.

<!--english end-->

새로운 연결이 성공적으로 핸드쉐이크를 했을 때 발생하는 이벤트이다.
리스너는 서버의 인증서가 권한 부여를 받았는 지에 상관없이 호출될 것이다. 이는 서버 인증서가 
지정한 CA중의 하나로 서명되었는지를 보려고 `cleartextStream.authorized`를 검사하는 
사용자에게 달려있다. `cleartextStream.authorized === false`인 경우 오류는 
`cleartextStream.authorizationError`에 있을 것이다. 또한 NPN이 사용되었다면 협상 
프로토콜을 위해 `cleartextStream.npnProtocol`를 확인할 수 있다.

### cleartextStream.authorized

<!--english start-->

A boolean that is `true` if the peer certificate was signed by one of the
specified CAs, otherwise `false`

<!--english end-->

피어(peer) 인증서가 지정한 CA중에 하나로 서명되었으면 `true` 그렇지 않으면 `false`인
불리언이다.

### cleartextStream.authorizationError

<!--english start-->

The reason why the peer's certificate has not been verified. This property
becomes available only when `cleartextStream.authorized === false`.

<!--english end-->

왜 피어(peer)의 인증서가 검증되지 못했는 지에 대한 내용이다. 이 프로퍼티는 
`cleartextStream.authorized === false`인 경우에만 사용할 수 있다.

### cleartextStream.getPeerCertificate()

<!--english start-->

Returns an object representing the peer's certificate. The returned object has
some properties corresponding to the field of the certificate.

Example:

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

If the peer does not provide a certificate, it returns `null` or an empty
object.

<!--english end-->

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

### cleartextStream.address()

<!--english start-->

Returns the bound address and port of the underlying socket as reported by the
operating system. Returns an object with two properties, e.g.
`{"address":"192.168.57.1", "port":62053}`

<!--english end-->

운영체제가 보고했듯이 기반하는 소켓의 바인딩된 주소와 포트를 반환한다. 다음과 같이 두 가지 
프로퍼티를 가진 객체를 반환한다. `{"address":"192.168.57.1", "port":62053}`

### cleartextStream.remoteAddress

<!--english start-->

The string representation of the remote IP address. For example,
`'74.125.127.100'` or `'2001:4860:a005::68'`.

<!--english end-->

원격 IP 주소를 나타내는 문자열이다. 예를 들면 `'74.125.127.100'`나 
`'2001:4860:a005::68'`이다.

### cleartextStream.remotePort

<!--english start-->

The numeric representation of the remote port. For example, `443`.

<!--english end-->

원격 포트르르 나태내는 숫자다. 예를 들면 `443`.
