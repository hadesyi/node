# Zlib

<!--english start-->

    Stability: 3 - Stable

You can access this module with:

    var zlib = require('zlib');

This provides bindings to Gzip/Gunzip, Deflate/Inflate, and
DeflateRaw/InflateRaw classes.  Each class takes the same options, and
is a readable/writable Stream.

<!--english end-->

    안정성: 3 - Stable

이 모듈은 다음과 같은 방법으로 접근한다:

    var zlib = require('zlib');

이 모듈은 Gzip/Uunzip, Deflate/Inflate, DeflateRaw/InflateRaw 클래스에 대한 바인딩이다. 각 클래스는 읽기/쓰기 가능한 스트림이고 옵션은 모두 동일하다.

## Examples

<!--english start-->

Compressing or decompressing a file can be done by piping an
fs.ReadStream into a zlib stream, then into an fs.WriteStream.

    var gzip = zlib.createGzip();
    var fs = require('fs');
    var inp = fs.createReadStream('input.txt');
    var out = fs.createWriteStream('input.txt.gz');

    inp.pipe(gzip).pipe(out);

Compressing or decompressing data in one step can be done by using
the convenience methods.

    var input = '.................................';
    zlib.deflate(input, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString('base64'));
      }
    });

    var buffer = new Buffer('eJzT0yMAAGTvBe8=', 'base64');
    zlib.unzip(buffer, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString());
      }
    });

To use this module in an HTTP client or server, use the
[accept-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3)
on requests, and the
[content-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11)
header on responses.

**Note: these examples are drastically simplified to show
the basic concept.**  Zlib encoding can be expensive, and the results
ought to be cached.  See [Memory Usage Tuning](#memory_Usage_Tuning)
below for more information on the speed/memory/compression
tradeoffs involved in zlib usage.

    // client request example
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    var request = http.get({ host: 'izs.me',
                             path: '/',
                             port: 80,
                             headers: { 'accept-encoding': 'gzip,deflate' } });
    request.on('response', function(response) {
      var output = fs.createWriteStream('izs.me_index.html');

      switch (response.headers['content-encoding']) {
        // or, just use zlib.createUnzip() to handle both cases
        case 'gzip':
          response.pipe(zlib.createGunzip()).pipe(output);
          break;
        case 'deflate':
          response.pipe(zlib.createInflate()).pipe(output);
          break;
        default:
          response.pipe(output);
          break;
      }
    });

    // server example
    // Running a gzip operation on every request is quite expensive.
    // It would be much more efficient to cache the compressed buffer.
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    http.createServer(function(request, response) {
      var raw = fs.createReadStream('index.html');
      var acceptEncoding = request.headers['accept-encoding'];
      if (!acceptEncoding) {
        acceptEncoding = '';
      }

      // Note: this is not a conformant accept-encoding parser.
      // See http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3
      if (acceptEncoding.match(/\bdeflate\b/)) {
        response.writeHead(200, { 'content-encoding': 'deflate' });
        raw.pipe(zlib.createDeflate()).pipe(response);
      } else if (acceptEncoding.match(/\bgzip\b/)) {
        response.writeHead(200, { 'content-encoding': 'gzip' });
        raw.pipe(zlib.createGzip()).pipe(response);
      } else {
        response.writeHead(200, {});
        raw.pipe(response);
      }
    }).listen(1337);

<!--english end-->

파일을 압축하거나 압축을 푸는 일은 fs.ReadStream()으로 파일을 읽어서 zlib 스트림으로 보내고 나서(pipe) 다시 fs.WriteStream에 보내는(pipe) 것으로 이루어진다.

    var gzip = zlib.createGzip();
    var fs = require('fs');
    var inp = fs.createReadStream('input.txt');
    var out = fs.createWriteStream('input.txt.gz');

    inp.pipe(gzip).pipe(out);

데이터를 압축하고 압축을 푸는 일은 간단하게 단축 메소드로(convenience method) 한방에 할 수 있다:

    var input = '.................................';
    zlib.deflate(input, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString('base64'));
      }
    });

    var buffer = new Buffer('eJzT0yMAAGTvBe8=', 'base64');
    zlib.unzip(buffer, function(err, buffer) {
      if (!err) {
        console.log(buffer.toString());
      }
    });

HTTP client나 server에서 이 모듈를 사용하려면 request에서 사용할 수 있는 [accept-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3)을 보고 response에서는 [content-encoding](http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.11)를 보면 된다.

**Note: 이 예제는 기본 개념을 보여주기 위해 극도로 단순화한 것임.** Zlib 인코딩은 비싸서 결과물을 캐시하는게 좋다. 
[Memory Usage Tuning](#memory_Usage_Tuning)을 보면 zlib 튜닝시 speed/memory/compression에 대해 고려해야 하는 점이 나와 있다.

    // 클라이언트 요청 예제
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    var request = http.get({ host: 'izs.me',
                             path: '/',
                             port: 80,
                             headers: { 'accept-encoding': 'gzip,deflate' } });
    request.on('response', function(response) {
      var output = fs.createWriteStream('izs.me_index.html');

      switch (response.headers['content-encoding']) {
        // 또는 두 경우를 모두 다루기 위해서 그냥 zlib.createUnzip()를 사용해라.
        case 'gzip':
          response.pipe(zlib.createGunzip()).pipe(output);
          break;
        case 'deflate':
          response.pipe(zlib.createInflate()).pipe(output);
          break;
        default:
          response.pipe(output);
          break;
      }
    });

    // 서버 예제
    // 요청마다 gzip을 수행하는 것은 비용이 큰 작업이다.
    // 압축된 버퍼를 캐시해서 훨씬 효율적이 될 수 있다.
    var zlib = require('zlib');
    var http = require('http');
    var fs = require('fs');
    http.createServer(function(request, response) {
      var raw = fs.createReadStream('index.html');
      var acceptEncoding = request.headers['accept-encoding'];
      if (!acceptEncoding) {
        acceptEncoding = '';
      }

      // Note: 이는 적합한 accept-encoding 파서는 아니다.
      // http://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.3 를 봐라.
      if (acceptEncoding.match(/\bdeflate\b/)) {
        response.writeHead(200, { 'content-encoding': 'deflate' });
        raw.pipe(zlib.createDeflate()).pipe(response);
      } else if (acceptEncoding.match(/\bgzip\b/)) {
        response.writeHead(200, { 'content-encoding': 'gzip' });
        raw.pipe(zlib.createGzip()).pipe(response);
      } else {
        response.writeHead(200, {});
        raw.pipe(response);
      }
    }).listen(1337);

## Constants

<!--english start-->

<!--type=misc-->

All of the constants defined in zlib.h are also defined on
`require('zlib')`.  They are described in more detail in the zlib
documentation.  See <http://zlib.net/manual.html#Constants>
for more details.

<!--english end-->

<!--type=misc-->

zlib.h에 정의된 컨스턴트는 모두 `require('zlib')에도 정의돼 있다. zlib 문서에 자세히 설명돼 있으니 <http://zlib.net/manual.html#Constants>를 보라.

## zlib.createGzip([options])

<!--english start-->

Returns a new [Gzip](#zlib.Gzip) object with an [options](#options).

<!--english end-->

[options](#options)으로 [Gzip](#zlib.Gzip) 객채를 새로 만들어 리턴한다.

## zlib.createGunzip([options])

<!--english start-->

Returns a new [Gunzip](#zlib.Gunzip) object with an [options](#options).

<!--english end-->

[options](#options)으로 [Gunzip](#zlib.Gunzip) 객채를 새로 만들어 리턴한다.

## zlib.createDeflate([options])

<!--english start-->

Returns a new [Deflate](#zlib.Deflate) object with an [options](#options).

<!--english end-->

[options](#options)으로 [Deflate](#zlib.Deflate) 객채를 새로 만들어 리턴한다.

## zlib.createInflate([options])

<!--english start-->

Returns a new [Inflate](#zlib.Inflate) object with an [options](#options).

<!--english end-->

[options](#options)으로 [Inflate](#zlib.Inflate) 객채를 새로 만들어 리턴한다.

## zlib.createDeflateRaw([options])

<!--english start-->

Returns a new [DeflateRaw](#zlib.DeflateRaw) object with an [options](#options).

<!--english end-->

[options](#options)으로 [DeflateRaw](#zlib.DeflateRaw) 객채를 새로 만들어 리턴한다.

## zlib.createInflateRaw([options])

<!--english start-->

Returns a new [InflateRaw](#zlib.InflateRaw) object with an [options](#options).

<!--english end-->

[options](#options)으로 [InflateRaw](#zlib.InflateRaw) 객채를 새로 만들어 리턴한다.

## zlib.createUnzip([options])

<!--english start-->

Returns a new [Unzip](#zlib.Unzip) object with an [options](#options).

<!--english end-->

[options](#options)으로 [Unzip](#zlib.Unzip) 객채를 새로 만들어 리턴한다.

## Class: zlib.Gzip

<!--english start-->

Compress data using gzip.

<!--english end-->

gzip으로 데이터를 압축한다.

## Class: zlib.Gunzip

<!--english start-->

Decompress a gzip stream.

<!--english end-->

gzip 스트림의 압축을 푼다.

## Class: zlib.Deflate

<!--english start-->

Compress data using deflate.

<!--english end-->

deflate로 데이터를 압축한다.

## Class: zlib.Inflate

<!--english start-->

Decompress a deflate stream.

<!--english end-->

deflate 스트림의 압축을 푼다.

## Class: zlib.DeflateRaw

<!--english start-->

Compress data using deflate, and do not append a zlib header.

<!--english end-->

deflate로 데이터를 압축하지만 zlib 헤더는 넣지 않는다.

## Class: zlib.InflateRaw

<!--english start-->

Decompress a raw deflate stream.

<!--english end-->

raw deflate 스트림의 압축을 푼다.

## Class: zlib.Unzip

<!--english start-->

Decompress either a Gzip- or Deflate-compressed stream by auto-detecting
the header.

<!--english end-->

Gzip-이나 Deflate-로 압축한 스트림의 헤더를 자동으로 찾아서 압축을 푼다.

## Convenience Methods

<!--english start-->

<!--type=misc-->

All of these take a string or buffer as the first argument, and call the
supplied callback with `callback(error, result)`.  The
compression/decompression engine is created using the default settings
in all convenience methods.  To supply different options, use the
zlib classes directly.

<!--english end-->

<!--type=misc-->

여기에 있는 모든 메소드는 첫번째 인자로 버퍼나 스트링을 받는다. 그리고 콜백도 `callback(error, result)` 형식으로 호출한다. 압축/압축해제 엔진은 기본 설정으로 생성하고 다른 옵션으로 생성하고 싶으면 zlib 클래스를 직접사용해야 한다.

## zlib.deflate(buf, callback)

<!--english start-->

Compress a string with Deflate.

<!--english end-->

Deflate로 스트링을 압축한다.

## zlib.deflateRaw(buf, callback)

<!--english start-->

Compress a string with DeflateRaw.

<!--english end-->

DeflateRaw로 스트링을 압축한다.

## zlib.gzip(buf, callback)

<!--english start-->

Compress a string with Gzip.

<!--english end-->

Gzip으로 스트링을 압축한다.

## zlib.gunzip(buf, callback)

<!--english start-->

Decompress a raw Buffer with Gunzip.

<!--english end-->

Gunzip으로 Buffer의 압축을 푼다.

## zlib.inflate(buf, callback)

<!--english start-->

Decompress a raw Buffer with Inflate.

<!--english end-->

Inflate로 Buffer의 압축을 푼다.

## zlib.inflateRaw(buf, callback)

<!--english start-->

Decompress a raw Buffer with InflateRaw.

<!--english end-->

InflateRaw로 Buffer의 압축을 푼다.

## zlib.unzip(buf, callback)

<!--english start-->

Decompress a raw Buffer with Unzip.

<!--english end-->

Unzip으로 Buffer의 압축을 푼다.

## Options

<!--english start-->

<!--type=misc-->

Each class takes an options object.  All options are optional.  (The
convenience methods use the default settings for all options.)

Note that some options are only
relevant when compressing, and are ignored by the decompression classes.

* chunkSize (default: 16*1024)
* windowBits
* level (compression only)
* memLevel (compression only)
* strategy (compression only)

See the description of `deflateInit2` and `inflateInit2` at
<http://zlib.net/manual.html#Advanced> for more information on these.

<!--english end-->

<!--type=misc-->

모든 클래스는 옵션 객체를 인자로 받고 생략 가능하다(단축 메소드는 기본값을 사용한다).

어떤 옵션은 압축 클래스에만 필요하고 압축을 푸는 클래스에서는 무시한다.

* chunkSize (default: 16*1024)
* windowBits
* level (compression only)
* memLevel (compression only)
* strategy (compression only)

`deflateInit2`와 `inflateInit2`의 설명은 <http://zlib.net/manual.html#Advanced> 페이지에서 보라.

## Memory Usage Tuning

<!--english start-->

<!--type=misc-->

From `zlib/zconf.h`, modified to node's usage:

The memory requirements for deflate are (in bytes):

    (1 << (windowBits+2)) +  (1 << (memLevel+9))

that is: 128K for windowBits=15  +  128K for memLevel = 8
(default values) plus a few kilobytes for small objects.

For example, if you want to reduce
the default memory requirements from 256K to 128K, set the options to:

    { windowBits: 14, memLevel: 7 }

Of course this will generally degrade compression (there's no free lunch).

The memory requirements for inflate are (in bytes)

    1 << windowBits

that is, 32K for windowBits=15 (default value) plus a few kilobytes
for small objects.

This is in addition to a single internal output slab buffer of size
`chunkSize`, which defaults to 16K.

The speed of zlib compression is affected most dramatically by the
`level` setting.  A higher level will result in better compression, but
will take longer to complete.  A lower level will result in less
compression, but will be much faster.

In general, greater memory usage options will mean that node has to make
fewer calls to zlib, since it'll be able to process more data in a
single `write` operation.  So, this is another factor that affects the
speed, at the cost of memory usage.

<!--english end-->

<!--type=misc-->

`zlib/zconf.h`에 있는 설정을 node에 맞게 수정하는 법:

deflate할 때 필요한 메모리(바이트 단위):

    (1 << (windowBits+2)) +  (1 << (memLevel+9))

이 표현은 windowBits=15일 때 128K가 필요하고 memLevel=8일 때 128k가 더 필요하다는 뜻이다(기본값임). 그리고 객체에 필요한 몇 킬로 바이트가 더 든다.

만약 필요한 메모리를 256K에서 128K로 줄이고 싶으면 옵션을 다음과 같이 주면된다:

    { windowBits: 14, memLevel: 7 }

물론 이 설정은 압축 성능을 떨어뜨린다(공짜 점심은 없다).

inflate에 필요한 메모리(바이트단위):

    1 << windowBits

이 표현은 windowBits=15일 때 32K가 필요하다는 말이다(기본값임). 그리고 객체에 필요한 몇 킬로바이트가 더 든다.

그리고 내부에 결과물을 위한 버퍼가 하나 있다. `chunkSize`의 값이 버퍼의 크기인데 기본 값은 16K이다.

zlib 압축의 속도는 `level` 설정이 가장 큰 영향을 끼친다. 레벨을 높이면 압축률은 높아지지만 더 오래 걸린다. 레벨을 낮추면 압축률은 낮아지지만 더 빨라진다.

보통 메모리를 크게 잡으면 `write` 오퍼레이션을 한번 할 때 데이터를 더 많이 처리하기 때문에 Node가 zlib을 더 적게 호출한다.  그래서 이 요소도 속도와 메모리 사용 효율에 영향을 준다.
