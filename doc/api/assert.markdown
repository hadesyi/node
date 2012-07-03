# Assert

    Stability: 5 - Locked

This module is used for writing unit tests for your applications, you can
access it with `require('assert')`.

    안정성: 5 - Locked

이 모듈을 어플리케이션에서 유닛 테스트를 작성하는데 사용하고 `require('assert')`로 
접근할 수 있다.

## assert.fail(actual, expected, message, operator)

Throws an exception that displays the values for `actual` and `expected` separated by the provided operator.

전달한 operator로 구분해서 `actual`과 `expected`의 값을 보여주는 예외를 던진다.

## assert(value, message), assert.ok(value, [message])

Tests if value is a `true` value, it is equivalent to `assert.equal(true, value, message);`

value가 `true`인지 검사한다. 이는 `assert.equal(true, value, message);`와 같다.

## assert.equal(actual, expected, [message])

Tests shallow, coercive equality with the equal comparison operator ( `==` ).

동등비교 오퍼레이터( `==` )로 얕고 강제적인 동등성을 검사한다.

## assert.notEqual(actual, expected, [message])

Tests shallow, coercive non-equality with the not equal comparison operator ( `!=` ).

부등비교 오퍼레이터( `!=` )로 얕고 강제적인 부등성을 검사한다.

## assert.deepEqual(actual, expected, [message])

Tests for deep equality.

깊은 동등성을 검사한다.

## assert.notDeepEqual(actual, expected, [message])

Tests for any deep inequality.

깊은 부등성을 검사한다.

## assert.strictEqual(actual, expected, [message])

Tests strict equality, as determined by the strict equality operator ( `===` )

엄격한 동등비교 오퍼레이터( `===` )로 엄격한 동등성을 검사한다.

## assert.notStrictEqual(actual, expected, [message])

Tests strict non-equality, as determined by the strict not equal operator ( `!==` )

엄격한 부등비교 오퍼레이터( `!==` )로 엄격한 부등성을 검사한다.

## assert.throws(block, [error], [message])

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

`block`이 오류는 던지기를 기대한다. `error`는 생성자, 정규표현식, 유효성검사 함수가 
될 수 있다.

생성자를 사용해서 instanceof를 검증한다.

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

커스텀 오류 검증

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

Expects `block` not to throw an error, see assert.throws for details.

`block`이 오류를 던지지 않기를 기대한다. 자세한 내용은 assert.throws를 참고해라.

## assert.ifError(value)

Tests if value is not a false value, throws if it is a true value. Useful when
testing the first argument, `error` in callbacks.

value가 false가 아닌지 검사하고 value가 true이면 예외를 던진다. 콜백에서 첫 아규먼트 
`error`를 검사하는 데 유용하다.
