# Timers

    Stability: 5 - Locked

All of the timer functions are globals.  You do not need to `require()`
this module in order to use them.

    안정성: 5 - Locked

timer 함수는 모두 전역객체이다. timer를 사용하기 위해 이 모듈을 `require()`할
필요가 없다.

## setTimeout(callback, delay, [arg], [...])

To schedule execution of a one-time `callback` after `delay` milliseconds. Returns a
`timeoutId` for possible use with `clearTimeout()`. Optionally you can
also pass arguments to the callback.

It is important to note that your callback will probably not be called in exactly
`delay` milliseconds - Node.js makes no guarantees about the exact timing of when
the callback will fire, nor of the ordering things will fire in. The callback will
be called as close as possible to the time specified.

`delay` 밀리초 후에 `callback`을 한번만 실행하도록 스케쥴링한다. `clearTimeout()`과 사용할 
수 있도록 `timeoutId`를 리턴한다. 선택적으로 콜백에 아규먼트를 전달할 수 있다.

콜백이 정확한 `delay` 밀리초에 실행되지 않을 수도 있다는 점은 중요하다. - Node.js는 콜백이
정확한 타이밍에 실행된다는 것을 보장하지 않고 순차적으로 실행된다는 것도 보장하지 않는다. 콜백은
지정한 시간과 가능한한 가깝게 호출할 것이다.

## clearTimeout(timeoutId)

Prevents a timeout from triggering.

타임머가 트리거되는 것을 막는다.

## setInterval(callback, delay, [arg], [...])

To schedule the repeated execution of `callback` every `delay` milliseconds.
Returns a `intervalId` for possible use with `clearInterval()`. Optionally
you can also pass arguments to the callback.

`delay` 밀리초마다 `callback` 실행을 반복하도록 스케쥴링한다. `clearInterval()`에서
사용할 수 있도록 `intervalId`를 리턴한다. 선택적으로 콜백에 아규먼트를 전달할 수도 있다.

## clearInterval(intervalId)

Stops a interval from triggering.

인터벌을 트리거되는 것을 멈춘다.
