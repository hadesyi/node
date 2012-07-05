# Global Objects

<!--english start-->

<!-- type=misc -->

These objects are available in all modules. Some of these objects aren't
actually in the global scope but in the module scope - this will be noted.

<!--english end-->

<!-- type=misc -->

이 객체들은 모든 모듈에서 이용할 수 있다. 이 객체들 중 일부는 실제로 전역 범위를 가지지
않고 모듈 범위를 가진다. - 이는 따로 표시할 것이다.

## global

<!--english start-->

<!-- type=global -->

* {Object} The global namespace object.

In browsers, the top-level scope is the global scope. That means that in
browsers if you're in the global scope `var something` will define a global
variable. In Node this is different. The top-level scope is not the global
scope; `var something` inside a Node module will be local to that module.

<!--english end-->

<!-- type=global -->

* {Object} 전역 네임스페이스 객체.

브라우저에서 최상위 범위는 전역 번위이다. 이는 브라우저의 전역 범위에서 `var something`가
전역 변수를 정의한다는 것을 의미한다. Node에서는 다르다. 최상위 범위는 전역 범위가 아니다.
Node 모듈에서 `var something`는 해당 모듈의 지역 범위가 된다.

## process

<!--english start-->

<!-- type=global -->

* {Object}

The process object. See the [process object](process.html#process) section.

<!--english end-->

<!-- type=global -->

* {Object}

process 객체. [process object](process.html#process)부분을 봐라.

## console

<!--english start-->

<!-- type=global -->

* {Object}

Used to print to stdout and stderr. See the [stdio](stdio.html) section.

<!--english end-->

<!-- type=global -->

* {Object}

stdout와 stderr에 출력하는 데 사용한다. [stdio](stdio.html)부분을 봐라.

## Buffer

<!--english start-->

<!-- type=global -->

* {Object}

Used to handle binary data. See the [buffer section](buffer.html).

<!--english end-->

<!-- type=global -->

* {Object}

바이너리 데이터를 다루는데 사용한다. [buffer section](buffer.html)를 봐라.

## require()

<!--english start-->

<!-- type=var -->

* {Function}

To require modules. See the [Modules](modules.html#modules) section.
`require` isn't actually a global but rather local to each module.


<!--english end-->

<!-- type=var -->

* {Function}

모듈을 require한다. [Modules](modules.html#modules) 부분을 봐라.
`require`는 실제로 전역이 아니라 각 모듈의 지역범위다.


### require.resolve()

<!--english start-->

Use the internal `require()` machinery to look up the location of a module,
but rather than loading the module, just return the resolved filename.

<!--english end-->

모듈의 위치를 검색하는데 내부 `require()` 장치(machinery)를 사용한다. 모듈을 로딩하는 
것이 아니라 처리된 파일명을 리턴할 뿐이다.

### require.cache

<!--english start-->

* {Object}

Modules are cached in this object when they are required. By deleting a key
value from this object, the next `require` will reload the module.

<!--english end-->

* {Object}

모듈을 require했을 때 모듈은 이 객체에 캐시된다. 이 객체에서 키 값을 삭제하면 다음 번
`require`에서 해당 모듈을 다시 로드할 것이다.

## __filename

<!--english start-->

<!-- type=var -->

* {String}

The filename of the code being executed.  This is the resolved absolute path
of this code file.  For a main program this is not necessarily the same
filename used in the command line.  The value inside a module is the path
to that module file.

Example: running `node example.js` from `/Users/mjr`

    console.log(__filename);
    // /Users/mjr/example.js

`__filename` isn't actually a global but rather local to each module.

<!--english end-->

<!-- type=var -->

* {String}

실행되는 코드의 파일명이다. 이 코드 파일을 처리한 절대경로이다. 메인 프로그램에서 이는
커맨드라인에서 사용한 것과 반드시 같은 파일명은 아니다. 모듈내부에서 이 값은 해당 모듈 파일에
대한 경로이다. 

예제: `/Users/mjr`에서 `node example.js`를 실행한다.

    console.log(__filename);
    // /Users/mjr/example.js

`__filename`은 실제로 전역이 아니라 각 모듈의 지역범위이다.

## __dirname

<!--english start-->

<!-- type=var -->

* {String}

The name of the directory that the currently executing script resides in.

Example: running `node example.js` from `/Users/mjr`

    console.log(__dirname);
    // /Users/mjr

`__dirname` isn't actually a global but rather local to each module.


<!--english end-->

<!-- type=var -->

* {String}

현재 실행되는 스크립트가 존재하는 디렉토리 이름이다.

예제: `/Users/mjr`에서 `node example.js`를 실행한다.

    console.log(__dirname);
    // /Users/mjr

`__dirname`는 실제로 전역이 아니라 각 모듈의 지역범위이다.


## module

<!--english start-->

<!-- type=var -->

* {Object}

A reference to the current module. In particular
`module.exports` is the same as the `exports` object. See `src/node.js`
for more information.
`module` isn't actually a global but rather local to each module.


<!--english end-->

<!-- type=var -->

* {Object}

현재 모듈에 대한 참조이다. 특히 `module.exports`는 `exports` 객체와 같다.
더 자세한 내용은 `src/node.js`를 봐라.
`module`는 실제로 전역이 아니라 각 모듈의 지역범위이다.


## exports

<!--english start-->

<!-- type=var -->

An object which is shared between all instances of the current module and
made accessible through `require()`.
`exports` is the same as the `module.exports` object. See `src/node.js`
for more information.
`exports` isn't actually a global but rather local to each module.

See the [module system documentation](modules.html) for more
information.

See the [module section](modules.html) for more information.

<!--english end-->

<!-- type=var -->

현재 모듈과 `require()`로 접근가능하게 된 모듈의 모든 인스턴스 사이에서 공유되는 객체다.
`exports`는 `module.exports`객체와 동일하다. 더 자세한 내용은 `src/node.js`를 봐라.
`exports`는 실제로 전역이 아니라 각 모듈의 지역범위이다. 

더 자세한 내용은 [module system documentation](modules.html)를 봐라.

더 자세한 내용은 [module section](modules.html)를 봐라.

## setTimeout(cb, ms)
## clearTimeout(t)
## setInterval(cb, ms)
## clearInterval(t)

<!--english start-->

<!--type=global-->

The timer functions are global variables. See the [timers](timers.html) section.

<!--english end-->

<!--type=global-->

timer 함수는 전역 변수이다. [timers](timers.html)부분을 봐라.
