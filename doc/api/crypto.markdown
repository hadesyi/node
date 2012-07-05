# Crypto

<!--english start-->

    Stability: 3 - Stable

Use `require('crypto')` to access this module.

The crypto module requires OpenSSL to be available on the underlying platform.
It offers a way of encapsulating secure credentials to be used as part
of a secure HTTPS net or http connection.

It also offers a set of wrappers for OpenSSL's hash, hmac, cipher, decipher, sign and verify methods.

<!--english end-->

    안정성: 3 - Stable

이 모듈에 접근하려면 `require('crypto')`를 사용해라.

crypto 모듈은 의존 플랫폼에서 이용할 수 있는 OpenSSL을 필요로 한다.
안정한 HTTPS net이나 http 연결에서 사용되는 안정한 인증서를 캡슐화하는 
방법을 제공한다.

OpenSSL의 hash, hmac, cipher, decipher, sign, verify 메서드의 랩퍼(wrapper)도
제공한다.

## crypto.createCredentials(details)

<!--english start-->

Creates a credentials object, with the optional details being a dictionary with keys:

* `pfx` : A string or buffer holding the PFX or PKCS12 encoded private key, certificate and CA certificates
* `key` : a string holding the PEM encoded private key
* `cert` : a string holding the PEM encoded certificate
* `passphrase` : A string of passphrase for the private key or pfx
* `ca` : either a string or list of strings of PEM encoded CA certificates to trust.
* `ciphers`: a string describing the ciphers to use or exclude. Consult
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT> for details
  on the format.

If no 'ca' details are given, then node.js will use the default publicly trusted list of CAs as given in
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>.

<!--english end-->

인증서 객체를 생성한다. 선택사항인 details는 키를 가진 딕셔너리가 된다.

* `pfx` : PFX나 PKCS12로 암호화된 개인키, 인증서, CA 증명서를 담고 있는 문자열이나 버퍼
* `key` : PEM으로 암호화된 개인키를 담고 있는 문자열
* `cert` : PEM으로 암호화된 증명서를 담고 있는 문자열
* `passphrase` : 개인키나 pfx에 대한 암호문(passphrase) 문자열
* `ca` : PEM으로 암호화된 믿을 수 있는 CA 증명서의 문자열이나 리스트 
* `ciphers`: 사용하거나 제외할 암호(cipher)를 설명하는 문자열. 자세한 형식은 
  <http://www.openssl.org/docs/apps/ciphers.html#CIPHER_LIST_FORMAT>를 참조해라.

'ca'에 대한 세부내용을 전달하지 않는다면 node.js는 공개적으로 신뢰할 수 있는 CAㅇ의 리스트를 
<http://mxr.mozilla.org/mozilla/source/security/nss/lib/ckfw/builtins/certdata.txt>에서 받아서 기본으로 사용할 것이다.


## crypto.createHash(algorithm)

<!--english start-->

Creates and returns a hash object, a cryptographic hash with the given algorithm
which can be used to generate hash digests.

`algorithm` is dependent on the available algorithms supported by the version
of OpenSSL on the platform. Examples are `'sha1'`, `'md5'`, `'sha256'`, `'sha512'`, etc.
On recent releases, `openssl list-message-digest-algorithms` will display the available digest algorithms.

Example: this program that takes the sha1 sum of a file

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

<!--english end-->

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

<!--english start-->

The class for creating hash digests of data.

Returned by `crypto.createHash`.

<!--english end-->

데이터의 해시 다이제스트를 생성하는 클래스다.

`crypto.createHash`가 반환하는 클래스다.

### hash.update(data, [input_encoding])

<!--english start-->

Updates the hash content with the given `data`, the encoding of which is given
in `input_encoding` and can be `'utf8'`, `'ascii'` or `'binary'`.
Defaults to `'binary'`.
This can be called many times with new data as it is streamed.

<!--english end-->

전달한 `input_encoding`의 `data`로 해시 내용을 갱신한다.
전달한 `input_encoding` 인코딩은 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
기본값은 `'binary'`이다. 
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출될 수 있다.

### hash.digest([encoding])

<!--english start-->

Calculates the digest of all of the passed data to be hashed.
The `encoding` can be `'hex'`, `'binary'` or `'base64'`.
Defaults to `'binary'`.

Note: `hash` object can not be used after `digest()` method been called.

<!--english end-->

해시되어야 하는 전달한 데이터의 모든 다이제스트를 계산한다. 
`encoding`은 `'hex'`, `'binary'`, `'base64'`가 될 수 있다.
기본값은 `'binary'`이다.

Note: `hash` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createHmac(algorithm, key)

<!--english start-->

Creates and returns a hmac object, a cryptographic hmac with the given algorithm and key.

`algorithm` is dependent on the available algorithms supported by OpenSSL - see createHash above.
`key` is the hmac key to be used.

<!--english end-->

hmac 객체를 생성하고 반환한다. 전달한 algorithm과 key의 암호화 해시이다. 

`algorithm`는 OpenSSL에서 사용할 수 있는 알고리즘에 의존한다. -- 위의 createHash를 봐라.
`key`는 사용할 hmac 키이다.

## Class: Hmac

<!--english start-->

Class for creating cryptographic hmac content.

Returned by `crypto.createHmac`.

<!--english end-->

암호화 hmac 내용을 생성하는 클래스다.

`crypto.createHmac`가 리턴하는 클래스다.

### hmac.update(data)

<!--english start-->

Update the hmac content with the given `data`.
This can be called many times with new data as it is streamed.

<!--english end-->

전달한 `data`로 hmac 내용을 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러번 호출될 수 있다.

### hmac.digest([encoding])

<!--english start-->

Calculates the digest of all of the passed data to the hmac.
The `encoding` can be `'hex'`, `'binary'` or `'base64'`.
Defaults to `'binary'`.

Note: `hmac` object can not be used after `digest()` method been called.

<!--english end-->

hmac이 되어야 하는 전달한 데이터의 모든 다이제스트를 계산한다. 
`encoding`은 `'hex'`, `'binary'`, `'base64'`가 될 수 있다.
기본값은 `'binary'`이다.

Note: `hmac` 객체는 `digest()` 메서드가 호출한 후에는 사용할 수 없다.


## crypto.createCipher(algorithm, password)

<!--english start-->

Creates and returns a cipher object, with the given algorithm and password.

`algorithm` is dependent on OpenSSL, examples are `'aes192'`, etc.
On recent releases, `openssl list-cipher-algorithms` will display the
available cipher algorithms.
`password` is used to derive key and IV, which must be `'binary'` encoded
string (See the [Buffer section](buffer.html) for more information).

<!--english end-->

전달한 algorithm과 password로 암호화한 암호화 객체를 생성하고 반환한다.

`algorithm`는 OpenSSL에 의존적이다. `'aes192'` 등이 있다.
OpenSSL 최근 릴리즈에서는 `openssl list-cipher-algorithms`로 사용할 수 있는
암호화 알고리즘을 볼 수 있다.
`password`는 key와 IV를 얻는데 사용하고 반드시 `'binary'`로 인코딩된 문자열이어야
한다.(더 자세한 내용은 [Buffer section](buffer.html)를 봐라.)

## crypto.createCipheriv(algorithm, key, iv)

<!--english start-->

Creates and returns a cipher object, with the given algorithm, key and iv.

`algorithm` is the same as the `createCipher()`. `key` is a raw key used in
algorithm. `iv` is an Initialization vector. `key` and `iv` must be `'binary'`
encoded string (See the [Buffer section](buffer.html) for more information).

<!--english end-->

전달한 algorithm, key, iv로 암호화된 암호화 객체를 생성하고 반환한다.

`algorithm`은 createCipher()`와 같다. `key`는 algorithm에서 사용하는 로우 키(raw key)
이다. `iv`는 초기화 벡터(Initialization vector)이고 `iv`는 반드시 `'binary'`로 인코딩된
문자열이어야 한다. (더 자세한 내용은 [Buffer section](buffer.html)를 봐라.)

## Class: Cipher

<!--english start-->

Class for encrypting data.

Returned by `crypto.createCipher` and `crypto.createCipheriv`.

<!--english end-->

데이터를 암호화하는 클래스이다.

`crypto.createCipher`와 `crypto.createCipheriv`가 반환하는 객체다.

### cipher.update(data, [input_encoding], [output_encoding])

<!--english start-->

Updates the cipher with `data`, the encoding of which is given in
`input_encoding` and can be `'utf8'`, `'ascii'` or `'binary'`.
Defaults to `'binary'`.

The `output_encoding` specifies the output format of the enciphered data,
and can be `'binary'`, `'base64'` or `'hex'`. Defaults to `'binary'`.

Returns the enciphered contents, and can be called many times with new data as it is streamed.

<!--english end-->

전달한 `input_encoding`의 인코딩의 `data`로 cipher를 갱신한다.
`input_encoding`는 `'utf8'`, `'ascii'`, `'binary'`가 될 수 있다.
기본값은 `'binary'`이다.

`output_encoding`는 암호화된 데이터의 출력형식을 지정하고 `'binary'`, `'base64'`, 
`'hex'`가 될 수 있다. 기본값은 `'binary'`이다.

암호화된 내용을 반환하고 스트림처럼 새로운 데이터로 여러번 호출할 수 있다.

### cipher.final([output_encoding])

<!--english start-->

Returns any remaining enciphered contents, with `output_encoding` being one of:
`'binary'`, `'base64'` or `'hex'`. Defaults to `'binary'`.

Note: `cipher` object can not be used after `final()` method been called.

<!--english end-->

`'binary'`, `'base64'`, `'hex'`중의 하나인 `output_encoding`로 남아있는 모든 
암호화된 내용을 반환한다. 기본값은 `'binary'`이다.

Note: `cipher` 객체는 `final()` 메서드를 호출한 후에는 사용할 수 없다.


## crypto.createDecipher(algorithm, password)

<!--english start-->

Creates and returns a decipher object, with the given algorithm and key.
This is the mirror of the [createCipher()](#crypto.createCipher) above.

<!--english end-->

전달한 algorithm와 key로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipher()](#crypto.createCipher)의 반영이다.

## crypto.createDecipheriv(algorithm, key, iv)

<!--english start-->

Creates and returns a decipher object, with the given algorithm, key and iv.
This is the mirror of the [createCipheriv()](#crypto.createCipheriv) above.

<!--english end-->

전달한 algorithm, key, iv로 decipher 객체를 생성하고 반환한다.
이 함수는 위의 [createCipheriv()](#crypto.createCipheriv)의 반영이다.

## Class: Decipher

<!--english start-->

Class for decrypting data.

Returned by `crypto.createDecipher` and `crypto.createDecipheriv`.

<!--english end-->

데이터를 복호화하는 클래스다.

`crypto.createDecipher`와 `crypto.createDecipheriv`가 반환하는 클래스다.

### decipher.update(data, [input_encoding], [output_encoding])

<!--english start-->

Updates the decipher with `data`, which is encoded in `'binary'`, `'base64'`
or `'hex'`. Defaults to `'binary'`.

The `output_decoding` specifies in what format to return the deciphered
plaintext: `'binary'`, `'ascii'` or `'utf8'`. Defaults to `'binary'`.

<!--english end-->

'binary'`, `'base64'`, `'hex'`로 인코딩된 `data`로 decipher를 갱신한다.
기본값은 `'binary'`이다.

`output_decoding`는 반환할 복호화된 평문의 형식을 지정한다. `'binary'`, `'ascii'`,
`'utf8'`가 될 수 있고 기본값은 `'binary'`이다.

### decipher.final([output_encoding])

<!--english start-->

Returns any remaining plaintext which is deciphered,
with `output_encoding` being one of: `'binary'`, `'ascii'` or `'utf8'`.
Defaults to `'binary'`.

Note: `decipher` object can not be used after `final()` method been called.

<!--english end-->

`'binary'`, `'ascii'`, `'utf8'`중에 하나가 될 수 있는 `output_encoding`로 남아있는 
모든 복호화된 평문을 반환한다. 기본값은 `'binary'`이다.

Note: `decipher` 객체는 `final()` 메서드가 호출된 후에는 사용할 수 없다.


## crypto.createSign(algorithm)

<!--english start-->

Creates and returns a signing object, with the given algorithm.
On recent OpenSSL releases, `openssl list-public-key-algorithms` will display
the available signing algorithms. Examples are `'RSA-SHA256'`.

<!--english end-->

전달한 algorithm으로 서명된 객체를 생성하고 반환한다.
최근 OpenSSL 릴리즈에서 `openssl list-public-key-algorithms`로 사용할 수 있는
서명된 알고리즘을 볼 수 있다. 예를 들면 `'RSA-SHA256'`가 있다.

## Class: Signer

<!--english start-->

Class for generating signatures.

Returned by `crypto.createSign`.

<!--english end-->

서명을 생성하는 클래스다.

`crypto.createSign`가 반환하는 클래스이다.

### signer.update(data)

<!--english start-->

Updates the signer object with data.
This can be called many times with new data as it is streamed.

<!--english end-->

data로 signer 객체를 갱신한다. 
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### signer.sign(private_key, [output_format])

<!--english start-->

Calculates the signature on all the updated data passed through the signer.
`private_key` is a string containing the PEM encoded private key for signing.

Returns the signature in `output_format` which can be `'binary'`, `'hex'` or
`'base64'`. Defaults to `'binary'`.

Note: `signer` object can not be used after `sign()` method been called.

<!--english end-->

전달한 갱신 데이터 모두를 signer를 통해서 서명을 계산한다.
`private_key`는 서명에 사용할 PEM 인코딩된 개인키를 담고 있는 문자열이다.

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 `output_format`의 서명을 반환한다.
기본값은 `'binary'`이다.

Note: `signer` 객체는 `sign()` 메서드를 호출한 후에는 사용할 수 없다.

## crypto.createVerify(algorithm)

<!--english start-->

Creates and returns a verification object, with the given algorithm.
This is the mirror of the signing object above.

<!--english end-->

전달한 algorithm으로 검증 객체를 생성하고 반환한다.
이 함수는 위의 서명객체의 반영이다.

## Class: Verify

<!--english start-->

Class for verifying signatures.

Returned by `crypto.createVerify`.

<!--english end-->

서명을 검증하는 클래스다.

`crypto.createVerify`가 반환하는 클래스이다.

### verifier.update(data)

<!--english start-->

Updates the verifier object with data.
This can be called many times with new data as it is streamed.

<!--english end-->

data로 verifier 객체를 갱신한다.
이 함수는 스트림처럼 새로운 데이터가 올 때마다 여러 번 호출할 수 있다.

### verifier.verify(object, signature, [signature_format])

<!--english start-->

Verifies the signed data by using the `object` and `signature`. `object` is  a
string containing a PEM encoded object, which can be one of RSA public key,
DSA public key, or X.509 certificate. `signature` is the previously calculated
signature for the data, in the `signature_format` which can be `'binary'`,
`'hex'` or `'base64'`. Defaults to `'binary'`.

Returns true or false depending on the validity of the signature for the data and public key.

Note: `verifier` object can not be used after `verify()` method been called.

<!--english end-->

`object`와 `signature`를 사용해서 서명된 데이터를 검증한다. `object`는 RSA 공개키, 
DSA 공개키, X.509 인증서 중 하나가 될 수 있는 PEM으로 인코딩된 객체를 담고 있는 문자열이다.
`signature`는 `'binary'`, `'hex'`, `'base64'`가 될 수 있는 `signature_format`의 
데이터에 대해 이전에 계산한 서명이다. 기본값은 `'binary'`이다.

데이터와 공개키에 대한 서명의 유효성에 따라 true나 false를 반환한다.

Note: `verifier` 객체는 `verify()` 메서드를 호출한 뒤에는 사용할 수 없다.

## crypto.createDiffieHellman(prime_length)

<!--english start-->

Creates a Diffie-Hellman key exchange object and generates a prime of the
given bit length. The generator used is `2`.

<!--english end-->

Diffie-Hellman 키 교환 객체를 생성하고 전달한 비트 길이의 소수를 생성한다.
사용된 제너레이터는 `2`이다.

## crypto.createDiffieHellman(prime, [encoding])

<!--english start-->

Creates a Diffie-Hellman key exchange object using the supplied prime. The
generator used is `2`. Encoding can be `'binary'`, `'hex'`, or `'base64'`.
Defaults to `'binary'`.

<!--english end-->

제공된 소수를 사용해서 Diffie-Hellman 키 교환 객체를 생성한다. 사용된 제너레이터는 
`2`이다. 인코딩은 `'binary'`, `'hex'`, `'base64'`가 될 수 있다.
기본값은 `'binary'`이다.

## Class: DiffieHellman

<!--english start-->

The class for creating Diffie-Hellman key exchanges.

Returned by `crypto.createDiffieHellman`.

<!--english end-->

Diffie-Hellman 키 교환을 생성하는 클래스이다.

`crypto.createDiffieHellman`가 반환하는 클래스이다.

### diffieHellman.generateKeys([encoding])

<!--english start-->

Generates private and public Diffie-Hellman key values, and returns the
public key in the specified encoding. This key should be transferred to the
other party. Encoding can be `'binary'`, `'hex'`, or `'base64'`.
Defaults to `'binary'`.

<!--english end-->

개인 Diffie-Hellman 키값과 공개 Diffie-Hellman 키값을 생성하고 지정한 인코딩으로 
공개키를 반환한다. 이 키는 다른 관련자에게 이동할 수 있다. 인코딩은 `'binary'`, 
`'hex'`, `'base64'`가 될 수 있다. 기본값은 `'binary'`이다.

### diffieHellman.computeSecret(other_public_key, [input_encoding], [output_encoding])

<!--english start-->

Computes the shared secret using `other_public_key` as the other party's
public key and returns the computed shared secret. Supplied key is
interpreted using specified `input_encoding`, and secret is encoded using
specified `output_encoding`. Encodings can be `'binary'`, `'hex'`, or
`'base64'`. The input encoding defaults to `'binary'`.
If no output encoding is given, the input encoding is used as output encoding.

<!--english end-->

다른 관련자의 공개키로 `other_public_key`를 사용하는 공유 시크릿을 계산하고 계산된 
공유 시크릿을 반환한다. 제공된 키는 지정한 `input_encoding`를 사용해서 해석된다. 시크릿은 
지정한 `output_encoding`을 사용하서 인코딩된다. 인코딩은 `'binary'`, `'hex'`, 
`'base64'`가 될 수 있고 기본 입력인코딩은 `'binary'`이다.
출력인코딩을 지정하지 않으면 입력인코딩을 출력인코딩으로 사용한다.

### diffieHellman.getPrime([encoding])

<!--english start-->

Returns the Diffie-Hellman prime in the specified encoding, which can be
`'binary'`, `'hex'`, or `'base64'`. Defaults to `'binary'`.

<!--english end-->

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 소수를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getGenerator([encoding])

<!--english start-->

Returns the Diffie-Hellman prime in the specified encoding, which can be
`'binary'`, `'hex'`, or `'base64'`. Defaults to `'binary'`.

<!--english end-->

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 제너레이터를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getPublicKey([encoding])

<!--english start-->

Returns the Diffie-Hellman public key in the specified encoding, which can
be `'binary'`, `'hex'`, or `'base64'`. Defaults to `'binary'`.

<!--english end-->

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 공개키를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.getPrivateKey([encoding])

<!--english start-->

Returns the Diffie-Hellman private key in the specified encoding, which can
be `'binary'`, `'hex'`, or `'base64'`. Defaults to `'binary'`.

<!--english end-->

`'binary'`, `'hex'`, `'base64'`가 될 수 있는 지정한 인코딩의 Diffie-Hellman 개인키를 
반환한다. 기본값은 `'binary'`이다.

### diffieHellman.setPublicKey(public_key, [encoding])

<!--english start-->

Sets the Diffie-Hellman public key. Key encoding can be `'binary'`, `'hex'`,
or `'base64'`. Defaults to `'binary'`.

<!--english end-->

Diffie-Hellman 공개키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가 
될 수 있다. 기본값은 `'binary'`이다.

### diffieHellman.setPrivateKey(public_key, [encoding])

<!--english start-->

Sets the Diffie-Hellman private key. Key encoding can be `'binary'`, `'hex'`,
or `'base64'`. Defaults to `'binary'`.

<!--english end-->

Diffie-Hellman 개인키를 설정한다. 키 인코딩은 `'binary'`, `'hex'`, `'base64'`가 
될 수 있다. 기본값은 `'binary'`이다.

## crypto.pbkdf2(password, salt, iterations, keylen, callback)

<!--english start-->

Asynchronous PBKDF2 applies pseudorandom function HMAC-SHA1 to derive
a key of given length from the given password, salt and iterations.
The callback gets two arguments `(err, derivedKey)`.

<!--english end-->

비동기적인 PBKDF2가 전달한 password, salt, iterations에서 전달한 길이의 키를 얻기 위해 
의사난수의(pseudorandom) 함수 HMAC-SHA1를 적용한다. 
callback은 2개의 아규먼트 `(err, derivedKey)`를 받는다.


## crypto.randomBytes(size, [callback])

<!--english start-->

Generates cryptographically strong pseudo-random data. Usage:

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

<!--english end-->

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
