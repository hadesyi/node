# Assert

<!--english start-->

    Stability: 5 - Locked

This module is used for writing unit tests for your applications, you can
access it with `require('assert')`.

<!--english end-->

    안정성: 5 - Locked

이 모듈을 어플리케이션에서 유닛 테스트를 작성하는데 사용하고 `require('assert')`로 
이 모듈을 사용할 수 있다.

## assert.fail(actual, expected, message, operator)

<!--english start-->

Throws an exception that displays the values for `actual` and `expected` separated by the provided operator.

<!--english end-->

`actual`과 `expected`의 값을 보여주는 예외를 던진다. 
operator는 두 값을 무엇으로 비교했는지 표시하는 것이다.
(역주, `operator`는 결국 Error.captureStackTrace로 넘겨지는데 무슨 역활인지 이해할 수 없음. 하지만 assert.js의 코드를 보면 모두 스트링이다. 비교하는데 사용한 함수나 Operator 이름임)

## assert(value, message), assert.ok(value, [message])

<!--english start-->

Tests if value is a `true` value, it is equivalent to `assert.equal(true, value, message);`

<!--english end-->

value가 `true`인지 검사한다. 이는 `assert.equal(true, value, message);`와 같다.

## assert.equal(actual, expected, [message])

<!--english start-->

Tests shallow, coercive equality with the equal comparison operator ( `==` ).

<!--english end-->

`==` 오퍼레이터로 같음(shallow, coercive equality)을 검사한다.

## assert.notEqual(actual, expected, [message])

<!--english start-->

Tests shallow, coercive non-equality with the not equal comparison operator ( `!=` ).

<!--english end-->

`!=` 오퍼레이터로 다름(shallow, coercive non-equality)을 검사한다.

## assert.deepEqual(actual, expected, [message])

<!--english start-->

Tests for deep equality.

<!--english end-->

깊은 동등성을 검사한다.
(역주, deep equality는 한마디로 설명하기 어렵다. 타입마다 다른 방법으로 같음을 테스트한다. 예를 들어, Date의 경우 `actual.getTime() === expected.getTime()`라고 비교한다. 필요하다면 assert.js을 일독하기를 권한다)

## assert.notDeepEqual(actual, expected, [message])

<!--english start-->

Tests for any deep inequality.

<!--english end-->

다름(deep inequality)를 검사한다.

## assert.strictEqual(actual, expected, [message])

<!--english start-->

Tests strict equality, as determined by the strict equality operator ( `===` )

<!--english end-->

`===` 오퍼레이터로 같음(strict equality)을 테스트한다.

## assert.notStrictEqual(actual, expected, [message])

<!--english start-->

Tests strict non-equality, as determined by the strict not equal operator ( `!==` )

<!--english end-->

`!==` 오퍼레이터로 다름(strict non-equality)을 테스트한다.

## assert.throws(block, [error], [message])

<!--english start-->

Expects `block` to throw an error. `error` can be constructor, regexp or 
validation function.

Validate instanceof using constructor:

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      Error
    );

Validate error message using RegExp:

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      /value/
    );

Custom error validation:

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      function(err) {
        if ( (err instanceof Error) && /value/.test(err) ) {
          return true;
        }
      },
      "unexpected error"
    );

<!--english end-->

`block`이 오류는 던지기를 기대한다. `error`는 생성자, 정규표현식, 유효성검사 함수가 
될 수 있다.

생성자의 인스턴스인지 테스트한다.

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      Error
    );

RegExp를 사용해서 오류 메시지를 검증한다.

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      /value/
    );

자신만의 방법으로 오류를 검증한다.

    assert.throws(
      function() {
        throw new Error("Wrong value");
      },
      function(err) {
        if ( (err instanceof Error) && /value/.test(err) ) {
          return true;
        }
      },
      "unexpected error"
    );

## assert.doesNotThrow(block, [error], [message])

<!--english start-->

Expects `block` not to throw an error, see assert.throws for details.

<!--english end-->

`block`이 오류를 던지지 않기를 기대한다. 자세한 내용은 assert.throws를 참고해라.

## assert.ifError(value)

<!--english start-->

Tests if value is not a false value, throws if it is a true value. Useful when
testing the first argument, `error` in callbacks.

<!--english end-->

value가 true인지 테스트하고 true이면 그 value를 던진다. 콜백에서 첫 아규먼트 
`error`를 검사하는 데 유용하다.
(역주, 예를 들면 `fs.readFile('notfound.js', assert.ifError);`라고 사용할 수 있어서 유용하다는 것이다)
