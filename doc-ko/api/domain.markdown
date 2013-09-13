# Domain

    Stability: 2 - Unstable

도메인은 하나의 그룹으로 여러 가지 다른 IO 작업을 다룰 수 있는 방법을 제공한다.
도메인에 등록된 이벤트 이미터나 콜백 등에서 `error` 이벤트가 발생하거나 오류를
던졌을 때 `process.on('uncaughtException')`에서 오류의 컨텍스트를 읽어버리거나
오류 코드로 프로그램이 즉시 종료되는 대신 도메인 객체가 이를 인지할 수 있다.

## Warning: Don't Ignore Errors!

<!-- type=misc -->

도메인 오류 핸들러는 오류 발생시 프로세스를 종료하는 것을 대체하지는 않는다.

JavaScript에서 `throw` 동작방식의 기본적인 특성으로 인해 참조를 누출하거나
undefined같은 류의 깨지기 쉬운 상태를 생성하지 않은 채로 안전하게 "벗어난 곳이 어디인지를 찾아내는"
방법이 거의 없다.

오류가 던져졌을 때 가장 안정한 반응은 프로세스를 종료하는 것이다. 물론 일반적인 웹서버에서
다수의 연결을 얼어놓고 있을 것인데 어디선가 오류가 발생해서 갑자기 종료시켜버리는 것은
합당하지는 않다.

더 좋은 접근방법은 오류를 발생시킨 요청에 오류 응답을 보내면서 다른 요청들은 정상적으로
종료해고 해당 워커는 새로운 요청을 받아들이지 않도록 하는 것이다.

이 접근에서 워커가 오류를 만났을 때 마스터 프로세스가 새로운 워커를 만들 수 있으므로 `domain`은
클러스터 모듈과 친하다. 다중 머신으로 확장하는 node 프로그램에서 프록시나 서비스 등록을 종료하는 것은
실패를 관리하고 적절하게 대응할 수 있다.

예를 들어 다음은 좋은 생각이 아니다.

```javascript
// XXX WARNING!  BAD IDEA!

var d = require('domain').create();
d.on('error', function(er) {
  // This is no better than process.on('uncaughtException')!
  // 오류는 프로세스를 깨뜨리지는 않지만 오류가 하는하는 일은 더 좋지 않다!
  // 갑작스러운 프로세스의 재시작을 막더라도 이런 일이 발생하면 리소스를 노출하게 된다.
  // 이는 process.on('uncaughtException')보다 나을게 없다!
  console.log('error, but oh well', er.message);
});
d.run(function() {
  require('http').createServer(function(req, res) {
    handleRequest(req, res);
  }).listen(PORT);
});
```

도메인의 컨텍스트와 프로그램을 다중 워커 프로세스로 분리하는 강력함을 사용해서
적절하게 대응하고 훨신 더 안전하게 오류를 다룰 수 있다.

```javascript
// 훨씬 좋다!

var cluster = require('cluster');
var PORT = +process.env.PORT || 1337;

if (cluster.isMaster) {
  // 실환경에서는 2개 이상의 워커를 사용할 것이고 마스터와 워커를 같은 파일에
  // 두지 않을 것이다.
  //
  // 물론 DoS 공격이나 다른 공격행위를 차단하기 위해서 필요한 커스텀 로직을 구현하고
  // 로깅을 더 깔끔하게 할 수도 있다.
  //
  // 클러스터 문서의 옵션을 찾고해라.
  //
  // 중요한 점은 의도치않은 오류의 복구능력을 높히면서 마스터가 하는 일은
  // 아주 적다는 것이다.

  cluster.fork();
  cluster.fork();

  cluster.on('disconnect', function(worker) {
    console.error('disconnect!');
    cluster.fork();
  });

} else {
  // 워커
  //
  // 버그를 넣을 곳이다!

  var domain = require('domain');

  // 요청을 처리하는 워커 프로세스를 사용하는 자세한 방법은 클러스터 문서를 참고해라.
  // 동작하는 방법이나 경고 등등

  var server = require('http').createServer(function(req, res) {
    var d = domain.create();
    d.on('error', function(er) {
      console.error('error', er.stack);

      // Note: 위험지역이다.
      // 당연히 원치않은 일이 일어난다.
      // 이제 어떤 일이든 발생할 수 있다! 조심해라!

      try {
        // 30초 내에 종료되었는지 확인한다
        var killtimer = setTimeout(function() {
          process.exit(1);
        }, 30000);
        // 하지만 이를 위해 프로세스를 열어둔 채 유지하지 말아라!
        killtimer.unref();

        // 새로운 요청을 받아들이는 것을 멈춘다.
        server.close();

        // 워커가 죽은 것을 마스터가 알도록 한다. 이는 클러스터 마스터에 'disconnect'를
        // 발생시키고 마스터가 새로운 워커를 포크할 것이다.
        cluster.worker.disconnect();

        // 문제을 일으킨 요청에 오류전송을 시도한다.
        res.statusCode = 500;
        res.setHeader('content-type', 'text/plain');
        res.end('Oops, there was a problem!\n');
      } catch (er2) {
        // 여기서 할 수 있는 일은 많지 않다.
        console.error('Error sending 500!', er2.stack);
      }
    });

    // req와 res는 이 도메인이 존재하기 전에 생성되므로 명시적으로 req와 res를
    // 추가해야 한다. 아래의 암시적/명시적 바인딩의 설명을 참고해라.
    d.add(req);
    d.add(res);

    // 이제 도메인에서 핸들러 함수를 실행한다.
    d.run(function() {
      handleRequest(req, res);
    });
  });
  server.listen(PORT);
}

// 이 부분은 중요하지는 않다.그냥 라우팅 예제일 뿐이다.
// 세련된 어플리케이션 로직을 여기에 둘 수 있다.
function handleRequest(req, res) {
  switch(req.url) {
    case '/error':
      // 비동기 작업을 수행하고...
      setTimeout(function() {
        // Whoops!
        flerb.bark();
      });
      break;
    default:
      res.end('ok');
  }
}
```

## Additions to Error objects

<!-- type=misc -->

도데인으로 오류객체가 전달될 때 몇가지 추가적인 필드가 추가된다.

* `error.domain` 최초에 오류를 다루는 도메인.
* `error.domainEmitter` 오류 객체와 함게 'error' 이벤트를 발생시킨 이벤트 이미터.
* `error.domainBound` 도메인에 바인딩된 콜백함수로 첫 아규먼트로 오류객체를
  전달한다.
* `error.domainThrown` 오류가 던져졌는지 오류 이벤트가 발생했는지 바인딩된
  콜백함수로 전달되었는지를 나타내는 불리언 값.

## Implicit Binding

<!--type=misc-->

도메인을 사용하면 **새로 생성되는** 모든 EventEmitter 객체(스트림 객체, 요청, 응답등을
포함해서)는 생성되면서 암묵적으로 활성화된 도메인에 바인딩 될 것이다.

게다가 저수준 이벤트 루프 요청(fs.open나 콜백을 받는 메서드같은)에 전달한 콜백은
자동적으로 활성화된 도메인에 바인딩된다. 이 콜백이 예외를 던지면 도메인이
이 오류를 잡아낸다.

과도한 메모리 사용을 막기 위해 도메인 객체 자체는 활성화된 도메인의 자식에
암묵적으로 추가되지 않는다. 도메인 객체가 있다면 요청 객체와 응답객체가
가비지 컬렉트되지 않도록 하는 것은 아주 쉽다.

부모 도메인의 자식처럼 중첩된 도메인 객체가 *필요하다면* 반드시 명시적으로
추가해야 한다.

암묵적인 바인딩은 던져진 오류나 `'error'` 이벤트를 도메인의 `error` 이벤트로
보내지만 도메인에 EventEmitter를 등록하지는 않기 때문에 `domain.dispose()`를
해도 EventEmitter를 종료하지 않을 것이다.
암묵적인 바인딩은 던져진 오류나 `'error'` 이벤트만 처리한다.

## Explicit Binding

<!--type=misc-->

때때로 사용중인 도메인을 특정 이벤트 이미터에서 사용하지 않아야 하는 경우가 있다.
또는 이벤트 이미터가 한 도메인의 컨텍스트에서 생성되었지만 다른 도메인에 바인딩되어야
하는 경우가 있다.

예를 들면 HTTP 서버에서 사용 중인 도메인이 있지만 각 요청마다 다른 도메인을
사용하길 원할 수 있다.

이는 명시적으로 바인딩해서 할 수 있다.

예를 들면:

```
// 서버에 대한 최상위 도메인을 생성한다
var serverDomain = domain.create();

serverDomain.run(function() {
// 서버는 serverDomain의 범위내에서 생성되었다
  http.createServer(function(req, res) {
    // req와 res도 serverDomain의 범위내에서 생성되었지만
    // 요청마다 다른 도메인을 사용하길 원한다.
    // 먼저 도메인을 생성하고 req와 res를 도메인에 추가한다
    var reqd = domain.create();
    reqd.add(req);
    reqd.add(res);
    reqd.on('error', function(er) {
      console.error('Error', er, req.url);
      try {
        res.writeHead(500);
        res.end('Error occurred, sorry.');
      } catch (er) {
        console.error('Error sending 500', er, req.url);
      }
    });
  }).listen(1337);
});
```

## domain.create()

* return: {Domain}

새로운 Domain 객체를 반환한다.

## Class: Domain

Domain 클래스는 오류와 잡지 못한 예외를 활성화된 도메인 객체로 보내는 기능을
은닉한다.

Domain은 [EventEmitter][]의 자식 클래스이다. 도메인이 잡은 오류를 다루려면
도메인의 `error` 이벤트에 리스너를 추가해라.

### domain.run(fn)

* `fn` {Function}

암묵적으로 모든 이벤트 이미터, 타이머, 해당 컨텍스트에서 생성된 저수준 요청을
바인딩한 도메인의 컨텍스트에서 제공된 함수를 실행한다

이는 도메인을 사용하는 가장 기본적인 방법이다.

예제:

```
var d = domain.create();
d.on('error', function(er) {
  console.error('Caught error!', er);
});
d.run(function() {
  process.nextTick(function() {
    setTimeout(function() { // 여러 비동기의 작업들을 시뮤레이트한다
      fs.open('non-existent file', 'r', function(er, fd) {
        if (er) throw er;
        // 처리중...
      });
    }, 100);
  });
});
```

이 예제에서 프로그램이 멈추는 대신에 `d.on('error')` 핸들러가 실행될 것이다.

### domain.members

* {Array}

도메인에 명시적으로 추가한 타이머와 이벤트 이미터의 배열

### domain.add(emitter)

* `emitter` {EventEmitter | Timer} 도메인에 추가할 이미터나 타이머

명시적으로 이미터를 도메인에 추가한다. 이미터에서 호출된 이벤트 핸들러가 오류를
던졌거나 이미터에서 `error` 이벤트가 발생했을 때 암묵적으로 바인딩한 것과 마찬가지로
도메인의 `error` 이벤트로 전달된다.

이 함수는 `setInterval`와 `setTimeout`가 반환하는 타이머에서도 동작한다.
타이머의 콜백함수가 예외를 던지면 도메인의 'error' 핸들러가 잡는다.

Timer나 EventEmitter가 도메인에 이미 바인딩되어 있으면 이전에 바인딩되어 있던
도에인에서는 제거되고 이 도메인에 다시 바인딩된다.

### domain.remove(emitter)

* `emitter` {EventEmitter | Timer} 도메인에서 제거할 이미터나 타이머

`domain.add(emitter)`과는 반대로 지정한 이미터를 도메인에서 제거한다.

### domain.bind(callback)

* `callback` {Function} 콜백 함수
* return: {Function} 바인딩된 함수

전달한 콜백함수를 감싸고 있는 랩퍼 함수를 반환한다. 반환된 함수를 호출했을 때
던져지는 모든 오류는 도메인의 `error` 이벤트로 전달된다.

#### Example

    var d = domain.create();

    function readSomeFile(filename, cb) {
      fs.readFile(filename, 'utf8', d.bind(function(er, data) {
        // 여기서 오류를 던지면 도메인으로 전달된다.
        return cb(er, data ? JSON.parse(data) : null);
      }));
    }

    d.on('error', function(er) {
      // 어디선가 오류가 발생함.
      // 여기서 오류를 던지면 프로그램은 보통의 줄번호와
      // 스택 메시지와 함께 종료된다.
    });

### domain.intercept(callback)

* `callback` {Function} 콜백 함수
* return: {Function} 가로챈 함수

이 함수는 `domain.bind(callback)`와 대부분 같지만 던져진 오류를 잡기 위해 함수의
첫 아규먼트로 보낸 `Error`객체를 가로챈다

이 방법을 통해 일반적인 `if (er) return callback(er);` 패턴을 하나의 오류
핸들러로 바꿀 수 있다.

#### Example

    var d = domain.create();

    function readSomeFile(filename, cb) {
      fs.readFile(filename, 'utf8', d.intercept(function(data) {
        // 첫 아규먼트를 'Error' 라고 가정하지만 도메인이
        // 가로챘기 때문에 첫 아규먼트는 콜백에
        // 전달되지 않는다

        // 여기서 오류를 던지면 도메인으로 전달되기 때문에
        // 프로그램 곳곳에 반복해서 오류 핸들링 로직을 작성하는 대신에
        // 도메인의 'error' 이벤트로 오류 핸들링 로직을 옮길 수 있다
        return cb(null, JSON.parse(data));
      }));
    }

    d.on('error', function(er) {
      // 어디선가 오류가 발생함.
      // 여기서 오류를 던지면 프로그램은 보통의 줄번호와
      // 스택 메시지와 함께 종료된다.
    });

### domain.enter()

`enter` 메서드는 활성화된 도메인을 설정하기 위해서 `run`, `bind`, `intercept` 메서드가
사용하는 배관시설이라고 볼 수 있다. `enter`는 `domain.active`와 `process.domain`를
도메인에 설정하고 암묵적으로 도메인모듈이 관리하는 도메인 스택에 도메인을 추가한다.(도메인 스텍의
자세한 내용은 `domain.exit()`을 봐라.) `enter`를 호출하면 도메인에 바인딩된 비동기 호출과
I/O 작업 체인의 시작부분으로 경계를 설정한다.

`enter` 호출은 활성화된 도메인만 변경하고 도메인 자체는 바꾸지 않는다. 하나의 도메인에서
`enter`와 `exit`를 여러 번 호출할 수 있다.

`enter`가 호출된 도메인이 이미 폐기되었으면 `enter`는 도메인을 설정하지 않고 끝날 것이다.

### domain.exit()

`exit` 메서드는 현재 도메인을 종료하고 도메인 스텍에서 현재 도메인을 제거한다. 비동기 호출의
다른 체인의 컨텍스트로 바꿀 때마다 현재 도메인이 종료되었음을 보장하는 것은 중요하다. `exit`
호출은 도메인에 바인딩된 비동기 호출과 I/O 작업의 체인의 끝이나 체인을 중단하는 것으로 경계를
설정한다.

현재 실행 컨텍스트에 여러 도메인이 중첩되어 있다면 `exit`는 해당 도메인내에 중첩된 모든
도메인을 종료할 것이다.

`exit`를 호출하면 활성화된 도메인만 변경하고 도메인 자체는 변경하지 않는다. 하나의 도메인에서
`enter`와 `exit`를 여러 번 호출할 수 있다.

`exit`가 호출된 도메인이 이미 폐기되었으면 `exit`는 도메인을 종료하지 않고 끝날 것이다.

### domain.dispose()

dispose 함수는 도메인을 파괴하고 도메인과 연관된 모든 IO를 정리하려고 최선을
다한다. 스트림은 중지되고 종료되고 닫힌 뒤 파괴된다. 타이머는 정리된다.
명시적으로 바인딩된 콜백은 더이상 호출하지 않는다. 여기서 발생한 모든 오류 이벤트를
무시한다.

`dispose` 호출의 의도는 보통 Domain 컨텍스트의 핵심 부분이 오류 상태라는 것을
발견했을 때 연쇄적인 오류를 막기 위한 것이다.

도메인이 처분되었을 때 `dispose` 이벤트가 발생할 것이다.

IO는 여전히 수행될 것이다. 하지만 도메인이 처분되고 나면 도메인의 이미터에서 발생하는
추가적인 오류는 무시할 것이다. 그래서 남아있는 작업이 진행중이더라도
Node.js는 그러한 작업과 더이상 통신하지 않을 것이다.

[EventEmitter]: events.html#events_class_events_eventemitter
