# Modules

<!--english start-->

    Stability: 5 - Locked

<!--name=module-->

Node has a simple module loading system.  In Node, files and modules are in
one-to-one correspondence.  As an example, `foo.js` loads the module
`circle.js` in the same directory.

The contents of `foo.js`:

    var circle = require('./circle.js');
    console.log( 'The area of a circle of radius 4 is '
               + circle.area(4));

The contents of `circle.js`:

    var PI = Math.PI;

    exports.area = function (r) {
      return PI * r * r;
    };

    exports.circumference = function (r) {
      return 2 * PI * r;
    };

The module `circle.js` has exported the functions `area()` and
`circumference()`.  To export an object, add to the special `exports`
object.

Variables
local to the module will be private. In this example the variable `PI` is
private to `circle.js`.

The module system is implemented in the `require("module")` module.

<!--english end-->

    안정성: 5 - Locked

<!--name=module-->

매우 간단하게 모듈을 로딩할 수 있다. 노드에서는 파일 하나가 모듈 하나다. 예를 들어 `foo.js` 파일에서 같은 디렉토리에 있는 `circle.js`를 로드하는 것을 살펴보자.

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

`circle.js` 모듈은 `area()`와 `circumference()`를 Export했다. 뭔가 Export하려면 해당 객체를 `exports` 객체에 할당한다. `exports`는 Export하기 위해 사용하는 객체다.

로컬 변수는 모듈 외부에 노출되지 않는다(private). 이 예제에서 `PI`는 `circle.js`에서만 사용할 수 있는 private 변수다.

이 모듈 시스템은 `module`이라는 모듈에 구현했다.

## Cycles

<!--english start-->

<!--type=misc-->

When there are circular `require()` calls, a module might not be
done being executed when it is returned.

Consider this situation:

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

When `main.js` loads `a.js`, then `a.js` in turn loads `b.js`.  At that
point, `b.js` tries to load `a.js`.  In order to prevent an infinite
loop an **unfinished copy** of the `a.js` exports object is returned to the
`b.js` module.  `b.js` then finishes loading, and its exports object is
provided to the `a.js` module.

By the time `main.js` has loaded both modules, they're both finished.
The output of this program would thus be:

    $ node main.js
    main starting
    a starting
    b starting
    in b, a.done = false
    b done
    in a, b.done = true
    a done
    in main, a.done=true, b.done=true

If you have cyclic module dependencies in your program, make sure to
plan accordingly.

<!--english end-->

<!--type=misc-->

두 모듈이 `require()` 함수로 서로 참조할 때는 한쪽 모듈은 아직 완전히 로딩하지 못한 미완성 모듈을 그냥 반환한다.

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

`main.js`는 `a.js`를 로드하고, `a.js`는 `b.js`를 로드한다. 여기서 `b.js`는 다시 `a.js`를 로드하려고 한다. 무한 루프가 생기지 않도록 아직 미완성인 `a.js`의 exports 객체를 `b.js`에 반환해 버린다. 그리고 `b.js`가 완성되면 `a.js`에 반환된다.

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

<!--english start-->

<!--type=misc-->

Node has several modules compiled into the binary.  These modules are
described in greater detail elsewhere in this documentation.

The core modules are defined in node's source in the `lib/` folder.

Core modules are always preferentially loaded if their identifier is
passed to `require()`.  For instance, `require('http')` will always
return the built in HTTP module, even if there is a file by that name.

<!--english end-->

<!--type=misc-->

Node 모듈 중에서는 바이너리로 컴파일해야 하는 모듈이 있다. 코어 모듈은 이 문서 곳곳에서 설명한다.

코어 모듈은 Node 소스코드의 `lib/` 폴더에 들어 있다.

모듈을 require하면 항상 코어 모듈이 먼저 로드된다. 예를 들어, `require('http')`로 로드될 것 같은 파일이 있어도 Node에 들어 있는 HTTP 모듈이 반환된다.

## File Modules

<!--english start-->

<!--type=misc-->

If the exact filename is not found, then node will attempt to load the
required filename with the added extension of `.js`, `.json`, and then `.node`.

`.js` files are interpreted as JavaScript text files, and `.json` files are
parsed as JSON text files. `.node` files are interpreted as compiled addon
modules loaded with `dlopen`.

A module prefixed with `'/'` is an absolute path to the file.  For
example, `require('/home/marco/foo.js')` will load the file at
`/home/marco/foo.js`.

A module prefixed with `'./'` is relative to the file calling `require()`.
That is, `circle.js` must be in the same directory as `foo.js` for
`require('./circle')` to find it.

Without a leading '/' or './' to indicate a file, the module is either a
"core module" or is loaded from a `node_modules` folder.

<!--english end-->

<!--type=misc-->

입력한 이름으로 파일을 못 찾으면 Node는 그 이름에 `.js`, `.json`, `.node`를 붙이고 해당 파일이 있는지 찾는다.

`.js` 파일은 JavaScript 텍스트 파일로 Interpret하고 `.json`은 JSON 텍스트 파일로 Interpret한다. 그리고 `.node` 파일은 컴파일한 addon 모듈이라서 `dlopen`으로 로드한다.

모듈을 절대 경로로 찾을 때는 모듈 이름을 `'/'`로 시작하면 된다. 예를 들어, `require('home/marco/foo.js')`는 `/home/marco/foo.js` 파일을 로드한다.

모듈을 상대 경로로 찾으려면 모듈 이름이 `'./'`로 시작하면 된다. 즉, `foo.js`라는 파일에서 `require('./circle')`라고 호출하면 같은 디렉토리에 있는 `circle.js`를 로드한다.

'/'이나 './'로 시작하지 않으면 그냥 파일이 아니라 코어 모듈이나 `node_modules` 폴더에 있는 모듈을 찾는다.

## Loading from `node_modules` Folders

<!--english start-->

<!--type=misc-->

If the module identifier passed to `require()` is not a native module,
and does not begin with `'/'`, `'../'`, or `'./'`, then node starts at the
parent directory of the current module, and adds `/node_modules`, and
attempts to load the module from that location.

If it is not found there, then it moves to the parent directory, and so
on, until the root of the tree is reached.

For example, if the file at `'/home/ry/projects/foo.js'` called
`require('bar.js')`, then node would look in the following locations, in
this order:

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

This allows programs to localize their dependencies, so that they do not
clash.

<!--english end-->

<!--type=misc-->

`require()`에 넘어온 모듈 ID가 네이티브 모듈을 가리키는 것도 아니고, 그 모듈 ID가 `'/'`, `'./'`, `'../'`로 시작하지도 않으면 Node는 그 모듈의 상위 디렉토리에서 찾기 시작한다. 상위 디렉토리에 있는 `/node_modules`에서 해당 모듈을 찾는다.

만약 못 찾으면 상위상위 디렉토리에서 찾고, 그래도 못 찾으면 상위상위상위 디렉토리에서 찾는다. 루트 디렉토리에 다다를 때까지 계속 찾는다.

예를 들어, `'home/ry/projects/foo.js'`라는 파일에서 `requre('bar.js')`라고 호출하면 다음과 같은 순서로 모듈을 찾는다:

* `/home/ry/projects/node_modules/bar.js`
* `/home/ry/node_modules/bar.js`
* `/home/node_modules/bar.js`
* `/node_modules/bar.js`

그래서 해당 프로그램만의 의존성을 독립적으로 관리할 수 있다. 다른 프로그램에 영향을 끼치지 않는다.

## Folders as Modules

<!--english start-->

<!--type=misc-->

It is convenient to organize programs and libraries into self-contained
directories, and then provide a single entry point to that library.
There are three ways in which a folder may be passed to `require()` as
an argument.

The first is to create a `package.json` file in the root of the folder,
which specifies a `main` module.  An example package.json file might
look like this:

    { "name" : "some-library",
      "main" : "./lib/some-library.js" }

If this was in a folder at `./some-library`, then
`require('./some-library')` would attempt to load
`./some-library/lib/some-library.js`.

This is the extent of Node's awareness of package.json files.

If there is no package.json file present in the directory, then node
will attempt to load an `index.js` or `index.node` file out of that
directory.  For example, if there was no package.json file in the above
example, then `require('./some-library')` would attempt to load:

* `./some-library/index.js`
* `./some-library/index.node`

<!--english end-->

<!--type=misc-->

모듈을 폴더로 관리하면 프로그램과 라이브러리를 묶음으로 관리할 수 있어 편리하다. 마치 한 파일로 된 모듈처럼 취급한다. 모듈이 폴더일 때 `require()`는 세 가지 방법으로 모듈을 찾는다.

프로그램 폴더에 `package.json` 파일을 만들고 main 모듈이 무엇인지 적는다:

    { "name" : "some-library",
      "main" : "./lib/some-library.js" }

이 파일이 `./some-library`라는 폴더에 있다고 하고, `require('./some-library')`를 호출하면 `./some-library/lib/some-library.js`를 찾아 로드한다.

Node가 package.json을 읽고 사용하기 때문에 이런 게 가능하다.

그 디렉토리에 package.json 파일이 없으면 Node는 `index.js`나 `index.node` 파일을 찾는다. package.json 파일이 없으면 `require('./some-library')`는 다음과 같은 파일을 로드한다:

* `./some-library/index.js`
* `./some-library/index.node`

## Caching

<!--english start-->

<!--type=misc-->

Modules are cached after the first time they are loaded.  This means
(among other things) that every call to `require('foo')` will get
exactly the same object returned, if it would resolve to the same file.

Multiple calls to `require('foo')` may not cause the module code to be
executed multiple times.  This is an important feature.  With it,
"partially done" objects can be returned, thus allowing transitive
dependencies to be loaded even when they would cause cycles.

If you want to have a module execute code multiple times, then export a
function, and call that function.

<!--english end-->

<!--type=misc-->

한 번 로드한 모듈은 계속 캐싱한다. 그래서 `require('foo')`을 여러 번 호출해도 계속 같은 객체를 반환한다. 단, `require('foo')가 계속 같은 파일을 로드할 때만 그렇다.

`require('foo')`를 여러 번 호출해도 해당 모듈 코드는 단 한 번만 호출된다. 그리고 아직 미완성인 객체가 반환될 수 있다는 점까지 더하면 특정 모듈이 서로 의존하고 있어도 성공적으로 로드되는 마법이 이루어진다.

어떤 코드가 꼭 여러 번 호출돼야 하면 함수 자체를 Export하고 그 함수를 여러 번 호출하라.

### Module Caching Caveats

<!--english start-->

<!--type=misc-->

Modules are cached based on their resolved filename.  Since modules may
resolve to a different filename based on the location of the calling
module (loading from `node_modules` folders), it is not a *guarantee*
that `require('foo')` will always return the exact same object, if it
would resolve to different files.

<!--english end-->

<!--type=misc-->

모듈은 찾은(resolved) 파일 이름을 키로 캐싱한다. `node_modules` 폴더에서 로딩하는 것이기 때문에 같은 require 코드라도 호출하는 위치에 따라 찾은 파일이 다를 수 있다. 즉, `require('foo')`가 다른 파일을 찾아낸다면 다른 객체를 리턴한다.

## The `module` Object

<!--english start-->

<!-- type=var -->
<!-- name=module -->

* {Object}

In each module, the `module` free variable is a reference to the object
representing the current module.  In particular
`module.exports` is the same as the `exports` object.
`module` isn't actually a global but rather local to each module.

<!--english end-->

<!-- type=var -->
<!-- name=module -->

* {Object}

모듈에서 `module` 변수는 해당 모듈 객체를 가리킨다. 특히 `module.exports`는 `exports`와 같은 객체를 가리킨다. `module`은 글로벌 변수가 아니라 모듈마다 다른 객체를 가리키는 로컬 변수다.

### module.exports

<!--english start-->

* {Object}

The `exports` object is created by the Module system. Sometimes this is not
acceptable, many want their module to be an instance of some class. To do this
assign the desired export object to `module.exports`. For example suppose we
were making a module called `a.js`

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

Then in another file we could do

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });


Note that assignment to `module.exports` must be done immediately. It cannot be
done in any callbacks.  This does not work:

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);


<!--english end-->

* {Object}

`exports` 객체는 Module 시스템이 자동으로 만들어 준다. Export하려는 객체를 `module.exports`에 할당해서 직접 만든 객체가 반환되게 할 수도 있다. `.js`라는 모듈을 만들어 보자:

    var EventEmitter = require('events').EventEmitter;

    module.exports = new EventEmitter();

    // Do some work, and after some time emit
    // the 'ready' event from the module itself.
    setTimeout(function() {
      module.exports.emit('ready');
    }, 1000);

이 모듈은 다음과 같이 사용한다:

    var a = require('./a');
    a.on('ready', function() {
      console.log('module a is ready');
    });

`module.exports`에 할당하는 것은 바로 실행되도록 해야 한다. 콜백으로 할당문이 실행되는 것을 미루면 뜻대로 동작하지 않는다. 다음과 같이 하지 마라:

x.js:

    setTimeout(function() {
      module.exports = { a: "hello" };
    }, 0);

y.js:

    var x = require('./x');
    console.log(x.a);


### module.require(id)

<!--english start-->

* `id` {String}
* Return: {Object} `exports` from the resolved module

The `module.require` method provides a way to load a module as if
`require()` was called from the original module.

Note that in order to do this, you must get a reference to the `module`
object.  Since `require()` returns the `exports`, and the `module` is
typically *only* available within a specific module's code, it must be
explicitly exported in order to be used.


<!--english end-->

* `id` {문자열}
* Return: {객체} 처리된 모듈의 `exports`

`module.require` 메소드로 모듈을 로드하면 해당 모듈에서 require()를 호출하는 것처럼 모듈을 로드한다.

이 메소드를 호출하려면 일단 `module` 객체의 레퍼런스를 얻어야 한다. `module` 객체의 레퍼런스는 해당 모듈에서만 접근할 수 있고 `require()`는 `module`이 아니라 `exports`를 리턴하기 때문에 해당 모듈에서 module 객체의 레퍼런스를 직접 리턴해야 한다.


### module.id

<!--english start-->

* {String}

The identifier for the module.  Typically this is the fully resolved
filename.


<!--english end-->

* {String}

모듈 ID인데 보통은 모듈 파일의 전체 경로를 사용한다.


### module.filename

<!--english start-->

* {String}

The fully resolved filename to the module.


<!--english end-->

* {String}

모듈 파일의 전체 경로(fully resolved filename).


### module.loaded

<!--english start-->

* {Boolean}

Whether or not the module is done loading, or is in the process of
loading.


<!--english end-->

* {Boolean}

모듈이 로드하고 있는 중인지 다 로드했는지를 나타낸다.


### module.parent

<!--english start-->

* {Module Object}

The module that required this one.


<!--english end-->

* {Module Object}

모듈을 require한 모듈을 가리킨다.


### module.children

<!--english start-->

* {Array}

The module objects required by this one.



<!--english end-->

* {Array}

모듈이 require한 모듈 객체를 가리킨다.



## All Together...

<!--english start-->

<!-- type=misc -->

To get the exact filename that will be loaded when `require()` is called, use
the `require.resolve()` function.

Putting together all of the above, here is the high-level algorithm
in pseudocode of what require.resolve does:

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
    3. If X.node is a file, load X.node as binary addon.  STOP

    LOAD_AS_DIRECTORY(X)
    1. If X/package.json is a file,
       a. Parse X/package.json, and look for "main" field.
       b. let M = X + (json main field)
       c. LOAD_AS_FILE(M)
    2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
    3. If X/index.node is a file, load X/index.node as binary addon.  STOP

    LOAD_NODE_MODULES(X, START)
    1. let DIRS=NODE_MODULES_PATHS(START)
    2. for each DIR in DIRS:
       a. LOAD_AS_FILE(DIR/X)
       b. LOAD_AS_DIRECTORY(DIR/X)

    NODE_MODULES_PATHS(START)
    1. let PARTS = path split(START)
    2. let ROOT = index of first instance of "node_modules" in PARTS, or 0
    3. let I = count of PARTS - 1
    4. let DIRS = []
    5. while I > ROOT,
       a. if PARTS[I] = "node_modules" CONTINUE
       c. DIR = path join(PARTS[0 .. I] + "node_modules")
       b. DIRS = DIRS + DIR
       c. let I = I - 1
    6. return DIRS

<!--english end-->

<!-- type=misc -->

`require()`로 모듈을 찾을 때 정확한 파일 경로가 궁금하면 `require.resolve()` 함수로 얻어온다.

require.resolve가 정확히 어떻게 동작하는지 슈도 코드로 살펴보자. 이 슈도 코드는 여태까지 설명한 것을 모두 합쳐 놓은 것이다:

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
    3. If X.node is a file, load X.node as binary addon.  STOP

    LOAD_AS_DIRECTORY(X)
    1. If X/package.json is a file,
       a. Parse X/package.json, and look for "main" field.
       b. let M = X + (json main field)
       c. LOAD_AS_FILE(M)
    2. If X/index.js is a file, load X/index.js as JavaScript text.  STOP
    3. If X/index.node is a file, load X/index.node as binary addon.  STOP

    LOAD_NODE_MODULES(X, START)
    1. let DIRS=NODE_MODULES_PATHS(START)
    2. for each DIR in DIRS:
       a. LOAD_AS_FILE(DIR/X)
       b. LOAD_AS_DIRECTORY(DIR/X)

    NODE_MODULES_PATHS(START)
    1. let PARTS = path split(START)
    2. let ROOT = index of first instance of "node_modules" in PARTS, or 0
    3. let I = count of PARTS - 1
    4. let DIRS = []
    5. while I > ROOT,
       a. if PARTS[I] = "node_modules" CONTINUE
       c. DIR = path join(PARTS[0 .. I] + "node_modules")
       b. DIRS = DIRS + DIR
       c. let I = I - 1
    6. return DIRS

## Loading from the global folders

<!--english start-->

<!-- type=misc -->

If the `NODE_PATH` environment variable is set to a colon-delimited list
of absolute paths, then node will search those paths for modules if they
are not found elsewhere.  (Note: On Windows, `NODE_PATH` is delimited by
semicolons instead of colons.)

Additionally, node will search in the following locations:

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

Where `$HOME` is the user's home directory, and `$PREFIX` is node's
configured `installPrefix`.

These are mostly for historic reasons.  You are highly encouraged to
place your dependencies locally in `node_modules` folders.  They will be
loaded faster, and more reliably.

<!--english end-->

<!-- type=misc -->

Node는 모듈을 못 찾으면 환경변수 `NODE_PATH`에 등록된 경로에서도 찾는다. 절대경로를 `NODE_PATH`에 할당하면 되는데 콜론(`:`)으로 구분해서 절대경로를 여러 개 등록할 수 있다(주의: 윈도우는 세미콜론(`;`)으로 구분한다).

그리고 Node는 다른 디렉토리에서도 찾는다:

* 1: `$HOME/.node_modules`
* 2: `$HOME/.node_libraries`
* 3: `$PREFIX/lib/node`

`$HOME`은 사용자의 홈 디렉토리이고 `$PREFIX`는 노드가 설치된 디렉토리를 말한다.

왜 그런지 말하자면 길다. 무엇보다 `node_modules` 폴더를 이용해 모듈을 로컬에 설치하는 것이 좋다. 이 방법이 속도도 더 빠르고 더 안전하다.

## Accessing the main module

<!--english start-->

<!-- type=misc -->

When a file is run directly from Node, `require.main` is set to its
`module`. That means that you can determine whether a file has been run
directly by testing

    require.main === module

For a file `foo.js`, this will be `true` if run via `node foo.js`, but
`false` if run by `require('./foo')`.

Because `module` provides a `filename` property (normally equivalent to
`__filename`), the entry point of the current application can be obtained
by checking `require.main.filename`.

<!--english end-->

<!-- type=misc -->

node로 어떤 파일을 실행하면 `require.main`은 그 파일의 `module` 객체를 가리킨다. 그래서 Node로 파일을 직접 실행한 건지 아닌지 알 수 있다:

    require.main === module

`foo.js`라는 파일에 이런 게 들어 있다고 하자. 이 구문의 결과는 `node foo.js`로 실행하면 `true`이고 `require('./foo')`로 실행하면 `false`가 된다.

`module`에는 `filename` 프로퍼티가 있어서(`__filename`과 같은 값이다) `require.main.filename`의 값을 확인하면 처음 실행한 파일을 무엇인지 알 수 있다.

## Addenda: Package Manager Tips

<!--english start-->

<!-- type=misc -->

The semantics of Node's `require()` function were designed to be general
enough to support a number of sane directory structures. Package manager
programs such as `dpkg`, `rpm`, and `npm` will hopefully find it possible to
build native packages from Node modules without modification.

Below we give a suggested directory structure that could work:

Let's say that we wanted to have the folder at
`/usr/lib/node/<some-package>/<some-version>` hold the contents of a
specific version of a package.

Packages can depend on one another. In order to install package `foo`, you
may have to install a specific version of package `bar`.  The `bar` package
may itself have dependencies, and in some cases, these dependencies may even
collide or form cycles.

Since Node looks up the `realpath` of any modules it loads (that is,
resolves symlinks), and then looks for their dependencies in the
`node_modules` folders as described above, this situation is very simple to
resolve with the following architecture:

* `/usr/lib/node/foo/1.2.3/` - Contents of the `foo` package, version 1.2.3.
* `/usr/lib/node/bar/4.3.2/` - Contents of the `bar` package that `foo`
  depends on.
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - Symbolic link to
  `/usr/lib/node/bar/4.3.2/`.
* `/usr/lib/node/bar/4.3.2/node_modules/*` - Symbolic links to the packages
  that `bar` depends on.

Thus, even if a cycle is encountered, or if there are dependency
conflicts, every module will be able to get a version of its dependency
that it can use.

When the code in the `foo` package does `require('bar')`, it will get the
version that is symlinked into `/usr/lib/node/foo/1.2.3/node_modules/bar`.
Then, when the code in the `bar` package calls `require('quux')`, it'll get
the version that is symlinked into
`/usr/lib/node/bar/4.3.2/node_modules/quux`.

Furthermore, to make the module lookup process even more optimal, rather
than putting packages directly in `/usr/lib/node`, we could put them in
`/usr/lib/node_modules/<name>/<version>`.  Then node will not bother
looking for missing dependencies in `/usr/node_modules` or `/node_modules`.

In order to make modules available to the node REPL, it might be useful to
also add the `/usr/lib/node_modules` folder to the `$NODE_PATH` environment
variable.  Since the module lookups using `node_modules` folders are all
relative, and based on the real path of the files making the calls to
`require()`, the packages themselves can be anywhere.

<!--english end-->

<!-- type=misc -->

`require()` 함수는 웬만한 디렉토리면 어디에서나 사용할 수 있다. `dpkg`, `rpm` 같은 패키지 매니저처럼 `npm`도 네이티브 Node 패키지를 아무런 수정 없이 빌드하게 할 수 있다.

다음은 어떻게 디렉토리를 구성해야 모듈이 제대로 동작하는지 설명한다:

모듈은 `/usr/lib/node/<some-package>/<some-version>`에 설치하는 것을 권장한다. 어떤 패키지의 어떤 버전이 설치됐는지 한 눈에 알 수 있어 좋다.

패키지는 다른 패키지에 의존할 수도 있다. 예를 들어 `foo` 패키지를 설치하려면 `bar` 패키지도 설치해야 한다. 그것도 특정 버전의 `bar` 패키지가 설치돼야 한다. 그리고 `bar` 패키지도 다른 패키지에 의존할 수 있는데 충돌이 있거나 서로(cycle) 의존할 수도 있다.

Node는 로드할 모듈을 찾을 때 `node_modules` 폴더에서 필요한 모듈을 찾는다. 그중에 심볼릭 링크가 있으면 그 링크가 가리키는 모듈도 잘 찾는다. 다음과 같이 모듈을 찾는 매커니즘은 매우 간단하다:

* `/usr/lib/node/foo/1.2.3/` - 버전이  1.2.3인 `foo` 패키지
* `/usr/lib/node/bar/4.3.2/` - `foo`가 의존하는 `bar` 패키지
* `/usr/lib/node/foo/1.2.3/node_modules/bar` - `/usr/lib/node/bar/4.3.2/`에 대한 심볼릭 링크
* `/usr/lib/node/bar/4.3.2/node_modules/*` - `bar`가 의존하는 패키지에 대한 심볼릭 링크

그리고 상호 참조나 의존성 충돌이 있어도 모듈을 사용할 수만 있으면 잘 로드한다.

`foo` 패키지에서 `require('bar')`라고 하면 `/usr/lib/node/foo/1.2.3/node_modules/bar`가 가리키는 모듈을 가져온다. 또 그 `bar` 패키지에서 `require('quux')`라고 호출하면 `/usr/lib/node/bar/4.3.2/node_modules/quux`가 가리키는 모듈을 가져온다.

최적화된 방법으로 모듈을 찾는 방법이 있는데 `/usr/lib/node` 디렉토리가 아니라 `/usr/lib/node_modules/<name>/<version>`에 모듈을 넣는다. 그러면 Node는 `/usr/node_modules`이나 `/node_modules`에서는 모듈을 찾지 않는다.

`/usr/lib/node_modules` 폴더를 환경 변수 `$NODE_PATH`에 넣으면 Node REPL에서도 모듈을 사용할 수 있다. `require()`를 호출한 파일이 있는 곳에서부터 상대경로로 `node_modules` 폴더에 있는 모듈을 찾기 때문에 패키지는 그 `node_modules` 폴더 중 한 곳에 넣으면 된다.
