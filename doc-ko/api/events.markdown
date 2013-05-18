# Events

    Stability: 4 - API Frozen

<!--type=module-->

노드의 많은 객체들은 이벤트를 발생시킨다. `net.Server`는 서버에 연결이 생길 때마다
이벤트를 발생시키고 `fs.readStream`는 파일이 열렸을 때 이벤트를 발생시킨다.
이벤트를 발생시키는 모든 객체들은 `events.EventEmitter`의 인스턴스다.
`require("events");`를 사용해서 이 모듈에 접근할 수 있다.

보통 이벤트명은 카멜케이스의 문자열로 표시하지만 이벤트명에 어떤 제약사항은 없기 때문에
어떤 문자열이라도 사용할 수 있다.

이벤트가 발생할 때 실행할 함수를 객체에 연결할 수 있다. 이러한 함수들을
_리스너(listener)_라고 부른다. 리스너 함수 내부에서 `this`는 리스너가 연결된
`EventEmitter`를 참조한다.


## Class: events.EventEmitter

EventEmitter 클래스에 접근하려면 `require('events').EventEmitter`를 사용한다.

`EventEmitter` 인스턴스에서 오류가 발생하면 보통 `'error'` 이벤트를 발생시킨다.
노드는 오류 이벤트를 특별한 경우로 대한다. 오류 이벤트에 대한 리스너가 등록되어 있지 않은
경우 기본 동작은 스택 트레이스를 출력하고 프로그램을 종료하는 것이다.

모든 이벤트이미터는 새로운 리스너를 추가되었을 때 `'newListener'` 이벤트를 발생시키고
리스너가 제거되었을 때 `'removeListener'` 이벤트를 발생시킨다.

### emitter.addListener(event, listener)
### emitter.on(event, listener)

지정한 event에 대한 리스너 배열의 끝에 listener를 추가한다.

    server.on('connection', function (stream) {
      console.log('someone connected!');
    });

이미터를 반환하므로 호출을 체인으로 연결할 수 있다.

### emitter.once(event, listener)

event에 **일회성** listener를 추가한다. 이 리스너는
이벤트가 다음 번에 발생했을 때 딱 한번만 실행된 후 제거된다.

    server.once('connection', function (stream) {
      console.log('Ah, we have our first user!');
    });

이미터를 반환하므로 호출을 체인으로 연결할 수 있다.

### emitter.removeListener(event, listener)

지정한 event에 대한 리스너 배열에서 listener를 제거한다.
**주의**: 리스너보다 뒤쪽에서 리스너 배열의 배열인덱스를 수정해라.

    var callback = function(stream) {
      console.log('someone connected!');
    };
    server.on('connection', callback);
    // ...
    server.removeListener('connection', callback);

이미터를 반환하므로 호출을 체인으로 연결할 수 있다.

### emitter.removeAllListeners([event])

event를 지정하지 않으면 모든 리스너를 제거하고 event를 지정하면 지정한 이벤트의
모든 리스너를 제거한다.

이미터를 반환하므로 호출을 체인으로 연결할 수 있다.

### emitter.setMaxListeners(n)

기본적으로 EventEmitter는 특정 이벤트에 10개 이상의 리스너가 추가되면 경고메시지를
출력할 것이다. 이 경고메시지는 메모리 누출을 찾는데 도움을 주는 유용한 기본기능이다.
명백히 모든 이미터가 10개로 제한되어야 하는 것은 아닐 것이다. 이 함수로 이 리스너 제한을
늘릴 수 있다. 0을 지정하면 무한대로 등록할 수 있다.


### emitter.listeners(event)

지정한 이벤트의 리스너 배열을 리턴한다.

    server.on('connection', function (stream) {
      console.log('someone connected!');
    });
    console.log(util.inspect(server.listeners('connection'))); // [ [Function] ]


### emitter.emit(event, [arg1], [arg2], [...])

전달한 아규먼트의 순서대로 각 리스너를 실행한다.

이벤트가 리스너를 가지고 있으면 `true`을 반환하고 가지고 있지 않으면 `false`를 반환한다.

### Class Method: EventEmitter.listenerCount(emitter, event)

해당 이벤트의 리스터 갯수를 반환한다.


### Event: 'newListener'

* `event` {String} 이벤트명
* `listener` {Function} 이벤트 핸들러 함수

이 이벤트를 새로운 리스너가 어딘가에 추가될 때마다 발생한다.


### Event: 'removeListener'

* `event` {String} The event name
* `listener` {Function} The event handler function

이 이벤트는 리스너를 제거할 때마다 발생한다.
