# Global Objects

<!-- type=misc -->

이 객체들은 모든 모듈에서 이용할 수 있다. 이 객체 중 일부는 실제로 전역 범위를 가지지
않고 모듈 범위를 가진다. - 이는 따로 표시할 것이다.

## global

<!-- type=global -->

* {Object} 전역 네임스페이스 객체.

브라우저에서 최상위 범위는 전역 범위이다. 이는 브라우저의 전역 범위에서 `var something`가
전역 변수를 정의한다는 것을 의미한다. Node에서는 다르다. 최상위 범위는 전역 범위가 아니다.
Node 모듈에서 `var something`는 해당 모듈의 지역 범위가 된다.

## process

<!-- type=global -->

* {Object}

process 객체. [process object][]부분을 봐라.

## console

<!-- type=global -->

* {Object}

stdout와 stderr에 출력하는 데 사용한다. [console][]부분을 봐라.

## Class: Buffer

<!-- type=global -->

* {Function}

바이너리 데이터를 다루는 데 사용한다. [buffer section][]을 봐라.

## require()

<!-- type=var -->

* {Function}

모듈을 require한다. [Modules][]부분을 봐라.
`require`는 실제로 전역이 아니라 각 모듈의 지역 범위다.

### require.resolve()

모듈의 위치를 검색하는데 내부 `require()` 장치(machinery)를 사용한다. 모듈을 로딩하는
것이 아니라 처리된 파일명을 반환할 뿐이다.

### require.cache

* {Object}

모듈을 require했을 때 모듈은 이 객체에 캐시 된다. 이 객체에서 킷값을 삭제하면 다음번
`require`에서 해당 모듈을 다시 로드할 것이다.

### require.extensions

    Stability: 0 - Deprecated

* {Object}

특정 파일 확장자를 어떻게 다룰지를 `require`에 지시한다.

`.sjs` 확장자의 파일을 `.js`처럼 처리한다.

    require.extensions['.sjs'] = require.extensions['.js'];

**Deprecated**  과거에 이 항목은 요청에 따라 컴파일해서 Node에 자바스크립트가 아닌 모듈을
로드하는데 사용했다. 하지만 실제로 다른 몇몇 Node 프로그램을 통해서 모듈을 로딩하거나
ahead of time으로 자바스크립트로 컴파일하는 등의 이를 위한 훨씬 더 좋은 방법이 존재한다.

Module 시스템이 locked 상태이므로 이 기능은 없어지지 않을 것이다. 하지만 미묘한 버그나
건드릴 수 없는 복잡성을 가질 수 있다.

## __filename

<!-- type=var -->

* {String}

실행되는 코드의 파일명이다. 이 코드 파일을 처리한 절대 경로이다. 메인 프로그램에서 이는
커맨드라인에서 사용한 것과 반드시 같은 파일명은 아니다. 모듈 내부에서 이 값은 해당 모듈 파일에
대한 경로이다.

예제: `/Users/mjr`에서 `node example.js`를 실행한다.

    console.log(__filename);
    // /Users/mjr/example.js

`__filename`은 실제로 전역이 아니라 각 모듈의 지역 범위이다.

## __dirname

<!-- type=var -->

* {String}

현재 실행되는 스크립트가 존재하는 디렉터리 이름이다.

예제: `/Users/mjr`에서 `node example.js`를 실행한다.

    console.log(__dirname);
    // /Users/mjr

`__dirname`는 실제로 전역이 아니라 각 모듈의 지역 범위이다.


## module

<!-- type=var -->

* {Object}

현재 모듈에 대한 참조이다. 특히, `module.exports`는 모듈이 무엇을 외부에 노출해서
`require()`로 사용할 수 있게 할 것인지 정의하는 데 사용한다.

`module`는 실제로 전역이 아니라 각 모듈의 지역 범위이다.

더 자세한 내용은 [module system documentation][]를 봐라.

## exports

<!-- type=var -->

`module.exports`에 대한 더 간략화된 참조다. 언제 `exports`를 사용하고
언제 `module.exports`를 사용하는지에 대한 자세한 내용은
[module system documentation][]를 참고해라.

`exports`는 실제로 전역이 아니라 각 모듈의 지역 범위이다.

더 자세한 내용은 [module system documentation][]를 봐라.

더 자세한 내용은 [module section][]를 봐라.

## setTimeout(cb, ms)

*최소* `ms` 밀리 초 후에 콜백 `cb`를 실행한다. 실제 지연시간은 OS 타이머의 크기와 시스템
부하 같은 외부 요소에 달려있다.

타임아웃은 1-2,147,483,647의 범위여야 한다. 값이 이 범위 밖이면 타임아웃은 1 밀리 초로
바뀐다. 대략 말해서 타이머는 24.8일 이상이 될 수 없다.

타이머를 나타내는 불투명한 값을 반환한다.

## clearTimeout(t)

이전에 `setTimeout()`로 생성된 타이머를 멈춘다. 콜백은 실행하지 않을 것이다.

## setInterval(cb, ms)

`ms` 밀리 초마다 반복적으로 콜백 `cb`를 실행한다. 실제 간격은 OS 타이머의 크기나 시스템
부하 같은 외부 요소에 따라 다양하다. 시간 간격은 `ms`보다 작을 수 없다.

간격은 1-2,147,483,647의 범위여야 한다. 값이 이 범위 밖이면 1밀리 초로 바뀐다. 대략
말해서 타이머는 24.8일 이상이 될 수 없다.

타이머를 나타내는 불투명한 값을 반환한다.

## clearInterval(t)

이전에 `setInterval()`로 생성된 타이머를 멈춘다. 콜백은 실행하지 않을 것이다.

<!--type=global-->

timer 함수는 전역 변수이다. [timers][]부분을 봐라.

[buffer section]: buffer.html
[module section]: modules.html
[module system documentation]: modules.html
[Modules]: modules.html#modules_modules
[process object]: process.html#process_process
[console]: console.html
[timers]: timers.html
