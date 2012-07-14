# Crypto

    Stability: 3 - Stable

이 모듈에 접근하려면 `require('crypto')`를 사용해라.

crypto 모듈은 의존 플랫폼에서 이용할 수 있는 OpenSSL을 필요로 한다.
안정한 HTTPS net이나 http 연결에서 사용되는 안정한 인증서를 캡슐화하는 
방법을 제공한다.

OpenSSL의 hash, hmac, cipher, decipher, sign, verify 메서드의 랩퍼(wrapper)도
제공한다.

## crypto.createCredentials(details)

인증서 객체를 생성한다. 선택사항인 details는 키를 가진 딕셔너리가 된다.

* `pfx` : PFX나 PKCS12로 암호화된 개인키, 인증서, CA 증명서를 담고 있는 문자열이나 버퍼
* `key` : PEM으로 암호화된 개인키를 담고 있는 문자열
* `cert` : PEM으로 암호화된 증명서를 담고 있는 문자열
* `passphrase` : 개인키나 pfx에 대한 암호문(passphrase) 문자열
* `ca` : PEM으로 암호화된 믿을 수 있는 CA 증명서의 문자열이나 리스트 
* `ciphers`: 사용하거나 제외할 암호(cipher)를 설명하는 문자열. 자세한 형식은 
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>를 참조해라.

'ca'에 대한 세부내용을 전달하지 않는다면 node.js는 공개적으로 신뢰할 수 있는 CA의 리스트를 기본으로
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>에서 받아서 사용할 것이다.


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

`crypto.createHash`가 반환하는 클래스다.

### hash.update(data, [input_encoding])

전달한 `input_encoding`의 `data`로 해시 내용을 갱신한다.
전달한 `input_encoding` 인코딩은 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
기본값은 `'binary'`이다. 
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출될 수 있다.

### hash.digest([encoding])

해시되어야 하는 전달한 데이터의 모든 다이제스트를 계산한다. 
`encoding`은 `'hex'`, `'binary'`, `'base64'`가 될 수 있다.
기본값은 `'binary'`이다.

Note: `hash` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createHmac(algorithm, key)

hmac 객체를 생성하고 반환한다. 전달한 algorithm과 key의 암호화 해시이다. 

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
기본값은 `'binary'`이다.

Note: `hmac` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createCipher(algorithm, password)

전달한 algorithm과 password로 암호화한 암호화 객체를 생성하고 반환한다.

`algorithm`는 OpenSSL에 의존적이다. `'aes192'` 등이 있다.
OpenSSL 최근 릴리즈에서는 `openssl list-cipher-algorithms`로 사용할 수 있는
암호화 알고리즘을 볼 수 있다.
`password`는 key와 IV를 얻는데 사용하고 반드시 `'binary'`로 인코딩된 문자열이어야
한다.(더 자세한 내용은 [Buffer section](buffer.html)를 봐라.)

## crypto.createCipheriv(algorithm, key, iv)

전달한 algorithm, key, iv로 암호화된 암호화 객체를 생성하고 반환한다.

`algorithm`은 createCipher()`와 같다. `key`는 algorithm에서 사용하는 로우 키(raw key)
이다. `iv`는 초기화 벡터(Initialization vector)이고 `iv`는 반드시 `'binary'`로 인코딩된
문자열이어야 한다. (더 자세한 내용은 [Buffer section](buffer.html)를 봐라.)

## Class: Cipher

데이터를 암호화하는 클래스이다.

`crypto.createCipher`와 `crypto.createCipheriv`가 반환하는 객체다.

### cipher.update(data, [input_encoding], [output_encoding])

전달한 `input_encoding`의 인코딩의 `data`로 cipher를 갱신한다.
`input_encoding`는 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
기본값은 `'binary'`이다.

`output_encoding`는 암호화된 데이터의 출력형식을 지정하고 `'binary'`, `'base64'`, 
`'hex'`가 될 수 있다. 기본값은 `'binary'`이다.

암호화된 내용을 반환하고 스트림처럼 새로운 데이터로 여러번 호출할 수 있다.

### cipher.final([output_encoding])

`'binary'`, `'base64'`, `'hex'`중의 하나인 `output_encoding`로 남아있는 모든 
암호화된 내용을 반환한다. 기본값은 `'binary'`이다.

Note: `cipher` 객체는 `final()` 메서드를 호출한 후에는 사용할 수 없다.


## crypto.createDecipher(algorithm, password)

전달한 algorithm와 key로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipher()](#crypto.createCipher)의 반영이다.

## crypto.createDecipheriv(algorithm, key, iv)

전달한 algorithm, key, iv로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipheriv()](#crypto.createCipheriv)의 반영이다.

## Class: Decipher

데이터를 복호화하는 클래스다.

`crypto.createDecipher`와 `crypto.createDecipheriv`가 반환하는 클래스다.

### decipher.update(data, [input_encoding], [output_encoding])

'binary'`, `'base64'`, `'hex'`로 인코딩된 `data`로 decipher를 갱신한다.
기본값은 `'binary'`이다.

`output_decoding`는 반환할 복호화된 평문의 형식을 지정한다. `'binary'`, `'ascii'`,
`'utf8'`가 될 수 있고 기본값은 `'binary'`이다.

### decipher.final([output_encoding])

`'binary'`, `'ascii'`, `'utf8'`중에 하나가 될 수 있는 `output_encoding`로 남아있는 
모든 복호화된 평문을 반환한다. 기본값은 `'binary'`이다.

Note: `decipher` 객체는 `final()` 메서드가 호출된 후에는 사용할 수 없다.


## crypto.createSign(algorithm)

전달한 algorithm으로 서명된 객체를 생성하고 반환한다.
최근 OpenSSL 릴리즈에서 `openssl list-public-key-algorithms`로 사용할 수 있는
서명된 알고리즘을 볼 수 있다. 예를 들면 `'RSA-SHA256'`가 있다.

## Class: Signer

서명을 생성하는 클래스다.

`crypto.createSign`가 반환하는 클래스이다.

### signer.update(data)

data로 signer 객체를 갱신한다. 
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### signer.sign(private_key, [output_format])

전달한 갱신 데이터 모두를 signer를 통해서 서명을 계산한다.
`private_key`는 서명에 사용할 PEM 인코딩된 개인키를 담고 있는 문자열이다.

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 `output_format`의 서명을 반환한다.
기본값은 `'binary'`이다.

Note: `signer` 객체는 `sign()` 메서드를 호출한 후에는 사용할 수 없다.

## crypto.createVerify(algorithm)

전달한 algorithm으로 검증 객체를 생성하고 반환한다.
이 함수는 위의 서명객체의 반영이다.

## Class: Verify

서명을 검증하는 클래스다.

`crypto.createVerify`가 반환하는 클래스이다.

### verifier.update(data)

data로 verifier 객체를 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### verifier.verify(object, signature, [signature_format])

`object`와 `signature`를 사용해서 서명된 데이터를 검증한다. `object`는 RSA 공개키, 
DSA 공개키, X.509 인증서 중 하나가 될 수 있는 PEM으로 인코딩된 객체를 담고 있는 문자열이다.
`signature`는 `'binary'`, `'hex'`, `'base64'`가 될 수 있는 `signature_format`의 
데이터에 대해 이전에 계산한 서명이다. 기본값은 `'binary'`이다.

데이터와 공개키에 대한 서명의 유효성에 따라 true나 false를 반환한다.

Note: `verifier` 객체는 `verify()` 메서드를 호출한 뒤에는 사용할 수 없다.

## crypto.createDiffieHellman(prime_length)

Diffie-Hellman 키 교환 객체를 생성하고 전달한 비트 길이의 소수를 생성한다.
사용된 제너레이터는 `2`이다.

## crypto.createDiffieHellman(prime, [encoding])

제공된 소수를 사용해서 Diffie-Hellman 키 교환 객체를 생성한다. 사용된 제너레이터는 
`2`이다. 인코딩은 `'binary'`, `'hex'`, `'base64'`가 될 수 있다.
기본값은 `'binary'`이다.

## Class: DiffieHellman

Diffie-Hellman 키 교환을 생성하는 클래스이다.

`crypto.createDiffieHellman`가 반환하는 클래스이다.

### diffieHellman.generateKeys([encoding])

개인 Diffie-Hellman 키값과 공개 Diffie-Hellman 키값을 생성하고 지정한 인코딩으로 
공개키를 반환한다. 이 키는 다른 관련자에게 이동할 수 있다. 인코딩은 `'binary'`, 
`'hex'`, `'base64'`가 될 수 있다. 기본값은 `'binary'`이다.

### diffieHellman.computeSecret(other_public_key, [input_encoding], [output_encoding])

다른 관련자의 공개키로 `other_public_key`를 사용하는 공유 시크릿을 계산하고 계산된 
공유 시크릿을 반환한다. 제공된 키는 지정한 `input_encoding`를 사용해서 해석된다. 시크릿은 
지정한 `output_encoding`을 사용하서 인코딩된다. 인코딩은 `'binary'`, `'hex'`, 
`'base64'`가 될 수 있고 기본 입력인코딩은 `'binary'`이다.
출력인코딩을 지정하지 않으면 입력인코딩을 출력인코딩으로 사용한다.

### diffieHellman.getPrime([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 소수를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getGenerator([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 제너레이터를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getPublicKey([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 공개키를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getPrivateKey([encoding])

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 개인키를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.setPublicKey(public_key, [encoding])

Diffie-Hellman 공개키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가 
될 수 있다. 기본값은 `'binary'`이다.

### diffieHellman.setPrivateKey(public_key, [encoding])

Diffie-Hellman 개인키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가 
될 수 있다. 기본값은 `'binary'`이다.

## crypto.pbkdf2(password, salt, iterations, keylen, callback)

비동기적인 PBKDF2가 전달한 password, salt, iterations에서 전달한 길이의 키를 얻기 위해 
의사난수의(pseudorandom) 함수 HMAC-SHA1를 적용한다. 
callback은 2개의 아규먼트 `(err, derivedKey)`를 받는다.


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
