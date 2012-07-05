# Path

<!--english start-->

    Stability: 3 - Stable

This module contains utilities for handling and transforming file
paths.  Almost all these methods perform only string transformations.
The file system is not consulted to check whether paths are valid.

`path.exists` and `path.existsSync` are the exceptions, and should
logically be found in the fs module as they do access the file system.

Use `require('path')` to use this module.  The following methods are provided:

<!--english end-->

    Stability: 3 - Stable

이 모듈에는 파일 경로를 다루고 변경하는 유틸리티가 포함되어 있다. 이 모듈 대부분의 
메서드들은 문자열 변경만 수행한다. 파일시스템은 경로가 유효한치 확인하지 않는다.

`path.exists`와 `path.existsSync`는 예외다. 이 두 메서드는 파일시스템에 접근하듯이 
fs 모듈에서 논리적으로 찾을 수 있어야 한다.

이 모듈을 사용하려면 `require('path')`를 사용해라. 다음의 메서드들이 제공된다.

## path.normalize(p)

<!--english start-->

Normalize a string path, taking care of `'..'` and `'.'` parts.

When multiple slashes are found, they're replaced by a single one;
when the path contains a trailing slash, it is preserved.
On windows backslashes are used. 

Example:

    path.normalize('/foo/bar//baz/asdf/quux/..')
    // returns
    '/foo/bar/baz/asdf'

<!--english end-->

`'..'`와 `'.'` 부분을 처리해서 문자열 경로를 정규화한다.

슬래시가 여러 개 있는 경우 슬래시 하나로 교체하고 경로의 마지막에 슬래시가 
있는 경우에는 유지한다. 
windows에서는 역슬래시를 사용한다. 

예제:

    path.normalize('/foo/bar//baz/asdf/quux/..')
    // returns
    '/foo/bar/baz/asdf'

## path.join([path1], [path2], [...])

<!--english start-->

Join all arguments together and normalize the resulting path.
Non-string arguments are ignored.

Example:

    path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
    // returns
    '/foo/bar/baz/asdf'

    path.join('foo', {}, 'bar')
    // returns
    'foo/bar'

<!--english end-->

모든 아규먼트를 합쳐서 최종 경로로 정규화한다.
문자열이 아닌 아규먼트는 무시한다.

예제:

    path.join('/foo', 'bar', 'baz/asdf', 'quux', '..')
    // returns
    '/foo/bar/baz/asdf'

    path.join('foo', {}, 'bar')
    // returns
    'foo/bar'

## path.resolve([from ...], to)

<!--english start-->

Resolves `to` to an absolute path.

If `to` isn't already absolute `from` arguments are prepended in right to left
order, until an absolute path is found. If after using all `from` paths still
no absolute path is found, the current working directory is used as well. The
resulting path is normalized, and trailing slashes are removed unless the path 
gets resolved to the root directory. Non-string arguments are ignored.

Another way to think of it is as a sequence of `cd` commands in a shell.

    path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')

Is similar to:

    cd foo/bar
    cd /tmp/file/
    cd ..
    cd a/../subfile
    pwd

The difference is that the different paths don't need to exist and may also be
files.

Examples:

    path.resolve('/foo/bar', './baz')
    // returns
    '/foo/bar/baz'

    path.resolve('/foo/bar', '/tmp/file/')
    // returns
    '/tmp/file'

    path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
    // if currently in /home/myself/node, it returns
    '/home/myself/node/wwwroot/static_files/gif/image.gif'

<!--english end-->

`to`를 절대경로로 변환한다.

`to`가 절대경로가 아니면 절대경로를 찾을 때까지 `from` 아규먼트들을 우측에서 좌측의 순서로 
앞에 이어붙힌다.모든 `from` 경로를 사용한 후에도 절대경로를 찾지 못하면 현재 워킹 디렉토리를 
사용한다. 최종 경로는 정규화되고 경로가 루트 디렉토리로 처리되지 않는한 마지막 슬래시는 제거한다. 
문자열이 아닌 아규먼트는 무시한다.

이는 쉘에서 `cd` 명령어를 순서대로 실행한 것과 같다.

    path.resolve('foo/bar', '/tmp/file/', '..', 'a/../subfile')

이는 다음과 비슷하다.:

    cd foo/bar
    cd /tmp/file/
    cd ..
    cd a/../subfile
    pwd

다른 경로라 존재할 필요가 없거나 파일일 수도 있다는 점만이 다르다.

예제:

    path.resolve('/foo/bar', './baz')
    // returns
    '/foo/bar/baz'

    path.resolve('/foo/bar', '/tmp/file/')
    // returns
    '/tmp/file'

    path.resolve('wwwroot', 'static_files/png/', '../gif/image.gif')
    // if currently in /home/myself/node, it returns
    '/home/myself/node/wwwroot/static_files/gif/image.gif'

## path.relative(from, to)

<!--english start-->

Solve the relative path from `from` to `to`.

At times we have two absolute paths, and we need to derive the relative
path from one to the other.  This is actually the reverse transform of
`path.resolve`, which means we see that:

    path.resolve(from, path.relative(from, to)) == path.resolve(to)

Examples:

    path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
    // returns
    '..\\..\\impl\\bbb'

    path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
    // returns
    '../../impl/bbb'

<!--english end-->

`from`에서 `to`까지의 상대경로를 처리한다.

때로는 두 개의 절대경로를 가지고 있고 하나에서 다른 하나로의 상대경로를 얻어야 한다. 
사실 이는 `path.resolve`의 반대 변환이다. 이 의미를 다음 예제에서 보자.

    path.resolve(from, path.relative(from, to)) == path.resolve(to)

예제:

    path.relative('C:\\orandea\\test\\aaa', 'C:\\orandea\\impl\\bbb')
    // returns
    '..\\..\\impl\\bbb'

    path.relative('/data/orandea/test/aaa', '/data/orandea/impl/bbb')
    // returns
    '../../impl/bbb'

## path.dirname(p)

<!--english start-->

Return the directory name of a path.  Similar to the Unix `dirname` command.

Example:

    path.dirname('/foo/bar/baz/asdf/quux')
    // returns
    '/foo/bar/baz/asdf'

<!--english end-->

경로의 디렉토리이름을 반환한다. Unix의 `dirname` 명령어와 비슷하다.

예제:

    path.dirname('/foo/bar/baz/asdf/quux')
    // returns
    '/foo/bar/baz/asdf'

## path.basename(p, [ext])

<!--english start-->

Return the last portion of a path.  Similar to the Unix `basename` command.

Example:

    path.basename('/foo/bar/baz/asdf/quux.html')
    // returns
    'quux.html'

    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // returns
    'quux'

<!--english end-->

경로의 마지막 부분을 반환한다. Unix의 `basename` 명령어와 비슷하다.

예제:

    path.basename('/foo/bar/baz/asdf/quux.html')
    // returns
    'quux.html'

    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // returns
    'quux'

## path.extname(p)

<!--english start-->

Return the extension of the path, from the last '.' to end of string
in the last portion of the path.  If there is no '.' in the last portion
of the path or the first character of it is '.', then it returns
an empty string.  Examples:

    path.extname('index.html')
    // returns
    '.html'

    path.extname('index.')
    // returns
    '.'

    path.extname('index')
    // returns
    ''

<!--english end-->

경로의 마지막 부분의 문자열에서 마지막 '.'에서부터 경로의 확장자를 반환한다. 
경로의 마지막 부분에 '.'가 없거나 첫 글자가 '.'이라면 빈 문자열을 반환한다.
예제:

    path.extname('index.html')
    // returns
    '.html'

    path.extname('index.')
    // returns
    '.'

    path.extname('index')
    // returns
    ''

## path.exists(p, [callback])

<!--english start-->

Test whether or not the given path exists by checking with the file system.
Then call the `callback` argument with either true or false.  Example:

    path.exists('/etc/passwd', function (exists) {
      util.debug(exists ? "it's there" : "no passwd!");
    });

<!--english end-->

파일 시스템으로 확인해서 주어진 경로가 존재하는 지 여부를 검사한다.
그 다음 true나 false 아규먼트로 `callback`을 호출한다. 예제:

    path.exists('/etc/passwd', function (exists) {
      util.debug(exists ? "it's there" : "no passwd!");
    });


## path.existsSync(p)

<!--english start-->

Synchronous version of `path.exists`.

<!--english end-->

`path.exists`의 동기 버전.
