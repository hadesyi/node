# Buffer

<!--english start-->

    Stability: 3 - Stable

Pure Javascript is Unicode friendly but not nice to binary data.  When
dealing with TCP streams or the file system, it's necessary to handle octet
streams. Node has several strategies for manipulating, creating, and
consuming octet streams.

Raw data is stored in instances of the `Buffer` class. A `Buffer` is similar
to an array of integers but corresponds to a raw memory allocation outside
the V8 heap. A `Buffer` cannot be resized.

The `Buffer` class is a global, making it very rare that one would need
to ever `require('buffer')`.

Converting between Buffers and JavaScript string objects requires an explicit
encoding method.  Here are the different string encodings.

* `'ascii'` - for 7 bit ASCII data only.  This encoding method is very fast, and
  will strip the high bit if set.
  Note that this encoding converts a null character (`'\0'` or `'\u0000'`) into
  `0x20` (character code of a space). If you want to convert a null character
  into `0x00`, you should use `'utf8'`.

* `'utf8'` - Multi byte encoded Unicode characters.  Many web pages and other document formats use UTF-8.

* `'ucs2'` - 2-bytes, little endian encoded Unicode characters. It can encode
  only BMP(Basic Multilingual Plane, U+0000 - U+FFFF).

* `'base64'` - Base64 string encoding.

* `'binary'` - A way of encoding raw binary data into strings by using only
  the first 8 bits of each character. This encoding method is deprecated and
  should be avoided in favor of `Buffer` objects where possible. This encoding
  will be removed in future versions of Node.

* `'hex'` - Encode each byte as two hexidecimal characters.

<!--english end-->

    안정성: 3 - Stable

자바스크립트 자체는 유니코드에 친화적이지만 바이너리 데이터에는 별로 좋지 않다. TCP 
스트림이나 파일시스템을 다룰 때 옥텟(octet) 스트림을 다룰 필요가 있다. Node에는
옥텟 스트림을 조작하고, 생성하고, 소비하는 여러 전략이 있다.

로우(raw) 데이터는 `Buffer`클래스의 인스턴스에 저장된다. `Buffer`는 정수(integer)
배열과 비슷하지만 V8 heap 외부에 할당된 로우 메모리에 대응된다. `Buffer`는 크기를
다시 조정할 수 없다.

`Buffer`클래스는 전역범위로 `require('buffer')`가 필요한 경우는 매우 흔치 않다.

버퍼와 자바스크립트 문자열 객체간에 변환을 하려면 명시적인 인코딩 메서드가 필요하다.
여러 가지 문자열 인코딩이 있다.

* `'ascii'` - 7 비트 ASCII 데이터 전용이다. 이 인코딩 메서드는 아주 빠르고 7비트가 넘는 
  비트가 있는 경우 제거한다. 
  이 인코딩은 null 문자(`'\0'`나 `'\u0000'`)를 `0x20` (공백의 문자코드)로 변환한다.
  null 문자를 `0x00`로 변환하려면 `'utf8'`를 사용해야 한다.

* `'utf8'` - 멀티 바이트로 인코딩된 유니코드 문자다. 다수의 웹페이지와 문서 형식을 UTF-8을 사용한다.

* `'ucs2'` - 2 바이트, 리들 엔디언(little endian)으로 인코딩된 유니코드 문자이다. 
  BMP(Basic Multilingual Plane, U+0000 - U+FFFF)만 인코딩할 수 있다.

* `'base64'` - Base64 문자열 인코딩.

* `'binary'` - 각 문자의 첫 8 비트만을 사용해서 로우(raw) 바이너리 데이터를 문자열로
  인코딩하는 방법이다. 이 인코딩 메서드는 폐기되었고 `Buffer` 객체가 가능한 곳에서 사용하지
  말아야 한다. 이 인코딩은 Node의 차기 버전에서는 제거될 것이다.

* `'hex'` - 각 바이트를 두 16진수 문자로 인코딩한다.

## Class: Buffer

<!--english start-->

The Buffer class is a global type for dealing with binary data directly.
It can be constructed in a variety of ways.

<!--english end-->

Buffer 클래스는 바이너리 데이터를 직접 다루는 글로벌 타입니다.
다양한 방법으로 생성할 수 있다.

### new Buffer(size)

<!--english start-->

* `size` Number

Allocates a new buffer of `size` octets.

<!--english end-->

* `size` 숫자

`size` 옥텟의 새로운 버퍼를 할당한다.

### new Buffer(array)

<!--english start-->

* `array` Array

Allocates a new buffer using an `array` of octets.

<!--english end-->

* `array` 배열

옥텟의 `array`를 사용해서 새로운 버퍼를 할당한다.

### new Buffer(str, [encoding])

<!--english start-->

* `str` String - string to encode.
* `encoding` String - encoding to use, Optional.

Allocates a new buffer containing the given `str`.
`encoding` defaults to `'utf8'`.

<!--english end-->

* `str` 문자열 - 인코딩할 문자열.
* `encoding` 문자열 - 사용할 인코딩(선택사항).

주어진 `str`를 담고있는 새로운 버퍼를 할당한다.
`encoding`의 기본값은 `'utf8'`이다.

### buf.write(string, [offset], [length], [encoding])

<!--english start-->

* `string` String - data to be written to buffer
* `offset` Number, Optional, Default: 0
* `length` Number, Optional, Default: `buffer.length - offset`
* `encoding` String, Optional, Default: 'utf8'

Writes `string` to the buffer at `offset` using the given encoding.
`offset` defaults to `0`, `encoding` defaults to `'utf8'`. `length` is
the number of bytes to write. Returns number of octets written. If `buffer` did
not contain enough space to fit the entire string, it will write a partial
amount of the string. `length` defaults to `buffer.length - offset`.
The method will not write partial characters.

    buf = new Buffer(256);
    len = buf.write('\u00bd + \u00bc = \u00be', 0);
    console.log(len + " bytes: " + buf.toString('utf8', 0, len));

The number of characters written (which may be different than the number of
bytes written) is set in `Buffer._charsWritten` and will be overwritten the
next time `buf.write()` is called.


<!--english end-->

* `string` 문자열 - 버퍼에 작성할 데이터
* `offset` 숫자, 선택사항, 기본값: 0
* `length` 숫자, 선택사항, 기본값: `buffer.length - offset`
* `encoding` 문자열, 선택사항, 기본값: 'utf8'

주어진 인코딩을 사용해서 버퍼의 `offset`위치에 `string`을 작성한다. 
`offset`의 기본값은 `0`이고 `encoding`의 기본값은 `'utf8'`이다. `length`는
작성할 바이트의 수이다. 작성된 옥텟의 수를 반환한다. 전체 문자열을 작성하기에 
`buffer`가 충분한 공간을 가지고 있지 않다면 문자열의 일부만 작성할 것이다.
`length`의 기본값은 `buffer.length - offset`이다. 이 메서드는 문자의 일부만 
작성하지는 않는다.

    buf = new Buffer(256);
    len = buf.write('\u00bd + \u00bc = \u00be', 0);
    console.log(len + " bytes: " + buf.toString('utf8', 0, len));

작성한 문자의 수(작성한 바이트의 수와는 다를 수 있다)는 `Buffer._charsWritten`에
설정되고 `buf.write()`를 다음 번에 호출했을 때 덮어써질 것이다.


### buf.toString([encoding], [start], [end])

<!--english start-->

* `encoding` String, Optional, Default: 'utf8'
* `start` Number, Optional, Default: 0
* `end` Number, Optional, Default: `buffer.length`

Decodes and returns a string from buffer data encoded with `encoding`
(defaults to `'utf8'`) beginning at `start` (defaults to `0`) and ending at
`end` (defaults to `buffer.length`).

See `buffer.write()` example, above.

<!--english end-->

* `encoding` 문자열, 선택사항, 기본값: 'utf8'
* `start` 숫자, 선택사항, 기본값: 0
* `end` 숫자, 선택사항, 기본값: `buffer.length`

`start`(기본값은 `0`)부터 `end` (기본값은 `buffer.length`)까지 `encoding`로
인코딩된 버퍼 데이터를 디코딩해서 문자열을 리턴한다.

`buffer.write()` 예제를 봐라.


### buf[index]

<!--english start-->

<!--type=property-->
<!--name=[index]-->

Get and set the octet at `index`. The values refer to individual bytes,
so the legal range is between `0x00` and `0xFF` hex or `0` and `255`.

Example: copy an ASCII string into a buffer, one byte at a time:

    str = "node.js";
    buf = new Buffer(str.length);

    for (var i = 0; i < str.length ; i++) {
      buf[i] = str.charCodeAt(i);
    }

    console.log(buf);

    // node.js

<!--english end-->

<!--type=property-->
<!--name=[index]-->

`index`위치의 옥텟을 가져오거나 설정한다. 이 값들은 개별적인 바이트를 참조하므로
`0x00`부터 `0xFF`의 16진수나 `0`부터 `255` 사이의 적법한 범위이다.

예제: ASCII 문자열을 한번에 한 바이트씩 버퍼로 복사한다.

    str = "node.js";
    buf = new Buffer(str.length);

    for (var i = 0; i < str.length ; i++) {
      buf[i] = str.charCodeAt(i);
    }

    console.log(buf);

    // node.js

### Class Method: Buffer.isBuffer(obj)

<!--english start-->

* `obj` Object
* Return: Boolean

Tests if `obj` is a `Buffer`.

<!--english end-->

* `obj` 객체
* 반환타입: 불리언

`obj`가 `Buffer`인지 확인한다.

### Class Method: Buffer.byteLength(string, [encoding])

<!--english start-->

* `string` String
* `encoding` String, Optional, Default: 'utf8'
* Return: Number

Gives the actual byte length of a string. `encoding` defaults to `'utf8'`.
This is not the same as `String.prototype.length` since that returns the
number of *characters* in a string.

Example:

    str = '\u00bd + \u00bc = \u00be';

    console.log(str + ": " + str.length + " characters, " +
      Buffer.byteLength(str, 'utf8') + " bytes");

    // ½ + ¼ = ¾: 9 characters, 12 bytes

<!--english end-->

* `string` 문자열
* `encoding` 문자열, 선택사항, 기본값: 'utf8'
* 반환타입: 숫자

문자열의 실제 바이트 길이를 리턴한다. `encoding`의 기본값은 `'utf8'`이다.
`String.prototype.length`는 스트링에서 *문자*의 수를 리턴하기 때문에 
`String.prototype.length`와 이 메서드는 같지 않다.

예제:

    str = '\u00bd + \u00bc = \u00be';

    console.log(str + ": " + str.length + " characters, " +
      Buffer.byteLength(str, 'utf8') + " bytes");

    // ½ + ¼ = ¾: 9 characters, 12 bytes

### buf.length

<!--english start-->

* Number

The size of the buffer in bytes.  Note that this is not necessarily the size
of the contents. `length` refers to the amount of memory allocated for the
buffer object.  It does not change when the contents of the buffer are changed.

    buf = new Buffer(1234);

    console.log(buf.length);
    buf.write("some string", "ascii", 0);
    console.log(buf.length);

    // 1234
    // 1234

<!--english end-->

* 숫자

바이트로 나타낸 버퍼 크기. 이는 반드시 내용의 크기인 것은 아니다.
`length`는 버퍼 객체가 할당된 메모리의 양을 참조한다. 
버퍼의 내용이 변경되었을 때도 변경되지 않는다.

    buf = new Buffer(1234);

    console.log(buf.length);
    buf.write("some string", "ascii", 0);
    console.log(buf.length);

    // 1234
    // 1234

### buf.copy(targetBuffer, [targetStart], [sourceStart], [sourceEnd])

<!--english start-->

* `targetBuffer` Buffer object - Buffer to copy into
* `targetStart` Number, Optional, Default: 0
* `sourceStart` Number, Optional, Default: 0
* `sourceEnd` Number, Optional, Default: `buffer.length`

Does copy between buffers. The source and target regions can be overlapped.
`targetStart` and `sourceStart` default to `0`.
`sourceEnd` defaults to `buffer.length`.

Example: build two Buffers, then copy `buf1` from byte 16 through byte 19
into `buf2`, starting at the 8th byte in `buf2`.

    buf1 = new Buffer(26);
    buf2 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
      buf2[i] = 33; // ASCII !
    }

    buf1.copy(buf2, 8, 16, 20);
    console.log(buf2.toString('ascii', 0, 25));

    // !!!!!!!!qrst!!!!!!!!!!!!!


<!--english end-->

* `targetBuffer` Buffer 객체 - 복사할 Buffer다
* `targetStart` 숫자, 선택사항, 기본값: 0
* `sourceStart` 숫자, 선택사항, 기본값: 0
* `sourceEnd` 숫자, 선택사항, 기본값: `buffer.length`

버퍼들간에 복사를 한다. 소스영역과 타겟영역은 일치할 수도 있다.
`targetStart`와 `sourceStart`의 기본값은 `0`이다.
`sourceEnd`의 기본값은 `buffer.length`이다.

예제: 두 버퍼를 만들고 `buf1`의 16 바이트부터 19 바이트까지를 `buf2`의 
8번째 바이트위치에 복사한다. 

    buf1 = new Buffer(26);
    buf2 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
      buf2[i] = 33; // ASCII !
    }

    buf1.copy(buf2, 8, 16, 20);
    console.log(buf2.toString('ascii', 0, 25));

    // !!!!!!!!qrst!!!!!!!!!!!!!


### buf.slice([start], [end])

<!--english start-->

* `start` Number, Optional, Default: 0
* `end` Number, Optional, Default: `buffer.length`

Returns a new buffer which references the same memory as the old, but offset
and cropped by the `start` (defaults to `0`) and `end` (defaults to
`buffer.length`) indexes.

**Modifying the new buffer slice will modify memory in the original buffer!**

Example: build a Buffer with the ASCII alphabet, take a slice, then modify one
byte from the original Buffer.

    var buf1 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
    }

    var buf2 = buf1.slice(0, 3);
    console.log(buf2.toString('ascii', 0, buf2.length));
    buf1[0] = 33;
    console.log(buf2.toString('ascii', 0, buf2.length));

    // abc
    // !bc

<!--english end-->

* `start` 숫자, 선택사항, 기본값: 0
* `end` 숫자, 선택사항, 기본값: `buffer.length`

기존의 버퍼가 참조하던 메모리와 같은 메모리를 참조하지만 `start` (기본값은 `0`)부터
`end` (기본값은 `buffer.length`)의 인덱스로 잘려진 새로운 버퍼를 리턴한다.

**새로운 버퍼 부분(slice)을 변경하면 기존 버퍼의 메모리를 변경할 것이다.!**

예제: ASCII 알파벳으로 버퍼를 만들고 slice를 한 뒤 기존 버퍼의 한 바이트를
수정한다.

    var buf1 = new Buffer(26);

    for (var i = 0 ; i < 26 ; i++) {
      buf1[i] = i + 97; // 97 is ASCII a
    }

    var buf2 = buf1.slice(0, 3);
    console.log(buf2.toString('ascii', 0, buf2.length));
    buf1[0] = 33;
    console.log(buf2.toString('ascii', 0, buf2.length));

    // abc
    // !bc

### buf.readUInt8(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads an unsigned 8 bit integer from the buffer at the specified offset.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Example:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    for (ii = 0; ii < buf.length; ii++) {
      console.log(buf.readUInt8(ii));
    }

    // 0x3
    // 0x4
    // 0x23
    // 0x42

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼의 지정한 offset에서 기호가 없은 8비트 정수(unsigned 8 bit integer)를 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

예제:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    for (ii = 0; ii < buf.length; ii++) {
      console.log(buf.readUInt8(ii));
    }

    // 0x3
    // 0x4
    // 0x23
    // 0x42

### buf.readUInt16LE(offset, [noAssert])
### buf.readUInt16BE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads an unsigned 16 bit integer from the buffer at the specified offset with
specified endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Example:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt16BE(0));
    console.log(buf.readUInt16LE(0));
    console.log(buf.readUInt16BE(1));
    console.log(buf.readUInt16LE(1));
    console.log(buf.readUInt16BE(2));
    console.log(buf.readUInt16LE(2));

    // 0x0304
    // 0x0403
    // 0x0423
    // 0x2304
    // 0x2342
    // 0x4223

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼의 지정한 offset에서 지정한 엔디언(endian) 형식으로 기호가 없는 16비트 
정수(unsigned 16 bit integer)를 읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

예제:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt16BE(0));
    console.log(buf.readUInt16LE(0));
    console.log(buf.readUInt16BE(1));
    console.log(buf.readUInt16LE(1));
    console.log(buf.readUInt16BE(2));
    console.log(buf.readUInt16LE(2));

    // 0x0304
    // 0x0403
    // 0x0423
    // 0x2304
    // 0x2342
    // 0x4223

### buf.readUInt32LE(offset, [noAssert])
### buf.readUInt32BE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads an unsigned 32 bit integer from the buffer at the specified offset with
specified endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Example:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt32BE(0));
    console.log(buf.readUInt32LE(0));

    // 0x03042342
    // 0x42230403

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼의 지정한 offset에서 지정한 엔디언(endian) 형식으로 기호가 없는 
32비트 정수(unsigned 32 bit integer)를 읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

예제:

    var buf = new Buffer(4);

    buf[0] = 0x3;
    buf[1] = 0x4;
    buf[2] = 0x23;
    buf[3] = 0x42;

    console.log(buf.readUInt32BE(0));
    console.log(buf.readUInt32LE(0));

    // 0x03042342
    // 0x42230403

### buf.readInt8(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads a signed 8 bit integer from the buffer at the specified offset.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Works as `buffer.readUInt8`, except buffer contents are treated as two's
complement signed values.

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼에서 지정한 offset에서 기호가 있는 8비트 정수(signed 8 bit integer)를 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

`buffer.readUInt8`와 같이 동작하지만 버퍼의 내용을 2가지 완전한 기호가 있는 값으로 
다룬다는 점이 다르다.

### buf.readInt16LE(offset, [noAssert])
### buf.readInt16BE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads a signed 16 bit integer from the buffer at the specified offset with
specified endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Works as `buffer.readUInt16*`, except buffer contents are treated as two's
complement signed values.

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼에서 지정한 offset에서 기호가 있는 16비트 정수(signed 16 bit integer)를 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

`buffer.readUInt16*`와 같이 동작하지만 버퍼의 내용을 2가지 완전한 기호가 있는 값으로 
다룬다는 점이 다르다.

### buf.readInt32LE(offset, [noAssert])
### buf.readInt32BE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads a signed 32 bit integer from the buffer at the specified offset with
specified endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Works as `buffer.readUInt32*`, except buffer contents are treated as two's
complement signed values.

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼에서 지정한 offset에서 기호가 있는 32비트 정수(signed 32 bit integer)를 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

`buffer.readUInt32*`와 같이 동작하지만 버퍼의 내용을 2가지 완전한 기호가 있는 값으로 
다룬다는 점이 다르다.

### buf.readFloatLE(offset, [noAssert])
### buf.readFloatBE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads a 32 bit float from the buffer at the specified offset with specified
endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Example:

    var buf = new Buffer(4);

    buf[0] = 0x00;
    buf[1] = 0x00;
    buf[2] = 0x80;
    buf[3] = 0x3f;

    console.log(buf.readFloatLE(0));

    // 0x01

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼의 지정한 offset에서 지정한 엔디언(endian) 형식으로 32비트 소수(32 bit float)를 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

예제:

    var buf = new Buffer(4);

    buf[0] = 0x00;
    buf[1] = 0x00;
    buf[2] = 0x80;
    buf[3] = 0x3f;

    console.log(buf.readFloatLE(0));

    // 0x01

### buf.readDoubleLE(offset, [noAssert])
### buf.readDoubleBE(offset, [noAssert])

<!--english start-->

* `offset` Number
* `noAssert` Boolean, Optional, Default: false
* Return: Number

Reads a 64 bit double from the buffer at the specified offset with specified
endian format.

Set `noAssert` to true to skip validation of `offset`. This means that `offset`
may be beyond the end of the buffer. Defaults to `false`.

Example:

    var buf = new Buffer(8);

    buf[0] = 0x55;
    buf[1] = 0x55;
    buf[2] = 0x55;
    buf[3] = 0x55;
    buf[4] = 0x55;
    buf[5] = 0x55;
    buf[6] = 0xd5;
    buf[7] = 0x3f;

    console.log(buf.readDoubleLE(0));

    // 0.3333333333333333

<!--english end-->

* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false
* 반환타입: 숫자

버퍼의 지정한 offset에서 지정한 엔디언(endian) 형식으로 64 bit double을 
읽는다.

`offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 `offset`이
버퍼의 끝을 넘어갈 수도 있다는 의미다. 기본값은 `false`다.

예제:

    var buf = new Buffer(8);

    buf[0] = 0x55;
    buf[1] = 0x55;
    buf[2] = 0x55;
    buf[3] = 0x55;
    buf[4] = 0x55;
    buf[5] = 0x55;
    buf[6] = 0xd5;
    buf[7] = 0x3f;

    console.log(buf.readDoubleLE(0));

    // 0.3333333333333333

### buf.writeUInt8(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset. Note, `value` must be a
valid unsigned 8 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Example:

    var buf = new Buffer(4);
    buf.writeUInt8(0x3, 0);
    buf.writeUInt8(0x4, 1);
    buf.writeUInt8(0x23, 2);
    buf.writeUInt8(0x42, 3);

    console.log(buf);

    // <Buffer 03 04 23 42>

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버터의 지정한 offset에 `value`를 작성한다. `value`는 반드시 유효한 기호가 없은 8비트
정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

예제:

    var buf = new Buffer(4);
    buf.writeUInt8(0x3, 0);
    buf.writeUInt8(0x4, 1);
    buf.writeUInt8(0x23, 2);
    buf.writeUInt8(0x42, 3);

    console.log(buf);

    // <Buffer 03 04 23 42>

### buf.writeUInt16LE(value, offset, [noAssert])
### buf.writeUInt16BE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid unsigned 16 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Example:

    var buf = new Buffer(4);
    buf.writeUInt16BE(0xdead, 0);
    buf.writeUInt16BE(0xbeef, 2);

    console.log(buf);

    buf.writeUInt16LE(0xdead, 0);
    buf.writeUInt16LE(0xbeef, 2);

    console.log(buf);

    // <Buffer de ad be ef>
    // <Buffer ad de ef be>

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`을 작성한다.
`value`는 반드시 유효한 기호가 없는 16비트 정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

예제:

    var buf = new Buffer(4);
    buf.writeUInt16BE(0xdead, 0);
    buf.writeUInt16BE(0xbeef, 2);

    console.log(buf);

    buf.writeUInt16LE(0xdead, 0);
    buf.writeUInt16LE(0xbeef, 2);

    console.log(buf);

    // <Buffer de ad be ef>
    // <Buffer ad de ef be>

### buf.writeUInt32LE(value, offset, [noAssert])
### buf.writeUInt32BE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid unsigned 32 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Example:

    var buf = new Buffer(4);
    buf.writeUInt32BE(0xfeedface, 0);

    console.log(buf);

    buf.writeUInt32LE(0xfeedface, 0);

    console.log(buf);

    // <Buffer fe ed fa ce>
    // <Buffer ce fa ed fe>

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값  : false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`을 작성한다.
`value`는 반드시 유효한 기호가 없는 32비트 정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

예제:

    var buf = new Buffer(4);
    buf.writeUInt32BE(0xfeedface, 0);

    console.log(buf);

    buf.writeUInt32LE(0xfeedface, 0);

    console.log(buf);

    // <Buffer fe ed fa ce>
    // <Buffer ce fa ed fe>

### buf.writeInt8(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset. Note, `value` must be a
valid signed 8 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt8`, except value is written out as a two's complement
signed integer into `buffer`.

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 `value`를 작성한다. `value`는 반드시 유효하고 기호가 있는 8비트
정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

`buffer.writeUInt8`와 같이 동작하지만 `buffer`에 값을 2가지 완전한 기호가 있는 정수로
작성한다는 점이 다르다.

### buf.writeInt16LE(value, offset, [noAssert])
### buf.writeInt16BE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid signed 16 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt16*`, except value is written out as a two's
complement signed integer into `buffer`.

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`를 작성한다. `value`는 
반드시 유효하고 기호가 있는 16비트 정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

`buffer.writeUInt16*`와 같이 동작하지만 `buffer`에 값을 2가지 완전한 기호가 있는 정수로
작성한다는 점이 다르다.

### buf.writeInt32LE(value, offset, [noAssert])
### buf.writeInt32BE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid signed 32 bit integer.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Works as `buffer.writeUInt32*`, except value is written out as a two's
complement signed integer into `buffer`.

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`를 작성한다. `value`는 
반드시 유효하고 기호가 있는 32비트 정수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

`buffer.writeUInt32*`와 같이 동작하지만 `buffer`에 값을 2가지 완전한 기호가 있는 정수로
작성한다는 점이 다르다.

### buf.writeFloatLE(value, offset, [noAssert])
### buf.writeFloatBE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid 32 bit float.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Example:

    var buf = new Buffer(4);
    buf.writeFloatBE(0xcafebabe, 0);

    console.log(buf);

    buf.writeFloatLE(0xcafebabe, 0);

    console.log(buf);

    // <Buffer 4f 4a fe bb>
    // <Buffer bb fe 4a 4f>

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`를 작성한다. `value`는 
반드시 유효한 32비트 실수여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

예제:

    var buf = new Buffer(4);
    buf.writeFloatBE(0xcafebabe, 0);

    console.log(buf);

    buf.writeFloatLE(0xcafebabe, 0);

    console.log(buf);

    // <Buffer 4f 4a fe bb>
    // <Buffer bb fe 4a 4f>

### buf.writeDoubleLE(value, offset, [noAssert])
### buf.writeDoubleBE(value, offset, [noAssert])

<!--english start-->

* `value` Number
* `offset` Number
* `noAssert` Boolean, Optional, Default: false

Writes `value` to the buffer at the specified offset with specified endian
format. Note, `value` must be a valid 64 bit double.

Set `noAssert` to true to skip validation of `value` and `offset`. This means
that `value` may be too large for the specific function and `offset` may be
beyond the end of the buffer leading to the values being silently dropped. This
should not be used unless you are certain of correctness. Defaults to `false`.

Example:

    var buf = new Buffer(8);
    buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    // <Buffer 43 eb d5 b7 dd f9 5f d7>
    // <Buffer d7 5f f9 dd b7 d5 eb 43>

<!--english end-->

* `value` 숫자
* `offset` 숫자
* `noAssert` 불리언, 선택사항, 기본값: false

버퍼의 지정한 offset에 지정한 엔디언(endian) 형식으로 `value`를 작성한다. `value`는 
반드시 유효한 64비트 더블이여야 한다.

`value`와 `offset`의 유효성 검사를 건너뛰려면 `noAssert`을 true로 설정한다. 이 말은 
지정한 함수에 `value`가 너무 크거나 `offset`이 버퍼의 끝을 넘어가서 값들이 어떤 경고없이
버려질 수 있다는 것을 의미한다. 확실히 정확함을 유지할 수 없다면 사용하지 말아야 한다.
기본값은 `false`다.

예제:

    var buf = new Buffer(8);
    buf.writeDoubleBE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    buf.writeDoubleLE(0xdeadbeefcafebabe, 0);

    console.log(buf);

    // <Buffer 43 eb d5 b7 dd f9 5f d7>
    // <Buffer d7 5f f9 dd b7 d5 eb 43>

### buf.fill(value, [offset], [end])

<!--english start-->

* `value`
* `offset` Number, Optional
* `end` Number, Optional

Fills the buffer with the specified value. If the `offset` (defaults to `0`)
and `end` (defaults to `buffer.length`) are not given it will fill the entire
buffer.

    var b = new Buffer(50);
    b.fill("h");

<!--english end-->

* `value`
* `offset` 숫자, 선택사항
* `end` 숫자, 선택사항

버퍼를 지정한 값으로 채운다. `offset` (기본값은 `0`)과 `end` (기본값은 `buffer.length`)를 
전달하지 않으면 전체 버퍼를 채울 것이다.

    var b = new Buffer(50);
    b.fill("h");

## buffer.INSPECT_MAX_BYTES

<!--english start-->

* Number, Default: 50

How many bytes will be returned when `buffer.inspect()` is called. This can
be overridden by user modules.

Note that this is a property on the buffer module returned by
`require('buffer')`, not on the Buffer global, or a buffer instance.

<!--english end-->

* 숫자, 기본값: 50

`buffer.inspect()`가 호출되었을 때 얼마나 많은 바이트가 반환될 것인가를 지정한다. 
이 값은 사용자 모듈에서 오버라이드할 수 있다.

이 값은 Buffer 전역객체가 아니라 `require('buffer')`에서 반환되는 버퍼모듈이나 
버퍼 인스턴스의 프로퍼티이다.

## Class: SlowBuffer

<!--english start-->

This class is primarily for internal use.  JavaScript programs should
use Buffer instead of using SlowBuffer.

In order to avoid the overhead of allocating many C++ Buffer objects for
small blocks of memory in the lifetime of a server, Node allocates memory
in 8Kb (8192 byte) chunks.  If a buffer is smaller than this size, then it
will be backed by a parent SlowBuffer object.  If it is larger than this,
then Node will allocate a SlowBuffer slab for it directly.

<!--english end-->

이 클래스는 주로 내부에서 사용한다. 자바스크립트 프로그램은 SlowBuffer 대신 
Buffer를 사용해야 한다.

서버가 운영되는 동안 메모리의 작은 블럭에 많은 C++ Buffer 객체들을 할당하는 오버헤드를
피하려고 Node는 8Kb (8192 byte) 청크에 메모리를 할당한다. 버퍼가 이 크기보다 작으면
부모 SlowBuffer 객체에가 보완할 것이다. 버퍼가 이 크기보다 크면 Node는 직접적으로
버퍼에 SlowBuffer slab을 할당할 것이다.
