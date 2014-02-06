# punycode

    Stability: 2 - Unstable

[Punycode.js](http://mths.be/punycode)는 Node.js v0.6.2+부터 포함되었고
`require('punycode')`로 접근한다.(다른 Node.js 버전에서 이 모듈을 사용하려면
npm으로 `punycode`모듈을 설치해야 한다.)

## punycode.decode(string)

ASCII 코드의 퓨니코드(Punycode) 문자열을 유니코드 문자열로 변환한다.


    // 도메인 명 부분을 디코드한다
    punycode.decode('maana-pta'); // 'mañana'
    punycode.decode('--dqo34k'); // '☃-⌘'

## punycode.encode(string)

유니코드 문자열을 ASCII 코드의 퓨니코드(Punycode) 문자열로 변환한다.

    // 도메인명 부분을 인코딩한다
    punycode.encode('mañana'); // 'maana-pta'
    punycode.encode('☃-⌘'); // '--dqo34k'

## punycode.toUnicode(domain)

도메인 명을 나타내는 퓨니코드 문자열을 유니코드로 변환한다. 도메인 명에서 퓨니코드부분만
변환된다. 예를 들어 이미 유니코드로 변환된 문자열에서 이 함수를 호출해도 아무 문제가 없다.

    // 도메인 명을 디코드한다
    punycode.toUnicode('xn--maana-pta.com'); // 'mañana.com'
    punycode.toUnicode('xn----dqo34k.com'); // '☃-⌘.com'

## punycode.toASCII(domain)

도메인 명을 나타내는 유니코드 문자열을 퓨니코드로 변환한다. 도메인 명에서 ASCII가 아닌
부분만 변환한다. 이미 ASCII인 도메인에서 호출해도 괜찮다.

    // 도메인 명을 인코딩한다
    punycode.toASCII('mañana.com'); // 'xn--maana-pta.com'
    punycode.toASCII('☃-⌘.com'); // 'xn----dqo34k.com'

## punycode.ucs2

### punycode.ucs2.decode(string)

문자열의 각 유니코드 문자에 대한 십진수 코드를 담고 있는 배열을 생성한다. [JavaScript
가 내부적으로 UCS-2를 사용하기](http://mathiasbynens.be/notes/javascript-encoding)때문에
이 함수는 서로게이트 반쪽(surrogate halves)의 쌍을(UCS-2의 각각은 분리된 코드로 나타난다)
UTF-16에 맞는 단일 코드로 변환한다.

    punycode.ucs2.decode('abc'); // [97, 98, 99]
    // surrogate pair for U+1D306 tetragram for centre:
    punycode.ucs2.decode('\uD834\uDF06'); // [0x1D306]

### punycode.ucs2.encode(codePoints)

십진수코드 배열에 기반을 둔 문자열을 생성한다.

    punycode.ucs2.encode([97, 98, 99]); // 'abc'
    punycode.ucs2.encode([0x1D306]); // '\uD834\uDF06'

## punycode.version

현재 Punycode.js 버전번호를 나타내는 문자열.
