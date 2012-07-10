# URL

    Stability: 3 - Stable

이 모듈은 URL 처리와 파싱에 관한 유틸리티 모듈이다.
사용하려면 `require('url')`를 호출해라.

파싱된 URL 객체들은 다음의 필드들을 가지고 있다. 필드들은 URL 문자열에 존재여부에 따라 
있을 수도 있고 없을 수도 있다. URL 문자열에 없는 부분은 파싱된 객체에도 없다.
다음은 URL에 대한 예제를 보여준다.

`'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `href`: 파싱되기 전 원래의 전체 URL. 프로톨콜과 호스트가 모두 소문자이다.

    예제: `'http://user:pass@host.com:8080/p/a/t/h?query=string#hash'`

* `protocol`: 요청 프로토콜로 소문자다.

    예제: `'http:'`

* `host`: 포트정보를 포함해서 소문자로된 호스트 부분.

    예제: `'host.com:8080'`

* `auth`: URL의 인증정보 부분.

    예제: `'user:pass'`

* `hostname`: 호스트에서 소문자로 된 호스트명 부분.

    예제: `'host.com'`

* `port`: 호스트의 포트번호 부분.

    예제: `'8080'`

* `pathname`: 존재한다면 첫 슬래시를 포함해서 호스트 이후부터 쿼리 이전까지의
  URL의 경로부분.

    예제: `'/p/a/t/h'`

* `search`: URL에서 물음표로 시작되는 'query string' 부분.

    예제: `'?query=string'`

* `path`: `pathname`와 `search`의 연결.

    예제: `'/p/a/t/h?query=string'`

* `query`: 쿼리스트링의 'params' 부분이거나 쿼리스트링이 파싱된 객체다.

    예제: `'query=string'` or `{'query':'string'}`

* `hash`: 해시기호를 포함해서 URL의 'fragment' 부분.

    예제: `'#hash'`

URL 모듈에서 다음 메서드들을 제공한다.

## url.parse(urlStr, [parseQueryString], [slashesDenoteHost])

URL 문자열을 받아서 객체를 반환한다.

`querystring` 모듈을 사용해서 쿼리스트링을 파싱하려면 
두번째 아규먼트로 `true`를 전달한다.
기본값은 `false`이다.

`{ pathname: '//foo/bar' }`보다는 
`{ host: 'foo', pathname: '/bar' }`와 같이  `//foo/bar`를 
다루려면 세번째 아규먼트로 `true`를 전달한다.
기본값은 `false`이다.

## url.format(urlObj)

파싱된 URL 객체를 받아서 포매팅된 URL 문자열을 반환한다.

* `href`는 무시된다.
* `protocol`은 뒷 부분의 `:` (콜론)을 포함하거나 포함하지 않고 다룬다.
  * 프로토콜 `http`, `https`, `ftp`, `gopher`, `file`은 
    `://` (콜론-슬래시-슬래시)로 접미사가 붙는다.
  * 모든 다른 프로토콜 `mailto`, `xmpp`, `aim`, `sftp`, `foo` 등은 `:` 
    (콜론)로 접미사가 붙는다.
* 존재하는 경우`auth`를 사용한다.
* `host`가 없으면 `hostname`만 사용한다.
* `host`가 없으면 `port`를 사용한다.
* `host`를 `auth`, `hostname`, `port` 대신에 사용한다.
* `pathname`는 `/` (슬래시)로 시작되는 부분을 포함하거나 포함하지 않고 다룬다.
* `search`를 `query` 대신에 사용한다.
* `search`가 없으면 `query` (객체; `querystring`를 봐라)만 사용할 것이다.
* `search`는 `?` (물음표)로 시작되는 부분을 포함하거나 포함하지 않고 다룬다.
* `hash`는 `#` (해시기호, 앵커)로 시작하는 부분을 포함하거나 포함하지 않고 다룬다.

## url.resolve(from, to)

기준 URL과 href URL을 받아서 앵커테그를 위해서 브라우저처럼 처리한다.
