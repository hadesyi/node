# Modules

    Stability: 5 - Locked

<!--name=module-->

매우 간단하게 모듈을 로딩할 수 있다. 노드에서는 파일 하나가 모듈 하나다. 예를 들어 `foo.js` 파일에서
같은 디렉터리에 있는 `circle.js`를 로드하는 것을 살펴보자.

`foo.js`:

    var circle = require('./circle.js');
    console.log( 'The area of a circle of radius 4 is '
               + circle.area(4));

`circle.js`:

    var PI = Math.PI;

    exports.area = function (r) {
      return PI * r * r;
    };

    exports.circumference = function (r) {
      return 2 * PI * r;
    };

`circle.js` 모듈은 `area()`와 `circumference()`를 익스포트했다.
모듈 루트에 함수와 객체를 추가하려면 `exports` 객체에 추가하면 된다.

모듈 내의 로컬 변수는 모듈이 함수로 감싸진 것처럼 외부에 노출되지 않는다.(private)
이 예제에서 `PI` 변수는 `circle.js`의 private이다.

모듈 전체가 하나의 함수로 익스포트하고 싶거나(생성자처럼) 한 번에 하나의 프로퍼티를 구성하는
대신 한 번의 할당으로 전체 객체를 익스포트하고 싶다면 `exports`말고 `module.exports`에 할당해라.

아래 `bar.js`는 생성자를 익스포트하는 `square` 모듈을 사용하고 있다.

    var square = require('./square.js');
    var mySquare = square(2);
    console.log('The area of my square is ' + mySquare.area());

`square` 모듈은 `square.js`에 정의되어 있다.

    // exports에 할당하는 것이 모듈을 수정하지 않는다. 반드시 module.exports를 사용해라.
    module.exports = function(width) {
      return {
        area: function() {
          return width * width;
        }
      };
    }

이 모듈 시스템은 `require("module")` 모듈에 구현되어 있다.

## Cycles

<!--type=misc-->

두 모듈이 `require()` 함수로 서로 참조할 때는 한쪽 모듈은 아직 완전히 로딩하지 못한 미완성
모듈을 그냥 반환한다.

이게 무슨 소리냐 하면:

`a.js`:

    console.log('a starting');
    exports.done = false;
    var b = require('./b.js');
    console.log('in a, b.done = %j', b.done);
    exports.done = true;
    console.log('a done');

`b.js`:

    console.log('b starting');
    exports.done = false;
    var a = require('./a.js');
    console.log('in b, a.done = %j', a.done);
    exports.done = true;
    console.log('b done');

`main.js`:

    console.log('main starting');
    var a = require('./a.js');
    var b = require('./b.js');
    console.log('in main, a.done=%j, b.done=%j', a.done, b.done);

`main.js`는 `a.js`를 로드하고, `a.js`는 `b.js`를 로드한다. 여기서 `b.js`는 다시 `a.js`를
로드하려고 한다. 무한 루프가 생기지 않도록 아직 미완성인 `a.js`의 `exports` 객체를 `b.js`에
반환해 버린다. 그리고 `b.js`가 완성되면 `a.js`에 반환된다.

`main.js`이 두 모듈을 로드할 때는 이미 둘 다 완성됐다. 이 프로그램의 실행 결과는 다음과 같다:

    $ node main.js
    main starting
    a starting
    b starting
    in b, a.done = false
    b done
    in a, b.done = true
    a done
    in main, a.done=true, b.done=true

그러니까 꼭 모듈을 서로 참조하게 하여야 하면 계획을 잘 짜야 한다.

## Core Modules

<!--type=misc-->

Node 모듈 중에서는 바이너리로 컴파일해야 하는 모듈이 있다. 코어 모듈은 이 문서 곳곳에서 설명한다.

코어 모듈은 Node 소스코드의 `lib/` 폴더에 들어 있다.

모듈을 require 하면 항상 코어 모듈이 먼저 로드된다. 예를 들어, `require('http')`로 로드될 것
같은 파일이 있어도 Node에 들어 있는 HTTP 모듈이 반환된다.

## File Modules

<!--type=misc-->

입력한 이름으로 파일을 못 찾으면 Node는 그 이름에 `.js`, `.json`, `.node`를 붙이고
해당 파일이 있는지 찾는다.

`.js` 파일은 JavaScript 텍스트 파일로 Interpret하고 `.json`은 JSON 텍스트 파일로
Interpret한다. 그리고 `.node` 파일은 컴파일한 addon 모듈이라서 `dlopen`으로 로드한다.

모듈을 절대 경로로 찾을 때는 모듈 이름을 `'/'`로 시작하면 된다. 예를 들어,
`require('home/marco/foo.js')`는 `/home/marco/foo.js` 파일을 로드한다.

모듈을 상대 경로로 찾으려면 모듈 이름이 `'./'`로 시작하면 된다. 즉, `foo.js`라는 파일에서
`require('./circle')`라고 호출하면 같은 디렉터리에 있는 `circle.js`를 로드한다.

'/'이나 './'로 시작하지 않으면 그냥 파일이 아니라 "코어 모듈"이나 `node_modules` 폴더에 있는
모듈을 찾는다.

주어진 경로가 존재하지 않으면 `require()`는 `code` 프로퍼티를
`'MODULE_NOT_FOUND'`로 설정해서 Error를 던질 것이다.

## Loading from `node_modules` Folders

<!--type=misc-->

`require()`에 넘어온 모듈 ID가 네이티브 모듈을 가리키는 것도 아니고, 그 모듈 ID가
`'/'`, `'./'`, `'../'`로 시작하지도 않으면 Node는 그 모듈의 상위 디렉터리에서 찾기 시작한다.
상위 디렉터리에 있는 `/node_modules`에서 해당 모듈을 찾는다.

만약 못 찾으면 상위 디렉터리에서 찾고, 그래도 못 찾으면 상위의 상위 디렉터리에서 찾는다.
파일 시스템에서 루트 디렉터리에 다다를 때까지 계속 찾는다.

예를 들어, `'/home/ry/projects/foo.js'`라는 파일에서 `requre('bar.js')`라고 호출하면
다음과 같은 순서로 모듈을 찾는다:

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

그래서 해당 프로그램만의 의존성을 독립적으로 관리할 수 있다. 다른 프로그램에 영향을 끼치지 않는다.

## Folders as Modules

<!--type=misc-->

모듈을 폴더로 관리하면 프로그램과 라이브러리를 묶음으로 관리할 수 있어 편리하다. 마치 한 파일로 된
모듈처럼 취급한다. 모듈이 폴더일 때 `require()`는 세 가지 방법으로 모듈을 찾는다.

프로그램 폴더에 `package.json` 파일을 만들고 main 모듈이 무엇인지 적는다:

    { "name" : "some-library",
      "main" : "./lib/some-library.js" }

이 파일이 `./some-library`라는 폴더에 있다고 하고, `require('./some-library')`를 호출하면
`./some-library/lib/some-library.js`를 찾아 로드한다.

Node가 package.json을 읽고 사용하기 때문에 이런 게 가능하다.

그 디렉터리에 package.json 파일이 없으면 Node는 `index.js`나 `index.node` 파일을 찾는다.
package.json 파일이 없으면 `require('./some-library')`는 다음과 같은 파일을 로드한다.

* `./some-library/index.js`
* `./some-library/index.node`

## Caching

<!--type=misc-->

한 번 로드한 모듈은 계속 캐싱한다. 그래서 `require('foo')`을 여러 번 호출해도 계속 같은 객체를
반환한다. 단, `require('foo')가 계속 같은 파일을 로드할 때만 그렇다.

`require('foo')`를 여러 번 호출해도 해당 모듈 코드는 단 한 번만 호출된다. 그리고 아직 미완성인
객체가 반환될 수 있다는 점까지 더하면 특정 모듈이 서로 의존하고 있어도 성공적으로 로드되는 마법이
이루어진다.

어떤 코드가 꼭 여러 번 호출돼야 하면 함수 자체를 Export하고 그 함수를 여러 번 호출하라.

### Module Caching Caveats

<!--type=misc-->

모듈은 찾은(resolved) 파일 이름을 키로 캐싱한다. `node_modules` 폴더에서 로딩하는 것이기 때문에
같은 require 코드라도 호출하는 위치에 따라 찾은 파일이 다를 수 있다. 즉, `require('foo')`가
다른 파일을 찾아낸다면 다른 객체를 반환한다.

## The `module` Object

<!-- type=var -->
<!-- name=module -->

* {Object}

모듈에서 `module` 변수는 해당 모듈 객체를 가리킨다. `module.exports`는
`exports` 전역모듈(module-global)로 편리하게 접근할 수 있다. `module`은 글로벌 변수가
아니라 모듈마다 다른 객체를 가리키는 로컬 변수다.

### module.exports

* {Object}

`module.exports` 객체는 Module 시스템이 자동으로 만들어 준다. Export하려는 객체를
`module.exports`에 할당해서 직접 만든 객체가 반환되게 할 수도 있다. 원하는 객체를
`exports`에 할당하면 원하는 것과는 달리 로컬 `exports` 변수가 다시 바인딩 될 것이다.

`.js`라는 모듈을 만들어 보자.

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

이 모듈은 다음과 같이 사용한다.

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });

`module.exports`에 할당하는 것은 바로 실행되도록 해야 한다. 콜백으로 할당문이 실행되는 것을
미루면 뜻대로 동작하지 않는다. 다음과 같이 하지 마라.

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);

#### exports alias

모듈 내에서 사용 가능한 `exports` 변수는 처음에는 `module.exports`에 대한 참조이다. 일반적인
변수와 마찬가지로 새로운 값을 `exports`에 할당하면 이전 값에 대한 바인딩을 사라진다.

이 동작을 설명하기 위해 다음의 `require()`에 대한 가상 구현체를 생각해 보자.

    function require(...) {
      // ...
      function (module, exports) {
        // Your module code here
        exports = some_func;        // exports를 재할당한다. exports는 더는 단축키가
                                    // 아니고 아무것도 익스포트하지 않는다.
        module.exports = some_func; // 모듈이 0을 익스포트하게 한다.
      } (module, module.exports);
      return module;
    }

`exports`와 `module.exports`의 관계가 어렵게 느껴진다면 그냥 `exports`는 무시하고
`module.exports`만 사용해라.

### module.require(id)

* `id` {문자열}
* Return: {객체} 처리된 모듈의 `module.exports`

`module.require` 메소드로 모듈을 로드하면 해당 모듈에서 require()를 호출하는 것처럼 모듈을
로드한다.

이 메소드를 호출하려면 일단 `module` 객체의 참조를 얻어야 한다. `module` 객체의 참조는
해당 모듈에서만 접근할 수 있고 `require()`는 `module`이 아니라 `module.exports`를 리턴하기
때문에 해당 모듈에서 module 객체의 참조를 직접 반환해야 한다.


### module.id

* {String}

모듈 ID인데 보통은 모듈 파일의 전체 경로를 사용한다.


### module.filename

* {String}

모듈 파일의 전체 경로(fully resolved filename).


### module.loaded

* {Boolean}

모듈이 로드하고 있는 중인지 다 로드했는지를 나타낸다.


### module.parent

* {Module Object}

모듈을 require한 모듈을 가리킨다.


### module.children

* {Array}

모듈이 require한 모듈 객체를 가리킨다.



## All Together...

<!-- type=misc -->

`require()`로 모듈을 찾을 때 정확한 파일 경로가 궁금하면 `require.resolve()` 함수로 얻어온다.

require.resolve가 정확히 어떻게 동작하는지 슈도 코드로 살펴보자. 이 슈도 코드는 여태까지
설명한 것을 모두 합쳐 놓은 것이다.

    require(X) from module at path Y
    1. If X is a core module,
       a. return the core module
       b. STOP
    2. If X begins with './' or '/' or '../'
       a. LOAD_AS_FILE(Y + X)
       b. LOAD_AS_DIRECTORY(Y + X)
    3. LOAD_NODE_MODULES(X, dirname(Y))
    4. THROW "not found"

    require(X) from module at path Y
    1. If X is a core module,
       a. return the core module
       b. STOP
    2. If X begins with './' or '/' or '../'
       a. LOAD_AS_FILE(Y + X)
       b. LOAD_AS_DIRECTORY(Y + X)
    3. LOAD_NODE_MODULES(X, dirname(Y))
    4. THROW "not found"

    LOAD_AS_FILE(X)
    1. If X is a file, load X as JavaScript text.  STOP
    2. If X.js is a file, load X.js as JavaScript text.  STOP
    3. If X.json is a file, parse X.json to a JavaScript Object.  STOP
    4. If X.node is a file, load X.node as binary addon.  STOP

    LOAD_AS_DIRECTORY(X)
    1. If X/package.json is a file,
       a. Parse X/package.json, and look for "main" field.
       b. let M = X + (json main field)
       c. LOAD_AS_FILE(M)
    2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
    3. If X/index.json is a file, parse X/index.json to a JavaScript object. STOP
    4. If X/index.node is a file, load X/index.node as binary addon.  STOP

    LOAD_NODE_MODULES(X, START)
    1. let DIRS=NODE_MODULES_PATHS(START)
    2. for each DIR in DIRS:
       a. LOAD_AS_FILE(DIR/X)
       b. LOAD_AS_DIRECTORY(DIR/X)

    NODE_MODULES_PATHS(START)
    1. let PARTS = path split(START)
    2. let I = count of PARTS - 1
    3. let DIRS = []
    4. while I >= 0,
       a. if PARTS[I] = "node_modules" CONTINUE
       c. DIR = path join(PARTS[0 .. I] + "node_modules")
       b. DIRS = DIRS + DIR
       c. let I = I - 1
    5. return DIRS

## Loading from the global folders

<!-- type=misc -->

Node는 모듈을 못 찾으면 환경변수 `NODE_PATH`에 등록된 경로에서도 찾는다. 절대 경로를
`NODE_PATH`에 할당하면 되는데 콜론(`:`)으로 구분해서 절대 경로를 여러 개 등록할 수 있다(주의:
윈도우는 세미콜론(`;`)으로 구분한다).

그리고 Node는 다른 디렉터리에서도 찾는다:

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

`$HOME`은 사용자의 홈 디렉터리고 `$PREFIX`는 노드에 설정된 `node_prefix`를 말한다.

왜 그런지 말하자면 길다. 무엇보다 `node_modules` 폴더를 이용해 모듈을 로컬에 설치하는 것이 좋다.
이 방법이 속도도 더 빠르고 더 안전하다.

## Accessing the main module

<!-- type=misc -->

Node로 어떤 파일을 실행하면 `require.main`은 그 파일의 `module` 객체를 가리킨다. 그래서
Node로 파일을 직접 실행한 건지 아닌지 알 수 있다:

    require.main === module

`foo.js`라는 파일에 이런 게 들어 있다고 하자. 이 구문의 결과는 `node foo.js`로 실행하면
`true`이고 `require('./foo')`로 실행하면 `false`가 된다.

`module`에는 `filename` 프로퍼티가 있어서(`__filename`과 같은 값이다)
`require.main.filename`의 값을 확인하면 처음 실행한 파일을 무엇인지 알 수 있다.

## Addenda: Package Manager Tips

<!-- type=misc -->

`require()` 함수는 웬만한 디렉터리면 어디에서나 사용할 수 있다. `dpkg`, `rpm` 같은 패키지
매니저처럼 `npm`도 네이티브 Node 패키지를 아무런 수정 없이 빌드하게 할 수 있다.

다음은 어떻게 디렉터리를 구성해야 모듈이 제대로 동작하는지 설명한다:

모듈은 `/usr/lib/node/<some-package>/<some-version>`에 설치하는 것을 권장한다. 어떤
패키지의 어떤 버전이 설치됐는지 한 눈에 알 수 있어 좋다.

패키지는 다른 패키지에 의존할 수도 있다. 예를 들어 `foo` 패키지를 설치하려면 `bar` 패키지도
설치해야 한다. 그것도 특정 버전의 `bar` 패키지가 설치돼야 한다. 그리고 `bar` 패키지도 다른
패키지에 의존할 수 있는데 충돌이 있거나 서로(cycle) 의존할 수도 있다.

Node는 로드할 모듈을 찾을 때 `node_modules` 폴더에서 필요한 모듈을 찾는다. 그중에 심볼릭 링크가
있으면 그 링크가 가리키는 모듈도 잘 찾는다. 다음과 같이 모듈을 찾는 메커니즘은 매우 간단하다:

* `/usr/lib/node/foo/1.2.3/` - 버전이  1.2.3인 `foo` 패키지
* `/usr/lib/node/bar/4.3.2/` - `foo`가 의존하는 `bar` 패키지
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - `/usr/lib/node/bar/4.3.2/`에 대한 심볼릭 링크
* `/usr/lib/node/bar/4.3.2/node_modules/*` - `bar`가 의존하는 패키지에 대한 심볼릭 링크

그리고 상호 참조나 의존성 충돌이 있어도 모듈을 사용할 수만 있으면 잘 로드한다.

`foo` 패키지에서 `require('bar')`라고 하면
`/usr/lib/node/foo/1.2.3/node_modules/bar`가 가리키는 모듈을 가져온다. 또 그 `bar`
패키지에서 `require('quux')`라고 호출하면
`/usr/lib/node/bar/4.3.2/node_modules/quux`가 가리키는 모듈을 가져온다.

최적화된 방법으로 모듈을 찾는 방법이 있는데 `/usr/lib/node` 디렉터리가 아니라
`/usr/lib/node_modules/<name>/<version>`에 모듈을 넣는다. 그러면 Node는
`/usr/node_modules`이나 `/node_modules`에서는 모듈을 찾지 않는다.

`/usr/lib/node_modules` 폴더를 환경 변수 `$NODE_PATH`에 넣으면 Node REPL에서도 모듈을
사용할 수 있다. `require()`를 호출한 파일이 있는 곳에서부터 상대경로로 `node_modules` 폴더에
있는 모듈을 찾기 때문에 패키지는 그 `node_modules` 폴더 중 한곳에 넣으면 된다.
