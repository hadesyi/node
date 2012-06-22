# DNS

    Stability: 3 - Stable

Use `require('dns')` to access this module. All methods in the dns module
use C-Ares except for `dns.lookup` which uses `getaddrinfo(3)` in a thread
pool. C-Ares is much faster than `getaddrinfo` but the system resolver is
more constant with how other programs operate. When a user does
`net.connect(80, 'google.com')` or `http.get({ host: 'google.com' })` the
`dns.lookup` method is used. Users who need to do a large number of look ups
quickly should use the methods that go through C-Ares.

Here is an example which resolves `'www.google.com'` then reverse
resolves the IP addresses which are returned.

    var dns = require('dns');

    dns.resolve4('www.google.com', function (err, addresses) {
      if (err) throw err;

      console.log('addresses: ' + JSON.stringify(addresses));

      addresses.forEach(function (a) {
        dns.reverse(a, function (err, domains) {
          if (err) {
            console.log('reverse for ' + a + ' failed: ' +
              err.message);
          } else {
            console.log('reverse for ' + a + ': ' +
              JSON.stringify(domains));
          }
        });
      });
    });

    안정성: 3 - Stable

이 모듈을 접근하려면 `require('dns')`를 사용해라. 쓰레드 풀에서 `getaddrinfo(3)`를 
사용하는 `dns.lookup`를 제외한 dns 모듈의 모든 메서드들은 C-Ares를 사용한다.
C-Ares가 `getaddrinfo`보다 훨신 더 빠르지만 시스템 리졸버는 다른 프로그램이 어떻게 
동작하는 지에 더 관려이 있다. 사용자가 `net.connect(80, 'google.com')`나 
`http.get({ host: 'google.com' })`를 사용하면 `dns.lookup`메서드가 사용된다. 
빠르게 다량의 검색을 해야하는 사용자들은 C-Ares를 쓰는 메서드를 사용해야한다. 

다음은 `'www.google.com'`를 처리하고 반대로 반환된 IP 주소를 도메인으로 처리하는 
예제다.

    var dns = require('dns');

    dns.resolve4('www.google.com', function (err, addresses) {
      if (err) throw err;

      console.log('addresses: ' + JSON.stringify(addresses));

      addresses.forEach(function (a) {
        dns.reverse(a, function (err, domains) {
          if (err) {
            console.log('reverse for ' + a + ' failed: ' +
              err.message);
          } else {
            console.log('reverse for ' + a + ': ' +
              JSON.stringify(domains));
          }
        });
      });
    });

## dns.lookup(domain, [family], callback)

Resolves a domain (e.g. `'google.com'`) into the first found A (IPv4) or
AAAA (IPv6) record.
The `family` can be the integer `4` or `6`. Defaults to `null` that indicates
both Ip v4 and v6 address family.

The callback has arguments `(err, address, family)`.  The `address` argument
is a string representation of a IP v4 or v6 address. The `family` argument
is either the integer 4 or 6 and denotes the family of `address` (not
necessarily the value initially passed to `lookup`).

도메인(예시: `'google.com'`)을 처음으로 찾은 A (IPv4)나 AAAA (IPv6) 레코드로 
처리한다.
`family`는 정수 `4`나 `6`이 될 수 있다. 기본값은 Ip v4와 v6 주소 패밀리 둘 다를 
의미하는 `null`이다.

콜백은 `(err, address, family)` 아규먼트를 받는다. `address` 아규먼트는 IP v4나
IP v6 주소의 문자열 표현이다. `family` 아규먼트는 정수 4나 6이고 `address`의 
패밀리를 나타낸다. (이 값은 필수적으로 `lookup`에 전달해야 하는 값은 아니다.)


## dns.resolve(domain, [rrtype], callback)

Resolves a domain (e.g. `'google.com'`) into an array of the record types
specified by rrtype. Valid rrtypes are `'A'` (IPV4 addresses, default),
`'AAAA'` (IPV6 addresses), `'MX'` (mail exchange records), `'TXT'` (text
records), `'SRV'` (SRV records), `'PTR'` (used for reverse IP lookups),
`'NS'` (name server records) and `'CNAME'` (canonical name records).

The callback has arguments `(err, addresses)`.  The type of each item
in `addresses` is determined by the record type, and described in the
documentation for the corresponding lookup methods below.

On error, `err` would be an instanceof `Error` object, where `err.errno` is
one of the error codes listed below and `err.message` is a string describing
the error in English.

도메인(예시: `'google.com'`)을 rrtype에 지정한 레코드 종류의 배열로 처리한다. 
유효한 rrtype은 `'A'`(기본값인 IPV4 주소), `'AAAA'` (IPV6 주소), 
`'MX'` (메일 교환 레코드), `'TXT'` (텍스트 레코드), `'SRV'` (SRV 레코드), 
`'PTR'` (반대로 IP 검색에 사용된다.), `'NS'` (네임서버 레코드), 
`'CNAME'` (공인된 이름(canonical name) 레코드)다.

콜백은 `(err, addresses)` 아규먼트다. `addresses`에서 각 아이템의 타입은 
레코드 타입으로 결정되고 아래 대응하는 검색 메서드의 문서에서 설명한다.

`err`는 아래 목록에 있는 오류코드 중 하나인 `err.errno`와 영어로 오류를 묘사하는 
문자열인 `err.message`를 갖고 있는 `Error` 객체의 인스턴스다.


## dns.resolve4(domain, callback)

The same as `dns.resolve()`, but only for IPv4 queries (`A` records).
`addresses` is an array of IPv4 addresses (e.g.
`['74.125.79.104', '74.125.79.105', '74.125.79.106']`).

`dns.resolve()`와 같지만 IPv4 조회(`A` 레코드)에만 사용한다.
`addresses`는 IPv4 주소의 배열이다. (예시: 
`['74.125.79.104', '74.125.79.105', '74.125.79.106']`)

## dns.resolve6(domain, callback)

The same as `dns.resolve4()` except for IPv6 queries (an `AAAA` query).

`dns.resolve4()`와 같지만 IPv6 조회(`AAAA` 레코드)에만 사용한다.


## dns.resolveMx(domain, callback)

The same as `dns.resolve()`, but only for mail exchange queries (`MX` records).

`addresses` is an array of MX records, each with a priority and an exchange
attribute (e.g. `[{'priority': 10, 'exchange': 'mx.example.com'},...]`).

`dns.resolve()`와 같지만 메일 교환 조회(`MX` 레코드)에만 사용한다.

`addresses`는 MX 레코드의 배열이다. 각 레코드는 우선순위와 교환속성을 가진다.
(예시: `[{'priority': 10, 'exchange': 'mx.example.com'},...]`)

## dns.resolveTxt(domain, callback)

The same as `dns.resolve()`, but only for text queries (`TXT` records).
`addresses` is an array of the text records available for `domain` (e.g.,
`['v=spf1 ip4:0.0.0.0 ~all']`).

`dns.resolve()`와 같지만 텍스트 조회(`TXT` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용가능한 텍스트 레코드의 배열이다. 
(예시: `['v=spf1 ip4:0.0.0.0 ~all']`)

## dns.resolveSrv(domain, callback)

The same as `dns.resolve()`, but only for service records (`SRV` records).
`addresses` is an array of the SRV records available for `domain`. Properties
of SRV records are priority, weight, port, and name (e.g.,
`[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`).

`dns.resolve()`와 같지만 서비스 레코드(`SRV` 레코드)에만 사용한다. 
`addresses`는 `domain`에서 사용가능한 SRV 레코드의 배열이다. SRV 레코드의 프로퍼티들은 
우선순위, 중요도, 포트, 이름이다. (예시: 
`[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`)

## dns.reverse(ip, callback)

Reverse resolves an ip address to an array of domain names.

The callback has arguments `(err, domains)`.

반대로 ip 주소를 도메인명의 배열로 처리한다.

콜백은 `(err, domains)` 아규먼트를 받는다.

## dns.resolveNs(domain, callback)

The same as `dns.resolve()`, but only for name server records (`NS` records).
`addresses` is an array of the name server records available for `domain`
(e.g., `['ns1.example.com', 'ns2.example.com']`).

`dns.resolve()`와 같지만 네임서버 레코드(`NS` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용가능한 네임서버 레코드의 배열이다. 
(예시: `['ns1.example.com', 'ns2.example.com']`)

## dns.resolveCname(domain, callback)

The same as `dns.resolve()`, but only for canonical name records (`CNAME`
records). `addresses` is an array of the canonical name records available for
`domain` (e.g., `['bar.example.com']`).

If there an an error, `err` will be non-null and an instanceof the Error
object.

Each DNS query can return an error code.

- `dns.TEMPFAIL`: timeout, SERVFAIL or similar.
- `dns.PROTOCOL`: got garbled reply.
- `dns.NXDOMAIN`: domain does not exists.
- `dns.NODATA`: domain exists but no data of reqd type.
- `dns.NOMEM`: out of memory while processing.
- `dns.BADQUERY`: the query is malformed.

`dns.resolve()`와 같지만 공인된 이름(canonical name) 레코드(`CNAME` 레코드)에만 
사용한다. `addresses`는 `domain`에서 사용가능한 공인된 이름 레코드의 배열이다. 
(예시: `['bar.example.com']`)

오류가 있다면 `err`는 null이 아니고 Error 객체의 인스턴스다.

각 DNS 조회는 오류코드를 반환할 수 있다.

- `dns.TEMPFAIL`: 타임아웃, SERVFAIL이거나 비슷한.
- `dns.PROTOCOL`: 왜곡된 응답을 받았다.
- `dns.NXDOMAIN`: 도메인이 존재하지 않는다.
- `dns.NODATA`: 도메인은 존재하지만 reqd 타입의 데이터가 없다.
- `dns.NOMEM`: 처리 중에 메모리가 부족했다.
- `dns.BADQUERY`: 쿼리가 잘못되었다.
