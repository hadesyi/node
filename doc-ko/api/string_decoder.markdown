# StringDecoder

    Stability: 3 - Stable

이 모듈은 `require('string_decoder')`로 사용한다. StringDecoder는 버퍼를
문자열로 디코딩한다. `buffer.toString()`의 간단한 인터페이스이지만 utf8에 대한
추가적인 지원을 한다.

    var StringDecoder = require('string_decoder').StringDecoder;
    var decoder = new StringDecoder('utf8');

    var cent = new Buffer([0xC2, 0xA2]);
    console.log(decoder.write(cent));

    var euro = new Buffer([0xE2, 0x82, 0xAC]);
    console.log(decoder.write(euro));

## Class: StringDecoder

기본값이 `utf8`인 `encoding` 아규먼트를 받는다.

### decoder.write(buffer)

디코딩한 문자열을 반환한다.

### decoder.end()

버퍼에 남아있는 바이트를 반환한다.
