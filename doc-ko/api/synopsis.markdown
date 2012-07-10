# Synopsis

<!--type=misc-->

Node로 작성한 'Hello World'를 응답하는 [web server](http.html) 예제:

    var http = require('http');

    http.createServer(function (request, response) {
      response.writeHead(200, {'Content-Type': 'text/plain'});
      response.end('Hello World\n');
    }).listen(8124);

    console.log('Server running at http://127.0.0.1:8124/');

서버를 실행하려면 `example.js`파일에 코드를 입력하고 node 프로그램으로 실행해라.

    > node example.js
    Server running at http://127.0.0.1:8124/

문서의 모든 예제는 비슷하게 실행할 수 있다.
