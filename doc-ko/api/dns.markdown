# DNS

    Stability: 3 - Stable

이 모듈에 접근하려면 `require('dns')`를 사용해라.

이 모듈에는 두 분류의 함수가 있다.

1) 도메인 이름을 찾으려고 기반을 두는 운영체제의 기능을 사용하는 함수로 네트워크 통신이 전혀 필요치
않다. 이 분류에는 `dns.lookup` 하나의 함수만 있다. __같은 운영체제상의 다른 애플리케이션이
하는 것과 같은 방법으로 도메인 이름을 찾으려는 개발자는 `dns.lookup`를 사용해야 한다.__

다음은 `www.google.com`를 검색하는 예제이다.

    var dns = require('dns');

    dns.lookup('www.google.com', function onLookup(err, addresses, family) {
      console.log('addresses:', addresses);
    });

2) 도메인 이름을 찾으려고 실제 DNS 서버에 접속하는 함수로 DNS 조회를 하려고 _항상_ 네트워크를
사용한다. 이 분류에는 `dns.lookup`를 제외한 `dns` 모듈의 모든 함수가 있다. 이 함수는
`dns.lookup`이 사용하는 것처럼 같은 설정 파일들을 사용하지 않는다. 예를 들어, _이 함수들은
`/etc/hosts`의 설정을 사용하지 않는다._ 도메인 이름처리에 기반 운영체제를 사용하기를 원치
않고 _항상_ DNS 조회를 원하는 개발자가 이 함수를 사용해야 한다.

다음은 `'www.google.com'`를 처리하고 반대로 반환된 IP 주소를 도메인으로 처리하는
예제다.

    var dns = require('dns');

    dns.resolve4('www.google.com', function (err, addresses) {
      if (err) throw err;

      console.log('addresses: ' + JSON.stringify(addresses));

      addresses.forEach(function (a) {
        dns.reverse(a, function (err, domains) {
          if (err) {
            throw err;
          }

          console.log('reverse for ' + a + ': ' + JSON.stringify(domains));
        });
      });
    });

둘 중에서 선택할 때 미묘하게 결과가 다르므로 자세한 내용은
[Implementation considerations section](#dns_implementation_considerations)를
찾고 하길 바란다.

## dns.lookup(domain, [family], callback)

도메인(예시: `'google.com'`)을 처음으로 찾은 A (IPv4)나 AAAA (IPv6) 레코드로
처리한다.
`family`는 정수 `4`나 `6`이 될 수 있다. 기본값은 Ip v4와 v6 주소 패밀리 둘 다를
의미하는 `null`이다.

콜백은 `(err, address, family)` 아규먼트를 받는다. `address` 아규먼트는 IP v4나
IP v6 주소의 문자열 표현이다. `family` 아규먼트는 정수 4나 6이고 `address`의
패밀리를 나타낸다. (이 값은 필수적으로 `lookup`에 전달해야 하는 값은 아니다.)

오류 발생 시 `err`는 `Error` 객체이고 `err.code`는 오류 코드이다.
도메인이 없을 때뿐만 아니라 파일 디스크립터가 없는 등의 다른 이유로 검색이 실패했을 때도
`err.code`는 `'ENOENT'`가 된다는 것을 유념해라.

`dns.lookup`는 DNS 프로토콜과 동작해야 하는 어떤 것도 필요치 않다. 이름을 주소로 연결하거나
주소를 연결할 수 있는 운영체제의 기능만 필요할 뿐이다.

Node.js 프로그램의 동작에서 이 구현체에 따라 미묘하지만 중요한 결과가 달라질 수 있다.
사용하기 전에 시간을 내서 [Implementation considerations
section](#dns_implementation_considerations)를 꼭 읽어보기 바란다.

## dns.resolve(domain, [rrtype], callback)

도메인(예시: `'google.com'`)을 rrtype에 지정한 레코드 종류의 배열로 처리한다.
유효한 rrtype은 `'A'`(기본값인 IPV4 주소), `'AAAA'` (IPV6 주소),
`'MX'` (메일 교환 레코드), `'TXT'` (텍스트 레코드), `'SRV'` (SRV 레코드),
`'PTR'` (반대로 IP 검색에 사용된다.), `'NS'` (네임서버 레코드),
`'CNAME'` (공인된 이름(canonical name) 레코드)다.

콜백은 `(err, addresses)` 아규먼트다. `addresses`에서 각 아이템의 타입은
레코드 타입으로 결정되고 아래 대응하는 검색 메서드의 문서에서 설명한다.

오류가 있을 때 `err`는 아래 목록에 있는 오류코드 중 하나인
`err.code`가 있는 `Error` 객체다.


## dns.resolve4(domain, callback)

`dns.resolve()`와 같지만 IPv4 조회(`A` 레코드)에만 사용한다.
`addresses`는 IPv4 주소의 배열이다. (예시:
`['74.125.79.104', '74.125.79.105', '74.125.79.106']`)

## dns.resolve6(domain, callback)

`dns.resolve4()`와 같지만, IPv6 조회(`AAAA` 레코드)에만 사용한다.


## dns.resolveMx(domain, callback)

`dns.resolve()`와 같지만 메일 교환 조회(`MX` 레코드)에만 사용한다.

`addresses`는 MX 레코드의 배열이다. 각 레코드는 우선순위와 교환속성을 가진다.
(예시: `[{'priority': 10, 'exchange': 'mx.example.com'},...]`)

## dns.resolveTxt(domain, callback)

`dns.resolve()`와 같지만, 텍스트 조회(`TXT` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용 가능한 텍스트 레코드의 배열이다.
(예시: `['v=spf1 ip4:0.0.0.0 ~all']`)

## dns.resolveSrv(domain, callback)

`dns.resolve()`와 같지만 서비스 레코드(`SRV` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용 가능한 SRV 레코드의 배열이다. SRV 레코드의 프로퍼티들은
우선순위, 중요도, 포트, 이름이다. (예시:
`[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`)

## dns.resolveNs(domain, callback)

`dns.resolve()`와 같지만 네임 서버 레코드(`NS` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용 가능한 네임 서버 레코드의 배열이다.
(예시: `['ns1.example.com', 'ns2.example.com']`)

## dns.resolveCname(domain, callback)

`dns.resolve()`와 같지만 공인된 이름(canonical name) 레코드(`CNAME` 레코드)에만
사용한다. `addresses`는 `domain`에서 사용 가능한 공인된 이름 레코드의 배열이다.
(예시: `['bar.example.com']`)

## dns.reverse(ip, callback)

반대로 ip 주소를 도메인 명의 배열로 처리한다.

콜백은 `(err, domains)` 아규먼트를 받는다.

오류가 있을 때 `err`는 아래 목록에 있는 오류코드 중 하나인
`err.code`가 있는 `Error` 객체다.

## Error codes

각 DNS 조회는 다음 오류코드 중 하나를 반환할 수 있다.

- `dns.NODATA`: DNS 서버가 데이터가 없다는 응답함.
- `dns.FORMERR`: DNS 서버가 쿼리의 형식이 잘못되었다고 응답함.
- `dns.SERVFAIL`: DNS 서버가 일반적인 실패를 반환함.
- `dns.NOTFOUND`: 도메인 명을 못 찾음.
- `dns.NOTIMP`: DNS 서버가 요청된 작업을 구현하지 않음.
- `dns.REFUSED`: DNS 서버가 조회를 거절함.
- `dns.BADQUERY`: 형식이 잘못된 DNS 조회.
- `dns.BADNAME`: 형식이 잘못된 도메인 명.
- `dns.BADFAMILY`: 지원되지 않는 주소 패밀리.
- `dns.BADRESP`: 형식이 잘못된 DNS 응답.
- `dns.CONNREFUSED`: DNS 서버에 접속할 수 없음.
- `dns.TIMEOUT`: DNS 서버에 접속하는 동안 타임아웃 발생.
- `dns.EOF`: 파일의 끝.
- `dns.FILE`: 파일을 읽는 중 오류발생.
- `dns.NOMEM`: Out of memory.
- `dns.DESTRUCTION`: 채널이 파괴됨.
- `dns.BADSTR`: 형식이 잘못된 문자열.
- `dns.BADFLAGS`: 허용되지 않는 플래그가 지정됨.
- `dns.NONAME`: 주어진 호스트 명이 숫자가 아님.
- `dns.BADHINTS`: 허용되지 않는 힌트 플래그가 지정됨.
- `dns.NOTINITIALIZED`: c-ares 라이브러리 초기화가 아직 수행되지 않음.
- `dns.LOADIPHLPAPI`: iphlpapi.dll을 로딩하는 데 오류발생.
- `dns.ADDRGETNETWORKPARAMS`: GetNetworkParams 함수를 찾을 수 없음.
- `dns.CANCELLED`: DNS 조회가 취소됨.

## Implementation considerations

`dns.lookup`와 `dns.resolve*/dns.reverse` 함수가 네트워크 이름을 네트워크 주소로
연결한다는(혹은 그 반대로 연결) 같은 목적을 가지더라도 둘의 동작은 아주 다르다. 이 차이점 때문에
Node.js 프로그램에서 동작이 미묘하지만 중요하게 달라질 수 있다.

### dns.lookup

내부적으로 `dns.lookup`는 대부분의 다른 프로그램처럼 같은 운영체제의 기능을 사용한다.
예를 들어 `dns.lookup`은 `ping` 명령어와 같은 방법으로 해당 이름을 거의 항상 처리할 것이다.
대부분의 POSIX 호환 운영체제에서는 `dns.lookup`함수의 동작은 `nsswitch.conf(5)`나
`resolv.conf(5)` 설정에 따라 변경될 수 있지만, 이 파일을 변경하면 같은 운영체제의 모든
프로그램에서 동작이 달라질 것이므로 조심해야 한다.

이 호출이 JavaScript의 관점에서 비동기라고 하더라도 실제로는 libuv의 스레드 풀에서 동작하는
`getaddrinfo(3)`를 동기로 호출하는 식으로 구현된다. libuv의 스레드 풀의 크기가 고정되어 있으므로
`getaddrinfo(3)`의 호출에 긴 시간이 걸리는 이유가 무엇이든 간에 libuv의 스레드 풀에서 실행할 수
있는 다른 작업은(파일 시스템 작업 같은) 성능이 저하되리라는 것을 의미한다. 이 문제를 줄이기 위해
'UV_THREADPOOL_SIZE' 환경변수를 4보다(현재 기본값) 큰 값으로 설정해서 libuv의 스레드 풀의
크기를 증가시키는 것이 가능한 방법 중 하나이다. libuv의 스레드풀에 대한 자세한 내용은
[공식 libuv 문서](http://docs.libuv.org/en/latest/threadpool.html)를 참고해라.

### dns.resolve, functions starting with dns.resolve and dns.reverse

이 함수는 `dns.lookup`와는 아주 다르게 구현되었다. 이 함수는 `getaddrinfo(3)를 사용하지 않고
_항상_ 네트워크를 통해 DNS 조회를 한다. 이 네트워크 통신은 항상 비동기로 수행되고 libuv의
스레드 풀을 사용하지 않는다.

그래서 이 함수는 `dns.lookup`가 가질 수 있는 libuv 스레드 풀에서 발생하는 다른 작업 처리에
영향을 주는 문제가 발생할 수 없다.

`dns.lookup`가 사용하는 설정 파일들을 사용하지 않는다. 예를 들어
_이 함수는 `/etc/hosts`의 설정을 사용하지 않는다_.
