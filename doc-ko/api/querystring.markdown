# Query String

    Stability: 3 - Stable

<!--name=querystring-->

이 모듈은 쿼리스트링을 다루는 유틸리티 모듈이다.
다음의 메서드들을 제공한다.

## querystring.stringify(obj, [sep], [eq])

객체를 쿼리스트링으로 직렬화한다.
선택적으로 기본 구분기호(`'&'`)와 할당기호(`'='`)를 오버라이드 할 수 있다.

예제:

    querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' })
    // returns
    'foo=bar&baz=qux&baz=quux&corge='

    querystring.stringify({foo: 'bar', baz: 'qux'}, ';', ':')
    // returns
    'foo:bar;baz:qux'

## querystring.parse(str, [sep], [eq], [options])

쿼리스트링을 객체로 역직렬화한다.
선택적으로 기본 구분기호(`'&'`)와 할당기호(`'='`)를 오버라이드 할 수 있다.

options 객체는 `maxKeys` 프로퍼티를 가지고 있고(기본값은 1000이다) 이 프로퍼티는 처리되는
키를 제한하는데 사용한다. 키의 갯수 제한을 없애려면 0으로 설정한다.

예제:

    querystring.parse('foo=bar&baz=qux&baz=quux&corge')
    // returns
    { foo: 'bar', baz: ['qux', 'quux'], corge: '' }

## querystring.escape

`querystring.stringify`에서 사용하는 이스케이스 함수로 
필요하다면 오버라이드 할 수 있다.

## querystring.unescape

`querystring.parse`에서 사용하는 역이스케이스(unescape) 함수로 
필요하다면 오버라이드 할 수 있다.
