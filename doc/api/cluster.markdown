# Cluster

    Stability: 1 Experimental - Drastic changes in future versions

A single instance of Node runs in a single thread. To take advantage of
multi-core systems the user will sometimes want to launch a cluster of Node
processes to handle the load.

The cluster module allows you to easily create a network of processes that
all share server ports.

    var cluster = require('cluster');
    var http = require('http');
    var numCPUs = require('os').cpus().length;

    if (cluster.isMaster) {
      // Fork workers.
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('death', function(worker) {
        console.log('worker ' + worker.pid + ' died');
      });
    } else {
      // Worker processes have a http server.
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
      }).listen(8000);
    }

Running node will now share port 8000 between the workers:

    % node server.js
    Worker 2438 online
    Worker 2437 online

The difference between `cluster.fork()` and `child_process.fork()` is simply
that cluster allows TCP servers to be shared between workers. `cluster.fork`
is implemented on top of `child_process.fork`. The message passing API that
is available with `child_process.fork` is available with `cluster` as well.
As an example, here is a cluster which keeps count of the number of requests
in the master process via message passing:

    var cluster = require('cluster');
    var http = require('http');
    var numReqs = 0;

    if (cluster.isMaster) {
      // Fork workers.
      for (var i = 0; i < 2; i++) {
        var worker = cluster.fork();

        worker.on('message', function(msg) {
          if (msg.cmd && msg.cmd == 'notifyRequest') {
            numReqs++;
          }
        });
      }

      setInterval(function() {
        console.log("numReqs =", numReqs);
      }, 1000);
    } else {
      // Worker processes have a http server.
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
        // Send message to master process
        process.send({ cmd: 'notifyRequest' });
      }).listen(8000);
    }



    안정성: 1 Experimental - 차후 버전에서 많은 변경이 있을 예정

Node 프로세스 하나는 쓰레드 하나로 동작한다. 멀티 코어 시스템을 이용해서 부하를 처리하려면 Node 프로세스를 여러개 띄울 필요가 있다.

이 cluster 모듈은 server ports를 공유하는 프로세스 다발을 쉽게 만들수 있게 해준다.

    var cluster = require('cluster');
    var http = require('http');
    var numCPUs = require('os').cpus().length;

    if (cluster.isMaster) {
      // 워커 생성.
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('death', function(worker) {
        console.log('worker ' + worker.pid + ' died');
      });
    } else {
      // 워커 프로세스가 http 서버를 가진다.
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
      }).listen(8000);
    }

이 프로그램을 실행하면 워커는 8000번 포트를 공유한다:

    % node server.js
    Worker 2438 online
    Worker 2437 online

`cluster.fork()`와 `child_process.fork()`의 차이점은 간단하다. cluster는 워커들이 TCP 서버를 공유할 수 있게 해준다. `cluster.fork`는 `child_process.fork` 위에 구현한 것이라서 `child_process.fork` 에서 사용하는 메시지 패싱 API는 `cluster`에서도 사용할 수 있다. 다음은 메시지 패싱 API를 통해서 마스터 프로세스가 요청을 받은 수를 세는 cluster 예제이다.

    var cluster = require('cluster');
    var http = require('http');
    var numReqs = 0;

    if (cluster.isMaster) {
      // 워커 생성.
      for (var i = 0; i < 2; i++) {
        var worker = cluster.fork();

        worker.on('message', function(msg) {
          if (msg.cmd && msg.cmd == 'notifyRequest') {
            numReqs++;
          }
        });
      }

      setInterval(function() {
        console.log("numReqs =", numReqs);
      }, 1000);
    } else {
      // 워커 프로세스가 http 서버를 가진다.
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
        // 마스터 프로세스에 메시지를 보낸다
        process.send({ cmd: 'notifyRequest' });
      }).listen(8000);
    }



## cluster.fork()

Spawn a new worker process. This can only be called from the master process.

새 워커 프로세스를 띄운다. 이 함수는 마스터 프로세스에서만 호출할 수 있다.

## cluster.isMaster
## cluster.isWorker

Boolean flags to determine if the current process is a master or a worker
process in a cluster. A process `isMaster` if `process.env.NODE_WORKER_ID`
is undefined.

현 프로세스가 마스터 프로세스인지 워커 프로세스인지 판별하는 이진 플래그. 프로세스가 `isMaster`라면 `process.env.NODE_WORKER_ID`는 undefined가 된다.

## Event: 'death'

When any of the workers die the cluster module will emit the 'death' event.
This can be used to restart the worker by calling `fork()` again.

    cluster.on('death', function(worker) {
      console.log('worker ' + worker.pid + ' died. restart...');
      cluster.fork();
    });

Different techniques can be used to restart the worker depending on the
application.

워커가 죽을 때 cluster 모듈에는 `death` 이벤트가 발생한다. 이 이벤트가 발생하면 `fork()`를 다시 호출해서 워커를 재시작시킬 수 있다.

    cluster.on('death', function(worker) {
      console.log('worker ' + worker.pid + ' died. restart...');
      cluster.fork();
    });

다른 방법으로 애플리케이션에 의존하는 워커를 재시작할 수도 있다.
