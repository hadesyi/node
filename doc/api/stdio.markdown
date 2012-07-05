# console

<!--english start-->

    Stability: 4 - API Frozen

* {Object}

<!--type=global-->

For printing to stdout and stderr.  Similar to the console object functions
provided by most web browsers, here the output is sent to stdout or stderr.


<!--english end-->

    안정성: 4 - API Frozen

* {Object}

<!--type=global-->

stdout와 stderr에 출력하기 위해 사용한다. 대부분의 웹 브라우저가 제공하는 console 
객체의 기능과 유사하게 stdout이나 stderr로 출력한다.


## console.log()

<!--english start-->

Prints to stdout with newline. This function can take multiple arguments in a
`printf()`-like way. Example:

    console.log('count: %d', count);

If formatting elements are not found in the first string then `util.inspect`
is used on each argument.
See [util.format()](util.html#util.format) for more information.

<!--english end-->

새로운 라인으로 stdout에 출력한다. 이 함수는 `printf()`와 같은 방식으로 여러 아규먼트를
받는다. 예제:

    console.log('count: %d', count);

첫 문자열에 포매팅 객체가 없으면 각 아규먼트에 `util.inspect`를 사용한다.
더 자세한 내용은 [util.format()](util.html#util.format)를 봐라.

## console.info()

<!--english start-->

Same as `console.log`.

<!--english end-->

`console.log`와 동일하다.

## console.warn()
## console.error()

<!--english start-->

Same as `console.log` but prints to stderr.

<!--english end-->

`console.log`와 같지만 stderr에 출력한다.

## console.dir(obj)

<!--english start-->

Uses `util.inspect` on `obj` and prints resulting string to stderr.

<!--english end-->

`obj`에 `util.inspect`를 사용하고 결과 문자열을 stderr에 출력한다.

## console.time(label)

<!--english start-->

Mark a time.


<!--english end-->

시간을 마킹한다.


## console.timeEnd(label)

<!--english start-->

Finish timer, record output. Example

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');


<!--english end-->

타이머를 종료하고 결과를 기록한다. 예제

    console.time('100-elements');
    for (var i = 0; i < 100; i++) {
      ;
    }
    console.timeEnd('100-elements');


## console.trace()

<!--english start-->

Print a stack trace to stderr of the current position.

<!--english end-->

현지 위치의 stderr에 스택트레이스를 출력한다.

## console.assert()

<!--english start-->

Same as `assert.ok()`.

<!--english end-->

`assert.ok()`과 같다.

