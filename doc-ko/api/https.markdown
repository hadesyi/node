# HTTPS

    Stability: 3 - Stable

HTTPS는 TLS/SSL를 사용하는 HTTP 프로토콜이다. Node에서 HTTPS는 별도의 모듈로 
구현되었다.

## Class: https.Server

이 클래스는 `tls.Server`의 하위 클래스로 `http.Server`와 같은 이벤트를 
발생시킨다. 자세한 내용은 `http.Server`를 참고해라.

## https.createServer(options, [requestListener])

새로운 HTTPS 웹서버 객체를 반환한다. `options`은 [tls.createServer()][]와 
유사하다. `requestListener`는 `'request'` 이벤트에 자동으로 추가되는 
함수이다. 

예제:

    // curl -k https://localhost:8000/
    var https = require('https');
    var fs = require('fs');

    var options = {
      key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
      cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
    };

    https.createServer(options, function (req, res) {
      res.writeHead(200);
      res.end("hello world\n");
    }).listen(8000);

또는

    var https = require('https');
    var fs = require('fs');

    var options = {
      pfx: fs.readFileSync('server.pfx')
    };

    https.createServer(options, function (req, res) {
      res.writeHead(200);
      res.end("hello world\n");
    }).listen(8000);

## https.request(options, callback)

안전한 웹서버로의 요청을 생성한다.
Makes a request to a secure web server.
[http.request()][]의 모든 옵션이 유효하다.

예제:

    var https = require('https');

    var options = {
      host: 'encrypted.google.com',
      port: 443,
      path: '/',
      method: 'GET'
    };

    var req = https.request(options, function(res) {
      console.log("statusCode: ", res.statusCode);
      console.log("headers: ", res.headers);

      res.on('data', function(d) {
        process.stdout.write(d);
      });
    });
    req.end();

    req.on('error', function(e) {
      console.error(e);
    });

options 아규먼트는 다음과 같다.

- host: 요청을 보낼 호스트이 IP나 도메인. 기본값은 `'localhost'`이다.
- port: 요청을 보낼 호스트의 포트. 기본값은 443이다.
- path: 요청 경로. 기본값은 `'/'`이다.
- method: HTTP 요청 메서드. 기본값은 `'GET'`이다.

- `host`: 요청을 보낼 서버의 도메인명이나 IP 주소다. 기본값은 `'localhost'`이다.
- `hostname`: `url.parse()`를 지원하기 위해 `host`보다 `hostname`를 선호한다.
- `port`: 원격서버의 포트. 기본값은 443이다.
- `method`: HTTP 요청 메서드를 지정하는 문자열이다. 기본값은 `'GET'`이다.
- `path`: 요청 경로. 기본값은 `'/'`이다. 필요하다면 쿼리스트링도 포함해야 한다.
  예시. `'/index.html?page=12'`
- `headers`: 요청 헤더를 담고 있는 객체다.
- `auth`: 기본 인증. 예글 들면 인증헤더를 계산하는 `'user:password'`
- `agent`: [Agent][] 동작을 제어한다. Agent를 사용했을 때
  요청은 기본적으로 `Connection: keep-alive`가 될 것이다. 다음의 값들이 가능하다.
 - `undefined` (기본값): 해당 호스트와 포트에 [globalAgent][]를 사용한다.
 - `Agent` 객체: `Agent`에 명시적으로 전달된 객체를 사용한다.
 - `false`: Agent를 연결 풀링에 참가시키지 않는다. 기본적으로 요청은 
   `Connection: close`가 된다.

[tls.connect()][]의 다음 옵션들도 지정할 수 있다.
하지만 [globalAgent][]는 경고없이 이러한 값들을 무시한다.

- `pfx`: SSL에 사용할 인증서, 개인키, CA 인증서. 기본값은 `null`이다.
- `key`: SSL에 사용할 개인키. 기본값은 `null`이다.
- `passphrase`: 개인키나 pfx에 대한 암호문 문자열. 기본값은 `null`이다.
- `cert`: 사용할 공개 x509 인증서. 기본값은 `null`이다.
- `ca`: 원격 호스트에 대해 확인할 권한 인증이나 권한 인증의 배열이다.
- `ciphers`: 사용하거나 배제할 암호문을 나타내는 문자열. 자세한 형식은
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT> 를
  참고해라.
- `rejectUnauthorized`: 이 값이 `true`이면 서버 인증서를 제공된 CA 리스트로 
  검증한다. 검증이 실패했을 때 `'error'` 이벤트가 발생한다. 검증은 HTTP 요청을 
  보내기 *전* 연결단계에서 이뤄진다. 기본값은 `false`이다.

이러한 옵션들을 지정하려면 커스텀 `Agent`를 사용해라.

예제:

    var options = {
      host: 'encrypted.google.com',
      port: 443,
      path: '/',
      method: 'GET',
      key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
      cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem')
    };
    options.agent = new https.Agent(options);

    var req = https.request(options, function(res) {
      ...
    }

또는 `Agent`를 사용하지 마라.

예제:

    var options = {
      host: 'encrypted.google.com',
      port: 443,
      path: '/',
      method: 'GET',
      key: fs.readFileSync('test/fixtures/keys/agent2-key.pem'),
      cert: fs.readFileSync('test/fixtures/keys/agent2-cert.pem'),
      agent: false
    };

    var req = https.request(options, function(res) {
      ...
    }

## https.get(options, callback)

`http.get()`와 같지만 HTTPS다.

예제:

    var https = require('https');

    https.get({ host: 'encrypted.google.com', path: '/' }, function(res) {
      console.log("statusCode: ", res.statusCode);
      console.log("headers: ", res.headers);

      res.on('data', function(d) {
        process.stdout.write(d);
      });

    }).on('error', function(e) {
      console.error(e);
    });


## Class: https.Agent

[http.Agent][]와 유사한 HTTPS의 Agent 객체.
자세한 내용은 [https.request()][]를 참고해라.


## https.globalAgent

모든 HTTPS 클라이언트 요청에 대한 [https.Agent][]의 전역 인스턴스다.

[Agent]: #https_class_https_agent
[globalAgent]: #https_https_globalagent
[http.Agent]: http.html#http_class_http_agent
[http.request()]: http.html#http_http_request_options_callback
[https.Agent]: #https_class_https_agent
[tls.connect()]: tls.html#tls_tls_connect_options_secureconnectlistener
[tls.createServer()]: tls.html#tls_tls_createserver_options_secureconnectionlistener
