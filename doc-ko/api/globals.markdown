# Global Objects

<!-- type=misc -->

이 객체들은 모든 모듈에서 이용할 수 있다. 이 객체들 중 일부는 실제로 전역 범위를 가지지
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

process 객체. [process object](process.html#process)부분을 봐라.

## console

<!-- type=global -->

* {Object}

stdout와 stderr에 출력하는 데 사용한다. [stdio](stdio.html)부분을 봐라.

## Buffer

<!-- type=global -->

* {Object}

바이너리 데이터를 다루는데 사용한다. [buffer section](buffer.html)를 봐라.

## require()

<!-- type=var -->

* {Function}

모듈을 require한다. [Modules](modules.html#modules) 부분을 봐라.
`require`는 실제로 전역이 아니라 각 모듈의 지역범위다.


### require.resolve()

모듈의 위치를 검색하는데 내부 `require()` 장치(machinery)를 사용한다. 모듈을 로딩하는 
것이 아니라 처리된 파일명을 리턴할 뿐이다.

### require.cache

* {Object}

모듈을 require했을 때 모듈은 이 객체에 캐시된다. 이 객체에서 키 값을 삭제하면 다음 번
`require`에서 해당 모듈을 다시 로드할 것이다.

## __filename

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

<!-- type=var -->

* {String}

현재 실행되는 스크립트가 존재하는 디렉토리 이름이다.

예제: `/Users/mjr`에서 `node example.js`를 실행한다.

    console.log(__dirname);
    // /Users/mjr

`__dirname`는 실제로 전역이 아니라 각 모듈의 지역범위이다.


## module

<!-- type=var -->

* {Object}

현재 모듈에 대한 참조이다. 특히 `module.exports`는 `exports` 객체와 같다.
더 자세한 내용은 `src/node.js`를 봐라.
`module`는 실제로 전역이 아니라 각 모듈의 지역범위이다.


## exports

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

<!--type=global-->

timer 함수는 전역 변수이다. [timers](timers.html)부분을 봐라.
