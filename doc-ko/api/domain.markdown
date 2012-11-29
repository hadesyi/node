# Domain

    Stability: 1 - Experimental

도메인은 하나의 그룹으로 여러 가지 다른 IO 작업을 다룰 수 있는 방법을 제공한다.
도메인에 등록된 이벤트 이미터나 콜백 등에서 `error` 이벤트가 발생하거나 오류를
던졌을 때 `process.on('uncaughtException')`에서 오류의 컨텍스트를 읽어버리거나
오류 코드로 프로그램이 종료되는 대신 도메인 객체가 이를 인지할 수 있다.

이 기능은 Node 0.8버전에서 새롭게 추가된 기능이다. 처음 도입된 것이고 차후 버전에서
꽤 많은 변경이 있을 것이다. 사용해보고 개발팀에 피드백을 주기 바란다.

아직 실험적인 기능이기 때문에 최소 한번이라도 `domain` 모듈을 로드하지 않는한
도메인 기능은 사용하지 않도록 되어 있다. 기본적으로 도메인은 생성되거나 등록되지
않는다. 이는 현재 프로그램에 좋지 않은 영향을 주지 않으려는 설계적인 의도이다.
차후 Node.js 버전에서는 기본적으로 활성화되기를 기대한다.

## Additions to Error objects

<!-- type=misc -->

도데인으로 오류객체가 전달될 때 몇가지 추가적인 필드가 추가된다.

* `error.domain` 최초에 오류를 다루는 도메인.
* `error.domain_emitter` 오류 객체와 함게 'error' 이벤트를 발생시킨 이벤트 이미터. 
* `error.domain_bound` 도메인에 바인딩된 콜백함수로 첫 아규먼트로 오류객체를
  전달한다.
* `error.domain_thrown` 오류가 던져졌는지 오류 이벤트가 발생했는지 바인딩된
  콜백함수로 전달되었는지를 나타내는 불리언 값.

## Implicit Binding

<!--type=misc-->

도메인을 사용하면 새로 생성되는 모든 EventEmitter 객체(스트림 객체, 요청, 응답등을
포함해서)는 생성되면서 암묵적으로 활성화된 도메인에 바인딩 될 것이다.

게다가 저수준 이벤트 루프 요청(fs.open나 콜백을 받는 메서드같은)에 전달한 콜백은
자동적으로 활성화된 도메인에 바인딩된다. 이 콜백이 예외를 던지면 도메인이
이 오류를 잡아낸다.

과도한 메모리 사용을 막기 위해 도메인 객체 자체는 활성화된 도메인의 자식에
암묵적으로 추가되지 않는다. 도메인 객체가 있다면 요청 객체와 응답객체가
가비지 컬렉트되지 않도록 하는 것은 아주 쉽다.

부모 도메인의 자식처럼 중첩된 도메인 객체가 *필요하다면* 반드시 명시적으로
추가한 뒤 나중에 정리해야 한다.

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
        res.on('close', function() {
          // 이 도메인에 추가한 다른 것들도 강제적으로 종료한다.
          reqd.dispose();
        });
      } catch (er) {
        console.error('Error sending 500', er, req.url);
        // 최선을 다했다. 남아있는 모든 것을 정리한다.
        reqd.dispose();
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
