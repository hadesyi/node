# DNS

    Stability: 3 - Stable

이 모듈을 접근하려면 `require('dns')`를 사용해라. 쓰레드 풀에서 `getaddrinfo(3)`를
사용하는 `dns.lookup`를 제외한 dns 모듈의 모든 메서드들은 C-Ares를 사용한다.
C-Ares가 `getaddrinfo`보다 훨신 더 빠르지만 시스템 리졸버는 다른 프로그램이 어떻게
동작하는 지에 더 관련이 있다. 사용자가 `net.connect(80, 'google.com')`나
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
            throw err;
          }

          console.log('reverse for ' + a + ': ' + JSON.stringify(domains));
        });
      });
    });

## dns.lookup(domain, [family], callback)

도메인(예시: `'google.com'`)을 처음으로 찾은 A (IPv4)나 AAAA (IPv6) 레코드로
처리한다.
`family`는 정수 `4`나 `6`이 될 수 있다. 기본값은 Ip v4와 v6 주소 패밀리 둘 다를
의미하는 `null`이다.

콜백은 `(err, address, family)` 아규먼트를 받는다. `address` 아규먼트는 IP v4나
IP v6 주소의 문자열 표현이다. `family` 아규먼트는 정수 4나 6이고 `address`의
패밀리를 나타낸다. (이 값은 필수적으로 `lookup`에 전달해야 하는 값은 아니다.)


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

`dns.resolve4()`와 같지만 IPv6 조회(`AAAA` 레코드)에만 사용한다.


## dns.resolveMx(domain, callback)

`dns.resolve()`와 같지만 메일 교환 조회(`MX` 레코드)에만 사용한다.

`addresses`는 MX 레코드의 배열이다. 각 레코드는 우선순위와 교환속성을 가진다.
(예시: `[{'priority': 10, 'exchange': 'mx.example.com'},...]`)

## dns.resolveTxt(domain, callback)

`dns.resolve()`와 같지만 텍스트 조회(`TXT` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용가능한 텍스트 레코드의 배열이다.
(예시: `['v=spf1 ip4:0.0.0.0 ~all']`)

## dns.resolveSrv(domain, callback)

`dns.resolve()`와 같지만 서비스 레코드(`SRV` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용가능한 SRV 레코드의 배열이다. SRV 레코드의 프로퍼티들은
우선순위, 중요도, 포트, 이름이다. (예시:
`[{'priority': 10, {'weight': 5, 'port': 21223, 'name': 'service.example.com'}, ...]`)

## dns.resolveNs(domain, callback)

`dns.resolve()`와 같지만 네임서버 레코드(`NS` 레코드)에만 사용한다.
`addresses`는 `domain`에서 사용가능한 네임서버 레코드의 배열이다.
(예시: `['ns1.example.com', 'ns2.example.com']`)

## dns.resolveCname(domain, callback)

`dns.resolve()`와 같지만 공인된 이름(canonical name) 레코드(`CNAME` 레코드)에만
사용한다. `addresses`는 `domain`에서 사용가능한 공인된 이름 레코드의 배열이다.
(예시: `['bar.example.com']`)

## dns.reverse(ip, callback)

반대로 ip 주소를 도메인명의 배열로 처리한다.

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
- `dns.BADNAME`: 형식이 잘못된 도메인명.
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
- `dns.NONAME`: 주어진 호스트명이 숫자가 아님.
- `dns.BADHINTS`: 허용되지 않는 힌트 플래그가 지정됨.
- `dns.NOTINITIALIZED`: c-ares 라이브러리 초기화가 아직 수행되지 않음.
- `dns.LOADIPHLPAPI`: iphlpapi.dll을 로딩하는 데 오류발생.
- `dns.ADDRGETNETWORKPARAMS`: GetNetworkParams 함수를 찾을 수 없음.
- `dns.CANCELLED`: DNS 조회가 취소됨.

