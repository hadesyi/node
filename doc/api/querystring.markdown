# Query String

    Stability: 3 - Stable

<!--name=querystring-->

This module provides utilities for dealing with query strings.
It provides the following methods:

    안정성: 3 - Stable

<!--name=querystring-->

이 모듈은 쿼리스트링을 다루는 유틸리티 모듈이다.
다음의 메서드들을 제공한다.

## querystring.stringify(obj, [sep], [eq])

Serialize an object to a query string.
Optionally override the default separator (`'&'`) and assignment (`'='`)
characters.

Example:

    querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' })
    // returns
    'foo=bar&baz=qux&baz=quux&corge='

    querystring.stringify({foo: 'bar', baz: 'qux'}, ';', ':')
    // returns
    'foo:bar;baz:qux'

객체를 쿼리스트링으로 직렬화한다.
선택적으로 기본 구분기호(`'&'`)와 할당기호(`'='`)를 오버라이드 할 수 있다.

예제:

    querystring.stringify({ foo: 'bar', baz: ['qux', 'quux'], corge: '' })
    // returns
    'foo=bar&baz=qux&baz=quux&corge='

    querystring.stringify({foo: 'bar', baz: 'qux'}, ';', ':')
    // returns
    'foo:bar;baz:qux'

## querystring.parse(str, [sep], [eq])

Deserialize a query string to an object.
Optionally override the default separator (`'&'`) and assignment (`'='`)
characters.

Example:

    querystring.parse('foo=bar&baz=qux&baz=quux&corge')
    // returns
    { foo: 'bar', baz: ['qux', 'quux'], corge: '' }

쿼리스트링을 객체로 역직렬화한다.
선택적으로 기본 구분기호(`'&'`)와 할당기호(`'='`)를 오버라이드 할 수 있다.

예제:

    querystring.parse('foo=bar&baz=qux&baz=quux&corge')
    // returns
    { foo: 'bar', baz: ['qux', 'quux'], corge: '' }

## querystring.escape

The escape function used by `querystring.stringify`,
provided so that it could be overridden if necessary.

`querystring.stringify`에서 사용하는 이스케이스 함수로 
필요하다면 오버라이드 할 수 있다.

## querystring.unescape

The unescape function used by `querystring.parse`,
provided so that it could be overridden if necessary.

`querystring.parse`에서 사용하는 역이스케이스(unescape) 함수로 
필요하다면 오버라이드 할 수 있다.
