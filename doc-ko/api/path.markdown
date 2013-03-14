# Path

    Stability: 3 - Stable

이 모듈에는 파일 경로를 다루고 변경하는 유틸리티가 포함되어 있다. 이 모듈 대부분의
메서드들은 문자열 변경만 수행한다. 경로가 유효한지 확인하는데 파일 시스템이 관여하지 않는다.

이 모듈을 사용하려면 `require('path')`를 사용해라. 다음의 메서드들이 제공된다.

## path.normalize(p)

`'..'`와 `'.'` 부분을 처리해서 문자열 경로를 정규화한다.

슬래시가 여러 개 있는 경우 슬래시 하나로 교체하고 경로의 마지막에 슬래시가
있는 경우에는 유지한다.
windows에서는 역슬래시를 사용한다.

예제:

    path.normalize('/foo/bar//baz/asdf/quux/..')
    // returns
    '/foo/bar/baz/asdf'

## path.join([path1], [path2], [...])

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

경로의 디렉토리이름을 반환한다. Unix의 `dirname` 명령어와 비슷하다.

예제:

    path.dirname('/foo/bar/baz/asdf/quux')
    // returns
    '/foo/bar/baz/asdf'

## path.basename(p, [ext])

경로의 마지막 부분을 반환한다. Unix의 `basename` 명령어와 비슷하다.

예제:

    path.basename('/foo/bar/baz/asdf/quux.html')
    // returns
    'quux.html'

    path.basename('/foo/bar/baz/asdf/quux.html', '.html')
    // returns
    'quux'

## path.extname(p)

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

## path.sep

플랫폼의 파일 구분자. `'\\'`나 `'/'`이다.

*nix의 예제:

    'foo/bar/baz'.split(path.sep)
    // returns
    ['foo', 'bar', 'baz']

윈도우즈의 예제:

    'foo\\bar\\baz'.split(path.sep)
    // returns
    ['foo', 'bar', 'baz']

## path.delimiter

플랫폼에 특화된 경로 구분자인 `;`나 `':'`이다.

*nix의 예제:

    console.log(process.env.PATH)
    // '/usr/bin:/bin:/usr/sbin:/sbin:/usr/local/bin'

    process.env.PATH.split(path.delimiter)
    // returns
    ['/usr/bin', '/bin', '/usr/sbin', '/sbin', '/usr/local/bin']

윈도우즈의 예제:

    console.log(process.env.PATH)
    // 'C:\Windows\system32;C:\Windows;C:\Program Files\nodejs\'

    process.env.PATH.split(path.delimiter)
    // returns
    ['C:\Windows\system32', 'C:\Windows', 'C:\Program Files\nodejs\']
