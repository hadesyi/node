# console

    Stability: 4 - API Frozen

* {Object}

<!--type=global-->

stdout와 stderr에 출력하기 위해 사용한다. 대부분의 웹 브라우저가 제공하는 console 
객체의 기능과 유사하게 stdout이나 stderr로 출력한다.

## console.log([data], [...])

새로운 라인으로 stdout에 출력한다. 이 함수는 `printf()`와 같은 방식으로 여러 아규먼트를
받는다. 예제:

    console.log('count: %d', count);

첫 문자열에 포매팅 객체가 없으면 각 아규먼트에 `util.inspect`를 사용한다.
더 자세한 내용은 [util.format()][]를 봐라.

## console.info([data], [...])

`console.log`와 동일하다.

## console.error([data], [...])

`console.log`와 같지만 stderr에 출력한다.

## console.warn([data], [...])

`console.error`와 같다.

## console.dir(obj)

`obj`에 `util.inspect`를 사용하고 결과 문자열을 stdout에 출력한다.

## console.time(label)

시간을 마킹한다.

## console.timeEnd(label)

타이머를 종료하고 결과를 기록한다. 예제:

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');

## console.trace(label)

현지 위치의 stderr에 스택트레이스를 출력한다.

## console.assert(expression, [message])

`expression`가 `false`이면 `message`로 AssertionError를 던지는 [assert.ok()][]와 
같다.

[assert.ok()]: assert.html#assert_assert_value_message_assert_ok_value_message
[util.format()]: util.html#util_util_format_format
