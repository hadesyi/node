# Timers

    Stability: 5 - Locked

timer 함수는 모두 전역객체이다. timer를 사용하기 위해 이 모듈을 `require()`할
필요가 없다.

## setTimeout(callback, delay, [arg], [...])

`delay` 밀리초 후에 `callback`을 한번만 실행하도록 스케쥴링한다. `clearTimeout()`과 사용할
수 있도록 `timeoutId`를 리턴한다. 선택적으로 콜백에 아규먼트를 전달할 수 있다.

콜백이 정확한 `delay` 밀리초에 실행되지 않을 수도 있다는 점은 중요하다. - Node.js는 콜백이
정확한 타이밍에 실행된다는 것을 보장하지 않고 순차적으로 실행된다는 것도 보장하지 않는다. 콜백은
지정한 시간과 가능한한 가깝게 호출할 것이다.

## clearTimeout(timeoutId)

타임머가 트리거되는 것을 막는다.

## setInterval(callback, delay, [arg], [...])

`delay` 밀리초마다 `callback` 실행을 반복하도록 스케쥴링한다. `clearInterval()`에서
사용할 수 있도록 `intervalId`를 리턴한다. 선택적으로 콜백에 아규먼트를 전달할 수도 있다.

## clearInterval(intervalId)

인터벌의 실행을 멈춘다.

## unref()

`setTimeout`와 `setInterval`이 반환한 불투명한(opaque) 값도 활성화된 타이머이기는 하지만
이 타이머가 이벤트 루프에서 유일하게 남은 것일 경우에는 프로그램이 동작하도록 유지하지는 못하는 타이머를
생성할 수 있도록 할 `timer.unref()` 메서드를 가진다. 타이머에 이미 `unref`를 호출했다면
`unref`를 다시 호출해도 아무런 영향이 없을 것이다.

`setTimeout`의 경우 `unref`하면 이벤트 루프를 깨울 분리된 타이머를 생성한다. 이 타이머를
너무 많이 생성하면 이벤트 루프 성능에 역효과를 줄 수 있다. -- 잘 사용해라.

## ref()

이전에 `unref()`된 타이머가 있다면 타이머가 가진 프로그램을 열도록 명시적으로 요청하는 `ref()`를
호출할 수 있다. 타이머에 이미 `ref`를 호출했다면 `ref`를 다시 호출해도 아무런 영향이 없을 것이다.

## setImmediate(callback, [arg], [...])

I/O 이벤트 콜백후와 `setTimeout`와 `setInterval` 이전에 `callback`을 "즉시" 실행하도록
스케쥴링한다. `clearImmediate()`와 사용할 수 있는 `immediateId`를 반환한다. 선택적으로
콜백에 인자를 전달할 수도 있다.

Immediate는 생성한 순서대로 큐에 들어가고 루프 이터레이션마나 하나씩 큐에서 빠진다. 이 부분이
이터레이션마다 `process.maxTickDepth` 큐에 있는 콜백을 실행하는 `process.nextTick`와
다른 점이다. `setImmediate`는 큐에 있는 콜백을 실행한 후에 I/O가 실행되도록 보장하기 위해 이벤트
루프에 양보할 것이다. 실행하는 동안 순서가 유지되지만 다른 I/O 이벤트는 스케쥴된 두
immediate 콜백사이에서 실행될 것이다.

## clearImmediate(immediateId)

immediate의 실행을 멈춘다.
