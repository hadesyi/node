# util

    Stability: 5 - Locked

이 함수들은 `'util'` 모듈에 있다. 이 모듈에 접근하려면 `require('util')`를 사용해야
한다.


## util.format(format, [...])

`printf`같은 형식으로 첫 아규먼트를 사용해서 포매팅된 문자열을 반환한다.

첫 아규먼트는 *플레이스홀더*가 포함된 문자열이다.(플레이스 홀더는 없어도 된다.)
각 플레이스 홀더는 대응되는 아규먼트의 값으로 대체된다. 플레이스 홀더는
다음을 지원한다.

* `%s` - 문자열.
* `%d` - 숫자 (integer 와 float를 모두 지원한다.).
* `%j` - JSON.
* `%%` - 퍼센트기호 (`'%'`). 이 기호는 플레이스홀더 아규먼트를 사용하지 않는다.

플레이스 홀더에 대응되는 아규먼트가 없으면 플레이스홀더는 치환되지 않는다.

    util.format('%s:%s', 'foo'); // 'foo:%s'

플레이스홀더보다 많은 수의 아규먼트가 있으면 남는 아규먼트들은 `util.inspect()`를 사용해서
문자열로 변환되고 스페이스를 구분자로 이 문자열들을 이어붙힌다.

    util.format('%s:%s', 'foo', 'bar', 'baz'); // 'foo:bar baz'

첫 아규먼트가 문자열이 아니라면 `util.format()`은 모든 아규먼트를 공백문자로 이어붙혀서
리턴한다. 각 아규먼트는 `util.inspect()`를 통해 문자열로 변환한다.

    util.format(1, 2, 3); // '1 2 3'


## util.debug(string)

동기적인 출력함수. 프로세스를 블락할 것이고 `stderr`에 즉각적으로
`string`을 출력한다.

    require('util').debug('message on stderr');

## util.error([...])

즉시 모든 아규먼트를 `stderr`에 출력한다는 점을 제외하면 `util.debug()`와 같다.

## util.puts([...])

동기적인 출력함수. 프로세스를 블락할 것이고 각 아규먼트마다 새로운 라인으로 `stdout`에 
모든 아규먼트를 출력할 것이다.

## util.print([...])

동기적인 출력함수. 프로세스를 블락할 것이고 각 아규먼트를 문자열로 변환해서 `stdout`에 
출력한다. 아규먼트마다 새로운 라인을 넣지 않는다.

## util.log(string)

`stdout`에 타임스탬프를 출력한다.

    require('util').log('Timestamped message.');


## util.inspect(object, [showHidden], [depth], [colors])

디버깅에 유용한 `object`의 문자열 표현을 리턴한다.

`showHidden`가 `true`이면 객체의 enumerable하지 않는 프로퍼티도 보여준다.
기본값은 `false`이다.

`depth`는 객체를 포매팅할 때 `inspect`가 몇 번 재귀를 할 것인지를 지정한다.
이 값은 크고 복잡한 객체를 검사할 때 유용하다.

기본값은 딱 두번만 재귀한다. 무한대로 재귀하려면 `depth`에 `null`을 지정한다.

`colors`가 `true`이면 ANSI 색상코드로 스타일을 입혀서 출력한다.

다음은 `util`객체의 모든 프로퍼티를 검사하는 예제다:

    var util = require('util');

    console.log(util.inspect(util, true, null));

객체들도 `util.inspect()`가 호출해서 객체를 건사한 결과를 사용하는 자신만의
`inspect(depth)` 함수를 정의할 수 있다.

    var util = require('util');

    var obj = { name: 'nate' };
    obj.inspect = function(depth) {
      return '{' + this.name + '}';
    };

    util.inspect(obj);
      // "{nate}"


## util.isArray(object)

주어진 "object"가 `Array`이면 `true`를 리턴하고 `Array`가 아니면 `false`를
리턴한다.

    var util = require('util');

    util.isArray([])
      // true
    util.isArray(new Array)
      // true
    util.isArray({})
      // false


## util.isRegExp(object)

주어진 "object"가 `RegExp`이면 `true`를 리턴하고 `RegExp`가 아니면 
`false`를 리턴한다.

    var util = require('util');

    util.isRegExp(/some regexp/)
      // true
    util.isRegExp(new RegExp('another regexp'))
      // true
    util.isRegExp({})
      // false


## util.isDate(object)

주어진 "object"가 `Date`이면 `true`를 리턴하고 `Date`가 아니면 
`false`를 리턴한다.

    var util = require('util');

    util.isDate(new Date())
      // true
    util.isDate(Date())
      // false (without 'new' returns a String)
    util.isDate({})
      // false


## util.isError(object)

주어진 "object"가 `Error`이면 `true`를 리턴하고 `Error`가 아니면 
`false`를 리턴한다.

    var util = require('util');

    util.isError(new Error())
      // true
    util.isError(new TypeError())
      // true
    util.isError({ name: 'Error', message: 'an error occurred' })
      // false


## util.pump(readableStream, writableStream, [callback])

    Stability: 0 - Deprecated: readableStream.pipe(writableStream)를 사용해라

`readableStream`에서 데이터를 읽어서 읽은 데이터를 `writableStream`으로 보낸다.
`writableStream.write(data)`가 `false`를 리턴하면 `writableStream`에서 
`drain`이벤트가 발생할 때까지 `readableStream`은 멈출 것이다. `callback`은 유일한
아규먼트로 error를 받고 `writableStream`이 닫히거나 오류가 발생했을 때 호출된다.


## util.inherits(constructor, superConstructor)

한 객체의 
[생성자](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Object/constructor)
에서 다른 객체로 프로토타입 메서드를 상속 받는다. `constructor`의 프로토타입은 
`superConstructor`에서 생성된 새로운 객체로 설정될 것이다.

`superConstructor`는 `constructor.super_` 프로퍼티를
통해서 편리하게 접근할 수 있다.

    var util = require("util");
    var events = require("events");

    function MyStream() {
        events.EventEmitter.call(this);
    }

    util.inherits(MyStream, events.EventEmitter);

    MyStream.prototype.write = function(data) {
        this.emit("data", data);
    }

    var stream = new MyStream();

    console.log(stream instanceof events.EventEmitter); // true
    console.log(MyStream.super_ === events.EventEmitter); // true

    stream.on("data", function(data) {
        console.log('Received data: "' + data + '"');
    })
    stream.write("It works!"); // Received data: "It works!"
