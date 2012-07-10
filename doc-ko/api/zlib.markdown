# Zlib

    Stability: 3 - Stable

이 모듈은 다음과 같은 방법으로 접근한다:

    var zlib = require('zlib');

이 모듈은 Gzip/Uunzip, Deflate/Inflate, DeflateRaw/InflateRaw 클래스에 대한 바인딩이다. 각 클래스는 읽기/쓰기 가능한 스트림이고 옵션은 모두 동일하다.

## Examples

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

<!--type=misc-->

zlib.h에 정의된 컨스턴트는 모두 `require('zlib')에도 정의돼 있다. zlib 문서에 자세히 설명돼 있으니 <http://zlib.net/manual.html#Constants>를 보라.

## zlib.createGzip([options])

[options](#options)으로 [Gzip](#zlib.Gzip) 객채를 새로 만들어 리턴한다.

## zlib.createGunzip([options])

[options](#options)으로 [Gunzip](#zlib.Gunzip) 객채를 새로 만들어 리턴한다.

## zlib.createDeflate([options])

[options](#options)으로 [Deflate](#zlib.Deflate) 객채를 새로 만들어 리턴한다.

## zlib.createInflate([options])

[options](#options)으로 [Inflate](#zlib.Inflate) 객채를 새로 만들어 리턴한다.

## zlib.createDeflateRaw([options])

[options](#options)으로 [DeflateRaw](#zlib.DeflateRaw) 객채를 새로 만들어 리턴한다.

## zlib.createInflateRaw([options])

[options](#options)으로 [InflateRaw](#zlib.InflateRaw) 객채를 새로 만들어 리턴한다.

## zlib.createUnzip([options])

[options](#options)으로 [Unzip](#zlib.Unzip) 객채를 새로 만들어 리턴한다.

## Class: zlib.Gzip

gzip으로 데이터를 압축한다.

## Class: zlib.Gunzip

gzip 스트림의 압축을 푼다.

## Class: zlib.Deflate

deflate로 데이터를 압축한다.

## Class: zlib.Inflate

deflate 스트림의 압축을 푼다.

## Class: zlib.DeflateRaw

deflate로 데이터를 압축하지만 zlib 헤더는 넣지 않는다.

## Class: zlib.InflateRaw

raw deflate 스트림의 압축을 푼다.

## Class: zlib.Unzip

Gzip-이나 Deflate-로 압축한 스트림의 헤더를 자동으로 찾아서 압축을 푼다.

## Convenience Methods

<!--type=misc-->

여기에 있는 모든 메소드는 첫번째 인자로 버퍼나 스트링을 받는다. 그리고 콜백도 `callback(error, result)` 형식으로 호출한다. 압축/압축해제 엔진은 기본 설정으로 생성하고 다른 옵션으로 생성하고 싶으면 zlib 클래스를 직접사용해야 한다.

## zlib.deflate(buf, callback)

Deflate로 스트링을 압축한다.

## zlib.deflateRaw(buf, callback)

DeflateRaw로 스트링을 압축한다.

## zlib.gzip(buf, callback)

Gzip으로 스트링을 압축한다.

## zlib.gunzip(buf, callback)

Gunzip으로 Buffer의 압축을 푼다.

## zlib.inflate(buf, callback)

Inflate로 Buffer의 압축을 푼다.

## zlib.inflateRaw(buf, callback)

InflateRaw로 Buffer의 압축을 푼다.

## zlib.unzip(buf, callback)

Unzip으로 Buffer의 압축을 푼다.

## Options

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
