# Executing JavaScript

<!--english start-->

    Stability: 3 - Stable

<!--name=vm-->

You can access this module with:

    var vm = require('vm');

JavaScript code can be compiled and run immediately or compiled, saved, and run later.

<!--english end-->

    Stability: 3 - Stable

<!--name=vm-->

이 모듈은 다음과 같이 접근할 수 있다.

    var vm = require('vm');

자바스크립트 코드는 컴파일하고 바로 실행하거나 컴파일하고 저장한 후에 나중에 실행할 수 있다.


## vm.runInThisContext(code, [filename])

<!--english start-->

`vm.runInThisContext()` compiles `code`, runs it and returns the result. Running
code does not have access to local scope. `filename` is optional, it's used only
in stack traces.

Example of using `vm.runInThisContext` and `eval` to run the same code:

    var localVar = 123,
        usingscript, evaled,
        vm = require('vm');

    usingscript = vm.runInThisContext('localVar = 1;',
      'myfile.vm');
    console.log('localVar: ' + localVar + ', usingscript: ' +
      usingscript);
    evaled = eval('localVar = 1;');
    console.log('localVar: ' + localVar + ', evaled: ' +
      evaled);

    // localVar: 123, usingscript: 1
    // localVar: 1, evaled: 1

`vm.runInThisContext` does not have access to the local scope, so `localVar` is unchanged.
`eval` does have access to the local scope, so `localVar` is changed.

In case of syntax error in `code`, `vm.runInThisContext` emits the syntax error to stderr
and throws an exception.


<!--english end-->

`vm.runInThisContext()`는 `code`를 컴파일하고 실행해서 결과를 반환한다. 실행되는 코드는 
지역범위에 접근하지 않는다. `filename`은 선택사항이며 스택트레이스에서만 사용된다.

같은 코드를 실행하는데 `vm.runInThisContext`과 `eval`을 사용한 예제

    var localVar = 123,
        usingscript, evaled,
        vm = require('vm');

    usingscript = vm.runInThisContext('localVar = 1;',
      'myfile.vm');
    console.log('localVar: ' + localVar + ', usingscript: ' +
      usingscript);
    evaled = eval('localVar = 1;');
    console.log('localVar: ' + localVar + ', evaled: ' +
      evaled);

    // localVar: 123, usingscript: 1
    // localVar: 1, evaled: 1

`vm.runInThisContext`가 지역범위에 접근하지 않기 때문에 `localVar`는 변경되지 않았다.
`eval`은 지역범위에 접근하기 때문에 `localVar`가 변경되었다.

`code`에 문법오류가 있는 경우 `vm.runInThisContext`는 stderr에 문법오류를 발생시키고 예외를 
던진다.


## vm.runInNewContext(code, [sandbox], [filename])

<!--english start-->

`vm.runInNewContext` compiles `code`, then runs it in `sandbox` and returns the
result. Running code does not have access to local scope. The object `sandbox`
will be used as the global object for `code`.
`sandbox` and `filename` are optional, `filename` is only used in stack traces.

Example: compile and execute code that increments a global variable and sets a new one.
These globals are contained in the sandbox.

    var util = require('util'),
        vm = require('vm'),
        sandbox = {
          animal: 'cat',
          count: 2
        };

    vm.runInNewContext('count += 1; name = "kitty"', sandbox, 'myfile.vm');
    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 3, name: 'kitty' }

Note that running untrusted code is a tricky business requiring great care.  To prevent accidental
global variable leakage, `vm.runInNewContext` is quite useful, but safely running untrusted code
requires a separate process.

In case of syntax error in `code`, `vm.runInNewContext` emits the syntax error to stderr
and throws an exception.

<!--english end-->

`vm.runInNewContext`는 `code`를 컴파일하고 `sandbox`에서 실행한 뒤 결과를 반환한다. 
실행되는 코드는 지역범위에 접근하지 않는다. `sandbox` 객체는 `code`의 전역객체로 사용될 
것이다. 
`sandbox`와 `filename`는 선택사항이고 `filename`는 스택트레이스에서만 사용된다.

예제: 전역변수를 증가시키고 새로운 전역변수를 설정하는 코드를 컴파일하고 실행한다.
이 전역변수들은 sandbox안에 포함되어 있다.

    var util = require('util'),
        vm = require('vm'),
        sandbox = {
          animal: 'cat',
          count: 2
        };

    vm.runInNewContext('count += 1; name = "kitty"', sandbox, 'myfile.vm');
    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 3, name: 'kitty' }

신뢰할 수 없는 코드를 실행하는 것은 아주 신중을 요하는 복잡한 작업이다. 의도치않은 전역변수의 누수를 막는데 
`vm.runInNewContext`가 아주 유용하지만 신뢰하지 못하는 코드를 안전하게 실행하려면 프로세스를 분리할 필요가 
있다.

`code`에서 문법오류가 있는 경우 `vm.runInNewContext`는 stderr에 문법오류를 발생시키고 
예외를 던진다.

## vm.runInContext(code, context, [filename])

<!--english start-->

`vm.runInContext` compiles `code`, then runs it in `context` and returns the
result. A (V8) context comprises a global object, together with a set of
built-in objects and functions. Running code does not have access to local scope
and the global object held within `context` will be used as the global object
for `code`.
`filename` is optional, it's used only in stack traces.

Example: compile and execute code in a existing context.

    var util = require('util'),
        vm = require('vm'),
        initSandbox = {
          animal: 'cat',
          count: 2
        },
        context = vm.createContext(initSandbox);

    vm.runInContext('count += 1; name = "CATT"', context, 'myfile.vm');
    console.log(util.inspect(context));

    // { animal: 'cat', count: 3, name: 'CATT' }

Note that `createContext` will perform a shallow clone of the supplied sandbox object in order to
initialise the global object of the freshly constructed context.

Note that running untrusted code is a tricky business requiring great care.  To prevent accidental
global variable leakage, `vm.runInContext` is quite useful, but safely running untrusted code
requires a separate process.

In case of syntax error in `code`, `vm.runInContext` emits the syntax error to stderr
and throws an exception.

<!--english end-->

`vm.runInContext`는 `code`를 컴파일하고 `context`에서 실행한 뒤 결과를 반환한다. 
(V8) 컨텍스트트는 내장된 객체 및 기능들과 함께 하나의 전역객체로 이루어져 있다. 
실행되는 코드는 지역범위에 접근하지 않고 전역객체는 `code`의 전역객체로 사용될 
`context`내에 있다.
`filename`는 선택사항이고 스택트레이스에서만 사용된다.

예제: 존재하는 컨텍스트에서 코드를 컴파일하고 실행한다.

    var util = require('util'),
        vm = require('vm'),
        initSandbox = {
          animal: 'cat',
          count: 2
        },
        context = vm.createContext(initSandbox);

    vm.runInContext('count += 1; name = "CATT"', context, 'myfile.vm');
    console.log(util.inspect(context));

    // { animal: 'cat', count: 3, name: 'CATT' }

`createContext`는 새로 생성된 컨텍스트의 전역객체를 초기화하기 위해서 제공된 샌드박스 객체의 얕은 복제를 
수행할 것이다.

신뢰할 수 없는 코드를 실행하는 것은 아주 신중을 요하는 복잡한 작업이다. 의도치않은 전역변수의 누수를 막는데 
`vm.runInContext`가 아주 유용하지만 신뢰하지 못하는 코드를 안전하게 실행하려면 프로세스를 분리할 필요가 
있다.

`code`에서 문법오류가 있는 경우 `vm.runInContext`는 stderr에 문법오류를 발생시키고 
예외를 던진다.

## vm.createContext([initSandbox])

<!--english start-->

`vm.createContext` creates a new context which is suitable for use as the 2nd argument of a subsequent
call to `vm.runInContext`. A (V8) context comprises a global object together with a set of
build-in objects and functions. The optional argument `initSandbox` will be shallow-copied
to seed the initial contents of the global object used by the context.

<!--english end-->

`vm.createContext`는 뒤이은 `vm.runInContext` 호출의 두 번째 파라미터로 사용하기에 적합한 새로운 
컨텍스트를 생성한다. (V8) 컨텍스트트는 내장된 객체 및 기능들과 함께 하나의 전역객체로 이루어져 있다. 
선택적인 아규먼트인 `initSandbox`는 컨텍스트가 사용할 전역객체의 초기 내용을 생성하려고 얕은 
복사가 될 것이다.

## vm.createScript(code, [filename])

<!--english start-->

`createScript` compiles `code` but does not run it. Instead, it returns a
`vm.Script` object representing this compiled code. This script can be run
later many times using methods below. The returned script is not bound to any
global object. It is bound before each run, just for that run. `filename` is
optional, it's only used in stack traces.

In case of syntax error in `code`, `createScript` prints the syntax error to stderr
and throws an exception.

<!--english end-->

`createScript`는 `code`를 컴파일하지만 실행하지는 않는다. 대신 컴파일된 코드를 나타내는 
`vm.Script` 객체를 반환한다. 이 스크립트는 아래의 메서드들을 사용해서 나중에 여려 번 
실행할 수 있다. 반환된 스크립트는 어떤 전역객체에도 바인딩되지 않는다. 반환된 스크립트는 매번 
실행되기 전에 해당 실행에 대해서 바인딩된다. `filename`는 선택사항이고 스택트레이스에서만 
사용될 것이다.

`code`에 문법오류가 있는 경우 `createScript`는 stderr에 문법오류를 출력하고 
예외를 던진다.


## Class: Script

<!--english start-->

A class for running scripts.  Returned by vm.createScript.

<!--english end-->

스크립트를 실행하는 클래스. vm.createScript가 반환하는 클래스다.

### script.runInThisContext()

<!--english start-->

Similar to `vm.runInThisContext` but a method of a precompiled `Script` object.
`script.runInThisContext` runs the code of `script` and returns the result.
Running code does not have access to local scope, but does have access to the `global` object
(v8: in actual context).

Example of using `script.runInThisContext` to compile code once and run it multiple times:

    var vm = require('vm');

    globalVar = 0;

    var script = vm.createScript('globalVar += 1', 'myfile.vm');

    for (var i = 0; i < 1000 ; i += 1) {
      script.runInThisContext();
    }

    console.log(globalVar);

    // 1000

<!--english end-->

`vm.runInThisContext`와 유사하지만 미리 컴파일된 `Script` 객체의 메서드이다. 
`script.runInThisContext`는 `script`의 코드를 실행하고 결과를 반환한다.
실행되는 코드는 지역범위에 접근하지 않지만 `global` 객체에는 접근한다.
(v8: 실제 컨텍스트에서)

한번 코드를 컴파일하고 여러 번 실행하는 `script.runInThisContext`를 사용하는 예제

    var vm = require('vm');

    globalVar = 0;

    var script = vm.createScript('globalVar += 1', 'myfile.vm');

    for (var i = 0; i < 1000 ; i += 1) {
      script.runInThisContext();
    }

    console.log(globalVar);

    // 1000


### script.runInNewContext([sandbox])

<!--english start-->

Similar to `vm.runInNewContext` a method of a precompiled `Script` object.
`script.runInNewContext` runs the code of `script` with `sandbox` as the global object and returns the result.
Running code does not have access to local scope. `sandbox` is optional.

Example: compile code that increments a global variable and sets one, then execute this code multiple times.
These globals are contained in the sandbox.

    var util = require('util'),
        vm = require('vm'),
        sandbox = {
          animal: 'cat',
          count: 2
        };

    var script = vm.createScript('count += 1; name = "kitty"', 'myfile.vm');

    for (var i = 0; i < 10 ; i += 1) {
      script.runInNewContext(sandbox);
    }

    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 12, name: 'kitty' }

Note that running untrusted code is a tricky business requiring great care.  To prevent accidental
global variable leakage, `script.runInNewContext` is quite useful, but safely running untrusted code
requires a separate process.

<!--english end-->

`vm.runInNewContext`와 유사하지만 미리 컴파일된 `Script` 객체의 메서드이다.
`script.runInNewContext`는 전역객체인 `sandbox`와 함께 `script`의 코드를 실행하고 결과를 반환한다.
실행되는 코드는 지역범위에 접근하지 않는다. `sandbox`는 선택사항이다.

예제: 전역변수가 증가하고 전역변수를 설정하는 코드를 컴파일하고 이 코드를 여러 번 실행한다.
이 전역변수들은 sandbox안에 있다.

    var util = require('util'),
        vm = require('vm'),
        sandbox = {
          animal: 'cat',
          count: 2
        };

    var script = vm.createScript('count += 1; name = "kitty"', 'myfile.vm');

    for (var i = 0; i < 10 ; i += 1) {
      script.runInNewContext(sandbox);
    }

    console.log(util.inspect(sandbox));

    // { animal: 'cat', count: 12, name: 'kitty' }

신뢰할 수 없는 코드를 실행하는 것은 아주 신중을 요하는 복잡한 작업이다. 의도치않은 전역변수의 누수를 막는데 
`script.runInNewContext`가 아주 유용하지만 신뢰하지 못하는 코드를 안전하게 실행하려면 프로세스를 
분리할 필요가 있다.
