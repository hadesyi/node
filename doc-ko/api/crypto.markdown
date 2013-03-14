# Crypto

    Stability: 2 - Unstable; 차기 버전에서 API 변경을 논의 중이다.
    호환성을 깨뜨리는 정도의 변경은 최소화할 것이다. 하단을 참고해라.

이 모듈에 접근하려면 `require('crypto')`를 사용해라.

crypto 모듈은 안정한 HTTPS net이나 http 연결에서 사용되는
안정한 인증서를 캡슐화하는 방법을 제공한다.

OpenSSL의 hash, hmac, cipher, decipher, sign, verify 메서드의 랩퍼(wrapper)도
제공한다.

메서드에 대한 래퍼(wrapper) 세트를 제공한다.


## crypto.getCiphers()

지원하는 암호화 이름의 배열을 반환한다.

예제:

    var ciphers = crypto.getCiphers();
    console.log(ciphers); // ['AES128-SHA', 'AES256-SHA', ...]


## crypto.getHashes()

지원하는 해시 알고리즘 이름의 배열을 반환한다.

예제:

    var hashes = crypto.getHashes();
    console.log(hashes); // ['sha', 'sha1', 'sha1WithRSAEncryption', ...]


## crypto.createCredentials(details)

인증서 객체를 생성한다. 선택사항인 details는 키를 가진 딕셔너리가 된다.

* `pfx` : PFX나 PKCS12로 암호화된 개인키, 인증서, CA 증명서를 담고 있는 문자열이나 버퍼
* `key` : PEM으로 암호화된 개인키를 담고 있는 문자열
* `passphrase` : 개인키나 pfx에 대한 암호문(passphrase) 문자열
* `cert` : PEM으로 암호화된 증명서를 담고 있는 문자열
* `ca` : PEM으로 암호화된 믿을 수 있는 CA 증명서의 문자열이나 리스트
* `crl` : PEM으로 암호화된 CRL(Certificate Revocation List)의 문자열 혹은 문자열의 리스트
* `ciphers`: 사용하거나 제외할 암호(cipher)를 설명하는 문자열. 자세한 형식은
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>를 참조해라.

'ca'에 대한 세부내용을 전달하지 않는다면 node.js는 공개적으로 신뢰할 수 있는 CA의 리스트를 기본으로
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>에서
받아서 사용할 것이다.


## crypto.createHash(algorithm)

해시 객체를 생성하고 반환한다. 전달한 algorithm의 암호화 해시는 해시 다이제스트를
생성하는 데 사용할 수 있다.

`algorithm`은 플랫폼상의 OpenSSL 버전에서 사용할 수 있는 알고리즘에 의존한다.
`'sha1'`, `'md5'`, `'sha256'`, `'sha512'`등이 있다. 최근 릴리즈에서는
`openssl list-message-digest-algorithms`로 사용할 수 있는 다이제스트 알로리즘을 볼 수 있다.

예제: 이 프로그램은 파일의 sha1 합을 생성한다.

    var filename = process.argv[2];
    var crypto = require('crypto');
    var fs = require('fs');

    var shasum = crypto.createHash('sha1');

    var s = fs.ReadStream(filename);
    s.on('data', function(d) {
      shasum.update(d);
    });

    s.on('end', function() {
      var d = shasum.digest('hex');
      console.log(d + '  ' + filename);
    });

## Class: Hash

데이터의 해시 다이제스트를 생성하는 클래스다.

읽고 쓸 수 있는 [stream](stream.html)이다. 해시를 계산하는데 작성된 데이터를 사용한다.
스트림의 작성가능한 쪽이 종료되고 나면 계산된 해시 다이제스트를 얻는데 `read()` 메서드를
사용해라. 레거시 `update`와 `digest` 메서드도 지원한다.

`crypto.createHash`가 반환하는 클래스다.

### hash.update(data, [input_encoding])

전달한 `input_encoding`의 `data`로 해시 내용을 갱신한다.
전달한 `input_encoding` 인코딩은 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼라고 가정한다.

이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출될 수 있다.

### hash.digest([encoding])

해시되어야 하는 전달한 데이터의 모든 다이제스트를 계산한다.
`encoding`은 `'hex'`, `'binary'`, `'base64'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼를 반환한다.

Note: `hash` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createHmac(algorithm, key)

hmac 객체를 생성하고 반환한다. 전달한 algorithm과 key의 암호화 해시이다.

이는 읽고 쓸 수 있는 [stream](stream.html)이다. hmac을 계산하는데 쓰여진 데이터를 사용한다.
스트림의 쓰기 가능한 쪽이 종료되고 나면 계산한 다이제스트를 얻는데 `read()` 메서드를 사용해라.
레거시 `update`와 `digest` 메서드도 지원한다.

`algorithm`는 OpenSSL에서 사용할 수 있는 알고리즘에 의존한다. -- 위의 createHash를 봐라.
`key`는 사용할 hmac 키이다.

## Class: Hmac

암호화 hmac 내용을 생성하는 클래스다.

`crypto.createHmac`가 리턴하는 클래스다.

### hmac.update(data)

전달한 `data`로 hmac 내용을 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러번 호출될 수 있다.

### hmac.digest([encoding])

hmac이 되어야 하는 전달한 데이터의 모든 다이제스트를 계산한다.
`encoding`은 `'hex'`, `'binary'`, `'base64'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼를 반환한다.

Note: `hmac` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createCipher(algorithm, password)

전달한 algorithm과 password로 암호화한 암호화 객체를 생성하고 반환한다.

`algorithm`는 OpenSSL에 의존적이다. `'aes192'` 등이 있다.
OpenSSL 최근 릴리즈에서는 `openssl list-cipher-algorithms`로 사용할 수 있는
암호화 알고리즘을 볼 수 있다.
`password`는 key와 IV를 얻는데 사용하고 반드시 `'binary'`로 인코딩된 문자열이나
[buffer](buffer.html)이어야 한다.

이는 읽고 쓸 수 있는 [stream](stream.html)이다. 해시를 계산하는데 작성된 데이터를 사용한다.
스트림의 쓰기 가능한 쪽이 종료되면 계산된 해시 다이제스트를 얻을 때 `read()` 메서드를 사용해라.
레거시 `update`와 `digest` 메서드도 지원한다.

## crypto.createCipheriv(algorithm, key, iv)

전달한 algorithm, key, iv로 암호화된 암호화 객체를 생성하고 반환한다.

`algorithm`은 createCipher()`의 `algorithm`와 같다. `key`는 algorithm에서 사용하는 로우 키(raw key)
이다. `iv`는 [초기화
벡터(Initialization vector)](http://en.wikipedia.org/wiki/Initialization_vector)이다.

`key`와 `iv`는 반드시 `'binary'`로 인코딩된
문자열이나 [buffers](buffer.html)여야 한다.

## Class: Cipher

데이터를 암호화하는 클래스이다.

`crypto.createCipher`와 `crypto.createCipheriv`가 반환하는 객체다.

Cipher 객체는 읽고 쓸 수 있는 [streams](stream.html)이다. 읽기 가능한 쪽에서
암호화된 데이터를 생성할 때 작성한 평범한 텍스트 데이터를 사용한다. 레거시 `update`와
`final`메서드도 지원한다.

### cipher.update(data, [input_encoding], [output_encoding])

전달한 `input_encoding`의 인코딩의 `data`로 cipher를 갱신한다.
`input_encoding`는 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼라고 가정한다.

`output_encoding`는 암호화된 데이터의 출력형식을 지정하고 `'binary'`, `'base64'`,
`'hex'`가 될 수 있다. 기본값은 `'binary'`이다.

암호화된 내용을 반환하고 스트림처럼 새로운 데이터로 여러번 호출할 수 있다.

### cipher.final([output_encoding])

`'binary'`, `'base64'`, `'hex'`중의 하나인 `output_encoding`로 남아있는 모든
암호화된 내용을 반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

Note: `cipher` 객체는 `final()` 메서드를 호출한 후에는 사용할 수 없다.

### cipher.setAutoPadding(auto_padding=true)

입력데이터의 자동 패딩을 사용하지 않고 블럭 크기를 사용한다. `auto_padding`가 false이면 전체 입력데이터의 길이는 cipher의 블럭 크기의 배수가 되어야 하고 그렇지 않으면 `final`는 실패할 것이다.
표준이 아닌 패딩은 유용한데 예를 들어 PKCS 패딩 대신 `0x0`를 사용할 수 있다. 이 함수는 반드시 `cipher.final` 이전에 호출해야 한다.


## crypto.createDecipher(algorithm, password)

전달한 algorithm와 key로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipher()][]의 반영이다.

## crypto.createDecipheriv(algorithm, key, iv)

전달한 algorithm, key, iv로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipheriv()][]의 반영이다.

## Class: Decipher

데이터를 복호화하는 클래스다.

`crypto.createDecipher`와 `crypto.createDecipheriv`가 반환하는 클래스다.

Decipher 객체는 읽고 쓸 수 있는 [streams](stream.html)이다. 읽기 가능한 쪽에서
평범한 텍스트 데이터를 생성할 때 작성한 암호화된 데이터를 사용한다. 레거시 `update`와
`final`메서드도 지원한다.

### decipher.update(data, [input_encoding], [output_encoding])

'binary'`, `'base64'`, `'hex'`로 인코딩된 `data`로 decipher를 갱신한다.
인코딩을 지정하지 않으면 버퍼라고 가정한다.

`output_decoding`는 반환할 복호화된 평문의 형식을 지정한다. `'binary'`, `'ascii'`,
`'utf8'`가 될 수 있고 버퍼를 지정하지 않으면 버퍼를 반환한다.

### decipher.final([output_encoding])

`'binary'`, `'ascii'`, `'utf8'`중에 하나가 될 수 있는 `output_encoding`로 남아있는
모든 복호화된 평문을 반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

Note: `decipher` 객체는 `final()` 메서드가 호출된 후에는 사용할 수 없다.

### decipher.setAutoPadding(auto_padding=true)

표준 블럭 패팅없이 암호화된 데이터를 `decipher.final`가 확인하고 제거하지 않도록 자동 패딩을 사용하지 않을 수 있다. 입력데이터의 길이가 cipher 블락 크기의 배수일 때만 동작한다.
이 함수는 반드시 데이터를 `decipher.update`로 스트리밍하기 전에 호출해야 한다.


## crypto.createSign(algorithm)

전달한 algorithm으로 서명된 객체를 생성하고 반환한다.
최근 OpenSSL 릴리즈에서 `openssl list-public-key-algorithms`로 사용할 수 있는
서명된 알고리즘을 볼 수 있다. 예를 들면 `'RSA-SHA256'`가 있다.

## Class: Sign

서명을 생성하는 클래스다.

`crypto.createSign`가 반환하는 클래스이다.

Sign 객체는 쓸 수 있는 [streams](stream.html)이다. 사그니처를 생성할 때 작성한 데이터를
사용한다. 모든 데이터를 쓰고 나면 `sign` 메서드가 시그니처를 반환할 것이다.
레거시 `update` 메서드도 지원한다.

### sign.update(data)

data로 sign 객체를 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### sign.sign(private_key, [output_format])

전달한 갱신 데이터 모두를 sign을 통해서 서명을 계산한다.
`private_key`는 서명에 사용할 PEM 인코딩된 개인키를 담고 있는 문자열이다.

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 `output_format`의 서명을 반환한다.
인코딩을 지정하지 않으면 버퍼를 반환한다.

Note: `sign` 객체는 `sign()` 메서드를 호출한 후에는 사용할 수 없다.

## crypto.createVerify(algorithm)

전달한 algorithm으로 검증 객체를 생성하고 반환한다.
이 함수는 위의 서명객체의 반영이다.

## Class: Verify

서명을 검증하는 클래스다.

`crypto.createVerify`가 반환하는 클래스이다.

Verify 객체는 쓰기 가능한 [streams](stream.html)이다. 제공된 시그니처로 유효성 검사를
할 때 작성한 데이터를 사용한다. 모든 데이터를 쓰고 나서 제공된 시그니처가 유효한 경우
`verify` 메서드가 true를 반환할 것이다. 레거시 `update` 메서드도 지원한다.

### verifier.update(data)

data로 verifier 객체를 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### verifier.verify(object, signature, [signature_format])

`object`와 `signature`를 사용해서 서명된 데이터를 검증한다. `object`는 RSA 공개키,
DSA 공개키, X.509 인증서 중 하나가 될 수 있는 PEM으로 인코딩된 객체를 담고 있는 문자열이다.
`signature`는 `'binary'`, `'hex'`, `'base64'`가 될 수 있는 `signature_format`의
데이터에 대해 이전에 계산한 서명이다.
인코딩을 지정하지 않으면 버퍼라고 가정한다.

데이터와 공개키에 대한 서명의 유효성에 따라 true나 false를 반환한다.

Note: `verifier` 객체는 `verify()` 메서드를 호출한 뒤에는 사용할 수 없다.

## crypto.createDiffieHellman(prime_length)

Diffie-Hellman 키 교환 객체를 생성하고 전달한 비트 길이의 소수를 생성한다.
사용된 제너레이터는 `2`이다.

## crypto.createDiffieHellman(prime, [encoding])

제공된 소수를 사용해서 Diffie-Hellman 키 교환 객체를 생성한다. 사용된 제너레이터는
`2`이다. 인코딩은 `'binary'`, `'hex'`, `'base64'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼라고 가정한다.

## Class: DiffieHellman

Diffie-Hellman 키 교환을 생성하는 클래스이다.

`crypto.createDiffieHellman`가 반환하는 클래스이다.

### diffieHellman.generateKeys([encoding])

개인 Diffie-Hellman 키값과 공개 Diffie-Hellman 키값을 생성하고 지정한 인코딩으로
공개키를 반환한다. 이 키는 다른 관련자에게 이동할 수 있다. 인코딩은 `'binary'`,
`'hex'`, `'base64'`가 될 수 있다.
인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.computeSecret(other_public_key, [input_encoding], [output_encoding])

다른 관련자의 공개키로 `other_public_key`를 사용하는 공유 시크릿을 계산하고 계산된
공유 시크릿을 반환한다. 제공된 키는 지정한 `input_encoding`를 사용해서 해석된다. 시크릿은
지정한 `output_encoding`을 사용하서 인코딩된다. 인코딩은 `'binary'`, `'hex'`,
`'base64'`가 될 수 있고 입력 인코딩을 지정하지 않으면 버퍼로 간주한다.

출력인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.getPrime([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 소수를
반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.getGenerator([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 제너레이터를
반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.getPublicKey([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 공개키를
반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.getPrivateKey([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 개인키를
반환한다. 인코딩을 지정하지 않으면 버퍼를 반환한다.

### diffieHellman.setPublicKey(public_key, [encoding])

Diffie-Hellman 공개키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가
될 수 있다. 인코딩을 지정하지 않으면 버퍼로 간주한다.

### diffieHellman.setPrivateKey(private_key, [encoding])

Diffie-Hellman 개인키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가
될 수 있다. 인코딩을 지정하지 않으면 버퍼로 간주한다.

## crypto.getDiffieHellman(group_name)

미리 정의된 Diffie-Hellman 키 교환 객체를 생성한다.
지원하는 그룹은 `'modp1'`, `'modp2'`, `'modp5'`
([RFC 2412][]에 정의되어 있다.)
와 `'modp14'`, `'modp15'`, `'modp16'`, `'modp17'`, `'modp18'`
([RFC 3526][]에 정의되어 있다.)이다.
반환된 객체는 위의 [crypto.createDiffieHellman()][]가 생성한 객체의
인터페이스와 비슷하지만 키를 변경할 수 없다.(예를 들면
[diffieHellman.setPublicKey()][]를 사용해서)
이 루틴을 사용했을 때의 이점은 관련자들이 미리 그룹 규칙을 생성하지 않고
교환하지도 않아서 프로세서와 통신 시간을 모두 아낄 수 있다.

공유된 비밀키를 얻는 예제:

    var crypto = require('crypto');
    var alice = crypto.getDiffieHellman('modp5');
    var bob = crypto.getDiffieHellman('modp5');

    alice.generateKeys();
    bob.generateKeys();

    var alice_secret = alice.computeSecret(bob.getPublicKey(), null, 'hex');
    var bob_secret = bob.computeSecret(alice.getPublicKey(), null, 'hex');

    /* alice_secret와 bob_secret는 같아야 한다. */
    console.log(alice_secret == bob_secret);

## crypto.pbkdf2(password, salt, iterations, keylen, callback)

비동기적인 PBKDF2가 전달한 password, salt, iterations에서 전달한 길이의 키를 얻기 위해
의사난수의(pseudorandom) 함수 HMAC-SHA1를 적용한다.
callback은 2개의 아규먼트 `(err, derivedKey)`를 받는다.

## crypto.pbkdf2Sync(password, salt, iterations, keylen)

동기 PBKDF2 함수. derivedKey를 반환하거나 오류를 던진다.

## crypto.randomBytes(size, [callback])

강력한 암호의 의사난수(pseudo-random) 데이터를 생성한다. 사용법:

    // async
    crypto.randomBytes(256, function(ex, buf) {
      if (ex) throw ex;
      console.log('Have %d bytes of random data: %s', buf.length, buf);
    });

    // sync
    try {
      var buf = crypto.randomBytes(256);
      console.log('Have %d bytes of random data: %s', buf.length, buf);
    } catch (ex) {
      // handle error
    }

## crypto.pseudoRandomBytes(size, [callback])

암호가 *아닌* 강력한 임의의 의사 데이터(pseudo-random data)를 생성한다.
데이터가 충분히 길다면 반환된 데이터는 유일한 값이 될 것이지만 반드시 예측할 수 없는 것은
아니다. 이 때문에 이 함수의 출력은 암호화된 키의 생성처럼 예측불가능성이 중요한 곳에서는
사용하지 말아야 한다.

반면 사용방법은 `crypto.randomBytes`의 반대이다.

## crypto.DEFAULT_ENCODING

문자열이나 버퍼를 받을 수 있는 함수에 사용하는 기본 인코딩. 기본값은 Buffer 객체를 사용하는
`'buffer'`이다. crypto 모듈을 기본 인코딩으로 `'binary'`를 사용하는 레거시 프로그램과
쉽게 호환성을 맞출 수 있다.

새로운 프로그램은 버퍼를 기대할 것이므로 임시적으로만 이 값을 사용해라.

## Recent API Changes

Crypto 모듈은 통일된 Stream API의 개념과 바이너리 데이터를 다루는 Buffer 객체가
존재하기 전에 Node에 추가되었다.

그래서 다른 Node 클래스들에는 있는 일반적인 메서드가 스트리밍 클래스에는 없고 많은 메서드들이
Buffers 대신 기본적으로 바이너리로 인코딩된 문자열을 받아들이고 반환한다.
이는 기본적으로 Buffer를 대신 사용하도록 바뀌었다.

이 변경사항은 전부는 아니지만 일부에서는 호환성을 깨뜨릴 것이다.

예를 들어 현재 Sign 클래스에 기본 인자를 사용하고 그 결과를 Verify 클래스에 전달하고
데이터를 검사하지 않는다면 이전과 마차가지로 계속 동작할 것이다.
바이너리 문자열을 얻고 Verify 객체에 바이너리 문자열을 제공하는 곳에서
Buffer를 얻을 것이고 Verify 객체에 Buffer를 제공할 것이다.

하지만 Buffers에서 제대로 동작하지 않을 문자열 데이터로 어떤 작업을 한거나(문자열을
이어붙힌다거나 데이터베이스에 저장하는 등) 인코딩 인자없이 암호화함수에 바이너리 문자열을
전달한다면 사용하고자 하는 인코딩을 지정하는 인코딩 인자를 제공해야 할 것이다.
기본적으로 바이너리 문자열을 사용하는 이전 방식으로 변경하려면
`crypto.DEFAULT_ENCODING` 필드를 'binary'로 설정해라.
새로운 프로그램은 버퍼를 기대할 것이므로 이는 임시적으로만 사용해라.


[createCipher()]: #crypto_crypto_createcipher_algorithm_password
[createCipheriv()]: #crypto_crypto_createcipheriv_algorithm_key_iv
[crypto.createDiffieHellman()]: #crypto_crypto_creatediffiehellman_prime_encoding
[diffieHellman.setPublicKey()]: #crypto_diffiehellman_setpublickey_public_key_encoding
[RFC 2412]: http://www.rfc-editor.org/rfc/rfc2412.txt
[RFC 3526]: http://www.rfc-editor.org/rfc/rfc3526.txt
