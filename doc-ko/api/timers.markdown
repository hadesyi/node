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

인터벌을 트리거되는 것을 멈춘다.
