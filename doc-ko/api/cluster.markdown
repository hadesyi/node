# Cluster

    Stability: 1 - Experimental

Node 프로세스 하나는 쓰레드 하나로 동작한다. 멀티 코어 시스템을 이용해서 부하를 처리하려면 Node 프로세스를 여러 개 띄울 필요가 있다.

이 cluster 모듈은 서버 포트를 공유하는 프로세스 다발을 쉽게 만들 수 있게 해준다.

    var cluster = require('cluster');
    var http = require('http');
    var numCPUs = require('os').cpus().length;

    if (cluster.isMaster) {
      // 워커를 포크한다.
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('exit', function(worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' died');
      });
    } else {
      // 워커는 TCP 연결을 공유할 수 있다.
      // 이 예제는 HTTP 서버이다.
      http.createServer(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");
      }).listen(8000);
    }

이 프로그램에서 워커는 8000번 포트를 공유한다:

    % NODE_DEBUG=cluster node server.js
    23521,Master Worker 23524 online
    23521,Master Worker 23526 online
    23521,Master Worker 23523 online
    23521,Master Worker 23528 online


이 기능은 최근에 추가돼서 앞으로 변경될 수 있다. 해보고 안되면 피드백을 주길 바란다.

Windows에서는 네임드 파이프 서버를 만들 수 없으므로 주의해야 한다.

## How It Works

<!--type=misc-->

워커 프로세스는 `child_process.fork` 메소드로 생성(spawn)하기 때문에 IPC로 부모 프로세스와 통신하고 서버 핸들을 서로 주고받을 수 있다.

워커에서 `server.listen(...)`를 호출하면 아규먼트를 직열화하고 요청을 마스터 프로세스에 보낸다. 마스터 프로세스에 있는 리스닝 서버가 워커의 요구사항에 맞으면 그 핸들을 워커에 넘긴다. 만약 워커에 필요한 리스닝 서버가 없으면 하나 만들어서 자식 프로세스에 핸들을 넘긴다.

그래서 극단적인 상황에서(edge case) 다음과 같은 행동을 보인다.:

1. `server.listen({fd: 7})` 메시지를 마스터에 보내기 때문에 부모에서 7번 파일 디스크립터를 Listen하기 시작하고 그 핸들이 워커로 넘겨진다. 7번 파일 디스크립터가 무엇인지 워커에 묻거나 하는 것이 아니다.
2. `server.listen(handle)` 핸들에 직접 Listen하면 워커는 넘긴 핸들을 사용한다. 마스터에 메시지를 보내지 않는다. 워커가 이미 핸들을 가지고 있으면 이렇게 호출하는 사람은 자신이 무슨짓을 하는 것인지 알고 있다고 간주된다.
3. `server.listen(0)` 이렇게 호출하면 서버는 랜덤 포트에 Listen한다. 하지만, cluster에서는 워커가 `listen(0)`를 호출할 때마다 동일한 랜덤 포트를 받는다. 사실 처음에만 랜덤이지 그다음부터는 예측 가능하다. 워커마다 고유한(unique) 포트를 사용하려면 워커 ID를 이용해서 포트 넘버를 직접 생성하라.

같은 리소스에 `accept()`하는 프로세스가 여러 개이면 운영체제는 매우 효율적으로 Load-balance를 한다. Node.js에는 라우팅 로직이 없고 워커끼리 상태 정보도 공유하지도 않는다. 그러므로 로그인이나 세션 정보 같은 것을 메모리에 너무 과도하게 상주하지 않도록 프로그램을 설계하는 것이 중요하다.

워커는 서로 독립적인 프로세스이므로 프로그램의 필요에 따라 죽이거나(kill) 재생성(re-spawn)할 수 있다. 워커가 살아 있는 한 새 연결을 계속 수락한다. Node는 워커를 관리해주지 않는다. 애플리케이션에 필요한대로 워커 풀을 관리하는 것은 개발자 책임이다.

## cluster.settings

* {Object}
* `exec` {String} 워커 파일의 경로.  (Default=`__filename`)
  * `args` {Array} 워커에 넘겨지는 스트링 아규먼트.
    (Default=`process.argv.slice(2)`)
  * `silent` {Boolean} 워커의 output을 부모의 stdio로 보낼지 말지.
    (Default=`false`)

`.setupMaster()` 메소드로 설정하면 이 settings 객체에 저장된다. 이 객체를 직접 수정하지 말아야 한다.

## cluster.isMaster

* {Boolean}

프로세스가 마스터이면 true를 리턴한다. 이 프로퍼티는 `process.env.NODE_UNIQUE_ID` 값을 이용하는데 `process.env.NODE_UNIQUE_ID`이 undefined이면 `isMaster`는 true이다.

## cluster.isWorker

* {Boolean}

해당 프로세스가 워커면 true를 리턴한다. 이 프로퍼티는 `process.env.NODE_UNIQUE_ID` 값을 이용하는데 `process.env.NODE_UNIQUE_ID` 프로퍼티에 값이 할당돼 있으면 true이다.

## Event: 'fork'

* `worker` {Worker 객체}

워커가 하나 새로 포크되면 cluster 모듈은 'fork' 이벤트를 발생(emit)시킨다. 워커의 액티비티 로그를 남기거나 타임아웃을 생성하는 데 활용된다.

    var timeouts = [];
    function errorMsg() {
      console.error("Something must be wrong with the connection ...");
    }

    cluster.on('fork', function(worker) {
      timeouts[worker.id] = setTimeout(errorMsg, 2000);
    });
    cluster.on('listening', function(worker, address) {
      clearTimeout(timeouts[worker.id]);
    });
    cluster.on('exit', function(worker, code, signal) {
      clearTimeout(timeouts[worker.id]);
      errorMsg();
    });

## Event: 'online'

* `worker` {Worker 객체}

워커를 포크하면 워커는 '온라인' 메시지를 보낸다. 마스터가 그 '온라인' 메시지를 받으면 이 이벤트를 발생한다. 'fork' 이벤트와 'online' 이벤트의 차이는 간단하다. 'fork' 이벤트는 마스터가 워커 프로세스를 포크할 때 발생하는 것이고 'online' 이벤트는 워커가 실행되면 발생한다.

    cluster.on('online', function(worker) {
      console.log("Yay, the worker responded after it was forked");
    });

## Event: 'listening'

* `worker` {Worker 객체}
* `address` {Object}

워커에서 `listen()`을 호출하면 자동으로 'listening' 이벤트가 서버 인스턴스에 발생한다. 서버가 listening 중이면 listening' 이벤트가 발생한 마스터는 메시지를 하나 받는다.

이벤트 핸들러의 아규먼트는 두 개다. `worker`에는 해당 워커 객체가 넘어오고 `address`에는 `address`, `port`, `addressType` 프로퍼티가 있는 `address` 객체가 넘어온다. 이 이벤트는 워커가 하나 이상의 주소를 Listen할 때 유용하다.

    cluster.on('listening', function(worker, address) {
      console.log("A worker is now connected to " + address.address + ":" + address.port);
    });

## Event: 'disconnect'

* `worker` {Worker 객체}

워커의 IPC 채널이 끊기면 이 이벤트가 발생한다. 워커가 죽을 때도 이 이벤트가 발생한다. `.kill()`를 호출해서 워커가 죽을 때도 발생한다.

`.disconnect()`를 호출하면 `disconnect`와 `exit` 이벤트 사이에 약간의 딜레이가 있을 수 있다. 이 이벤트는 아직 살아있는 연결이 있는지 확인하거나 프로세스가 소거 중인지 확인할 때 유용하다:

    cluster.on('disconnect', function(worker) {
      console.log('The worker #' + worker.id + ' has disconnected');
    });

## Event: 'exit'

* `worker` {Worker 객체}
* `code` {Number} 워커의 exit 코드
* `signal` {String} 프로세스를 죽게 만든 시그널의 이름(eg. `'SIGHUP'`)

워커가 죽으면 cluster 모듈에 'exit' 이벤트가 발생한다. 그래서 워커가 죽으면 `fork()`를 호출해서 워커를 다시 띄울 수 있다.

    cluster.on('exit', function(worker, code, signal) {
      var exitCode = worker.process.exitCode;
      console.log('worker ' + worker.process.pid + ' died ('+exitCode+'). restarting...');
      cluster.fork();
    });

## Event: 'setup'

* `worker` {Worker 객체}

`.setupMaster()` 함수를 실행하면 이 이벤트가 발생한다. `fork()`를 호출하기 전에 `.setupMaster()`가 한 번도 실행된 적이 없으면 `fork()`를 실행할 때 아규먼트 없이 `.setupMaster()`가 한번 호출된다.

## cluster.setupMaster([settings])

* `settings` {Object}
  * `exec` {String} 워커 파일의 경로.  (Default=`__filename`)
  * `args` {Array} 워커에 넘겨지는 스트링 아규먼트.
    (Default=`process.argv.slice(2)`)
  * `silent` {Boolean} 워커의 output을 부모의 stdio로 보낼지 말지.
    (Default=`false`)

`setupMaster`는 'fork'의 기본 행동을 수정하는데 사용한다. 사실상 새로운 설정은 즉각적이고
영구적이라서 나중에 이를 수정할 수 없다.

예제:

    var cluster = require("cluster");
    cluster.setupMaster({
      exec : "worker.js",
      args : ["--use", "https"],
      silent : true
    });
    cluster.fork();

## cluster.fork([env])

* `env` {Object} 자식 프로세스의 환경변수, Key/value
* return {Worker 객체}

워커 프로세스를 하나 만든다(spawn). 이 함수는 마스터 프로세스에서만 호출할 수 있다.

## cluster.disconnect([callback])

* `callback` {Function} 모든 워커가 Disconnect되고 핸들러가 닫히면 호출되는 함수

이 메소드를 호출하면 워커가 전부 정상(graceful) 종료한다. 워커가 종료하면서 내부 핸들러도 닫힌다. 그래서 마스터 프로세스는 이벤트를 기다리는 것 없이 정상(graceful) 종료될 수 있다.

콜백을 아규먼트로 넘기면 끝날 때 호출된다.

## cluster.worker

* {Object}

현재 워커 객체에 대한 참조. 마스터 프로세스에서는 사용할 수 없다.

    var cluster = require('cluster');

    if (cluster.isMaster) {
      console.log('I am master');
      cluster.fork();
      cluster.fork();
    } else if (cluster.isWorker) {
      console.log('I am worker #' + cluster.worker.id);
    }

## cluster.workers

* {Object}

살아있는 워커 객체가 저장되는 해쉬로 `id`필드가 키다. 모든 워커를 쉽게 순회할 수 있다.
이는 마스터 프로세스에서만 사용할 수 있다.

    // 모든 워커에 적용한다.
    function eachWorker(callback) {
      for (var id in cluster.workers) {
        callback(cluster.workers[id]);
      }
    }
    eachWorker(function(worker) {
      worker.send('big announcement to all workers');
    });

통신으로 워커 레퍼런스를 주고받아야 하는 상황이라면 워커 id를 주고받는 것이 가장 좋다.

    socket.on('data', function(id) {
      var worker = cluster.workers[id];
    });

## Class: Worker

워커에 대한 Public 정보와 메소드는 워커 객체에 들어 있다. 마스터에서는 `cluster.workers`로 워커 객체에 접근하고 워커에서는 `cluster.worker`로 접근한다.

### worker.id

* {String}

모든 워커에는 고유한 id가 부여되고 그 값은 id 프로퍼티에 저장된다.

이 id가 clsuter.workers 프로퍼티에서 해당 워커 객체의 인덱스다. 워커가 살아 있는 동안에만 사용할 수 있다.

### worker.process

* {ChildProcess 객체}

워커 프로세스는 `child_process.fork()`로 생성하는 데 이 함수가 리턴한 객체가 process 프로퍼티에 저장된다.

See: [Child Process module](child_process.html)

### worker.suicide

* {Boolean}

`.kill()`를 호출하고 나서 해당 워커가 죽으면 true가 할당되고 `.disconnect()`를 호출하면 즉시 true가 할당된다. 그때까지는 `undefined`이다.

### worker.send(message, [sendHandle])

* `message` {Object}
* `sendHandle` {Handle 객체}

이 함수는 `child_process.fork()`로 생기는 send 메소드와 동일하다. 마스터에서 워커에 메시지를 보낼 때는 이 함수로 보내고 워커에서는 `process.send(message)`로 보내지만, 이 둘은 같은 함수다.

다음은 마스터가 워커에 보낸 매시지를 다시 그대로 리턴하는 echo 예제다:

    if (cluster.isMaster) {
      var worker = cluster.fork();
      worker.send('hi there');

    } else if (cluster.isWorker) {
      process.on('message', function(msg) {
        process.send(msg);
      });
    }

### worker.kill([signal='SIGTERM'])

* `signal` {String} 워커 프로세스에 보내는 kill 신호의 이름.

이 함수로 워커를 죽이고 워커를 다시 생성하지 말라고 마스터에게 알릴 수 있다. `suicide` 프로퍼티를 이용하면 워커가 죽은 게 계획적인지 예외적인지 구분할 수 있다.

    cluster.on('exit', function(worker, code, signal) {
      if (worker.suicide === true) {
        console.log('Oh, it was just suicide\' – no need to worry').
      }
    });

    // 워커를 파괴
    worker.kill();

이 메서드는 하위 호환성을 위한 `worker.destroy()`라는 별칭이 존재한다.

### worker.disconnect()

이 함수를 호출하면 해당 워커는 더는 연결을 수락하지 않는다(하지만, 다른 워커는 여전히 연결을 수락한다). 이미 맺어진 연결도 종료할 수 있다. 맺어진 연결이 없으면 IPC 연결이 닫히고 워커가 정상적으로(graceful) 죽는다. IPC 채널이 닫히면 `disconnect` 이벤트가 발생하고 이어서 워커가 죽으면서 `exit` 이벤트가 발생한다.

바로 끊기지 않는 연결이 있을 수도 있기 때문에 타임아웃을 사용하는 게 좋다. 먼저 워커를 Disconnect시키고 2초 후에 서버를 죽인다(destroy). 대신 2초 후에 `worker.kill()` 메소드를 실행할 수도 있지만, 워커가 충분히 소거하지 못할 수 있다.

    if (cluster.isMaster) {
      var worker = cluster.fork();
      var timeout;

      worker.on('listening', function(address) {
        worker.disconnect();
        timeout = setTimeout(function() {
          worker.send('force kill');
        }, 2000);
      });

      worker.on('disconnect', function() {
        clearTimeout(timeout);
      });

    } else if (cluster.isWorker) {
      var net = require('net');
      var server = net.createServer(function(socket) {
        // 연결이 끝나지 않는다.
      });

      server.listen(8000);

      server.on('close', function() {
        // 마무리
      });

      process.on('message', function(msg) {
        if (msg === 'force kill') {
          server.destroy();
        }
      });
    }

### Event: 'message'

* `message` {Object}

이 이벤트는 `child_process.fork()`의 것과 같다. 마스터에서 이 이벤트를 사용하고 워커에서는 `process.on('message')`를 사용한다.

다음은 마스터 프로세스에서 총 요청 수를 세는 예제다. 메시지 시스템을 사용해서 구현한다:

    var cluster = require('cluster');
    var http = require('http');

    if (cluster.isMaster) {

      // HTTP 요청 수를 저장한다.
      var numReqs = 0;
      setInterval(function() {
        console.log("numReqs =", numReqs);
      }, 1000);

      // 요청을 센다.
      function messageHandler(msg) {
        if (msg.cmd && msg.cmd == 'notifyRequest') {
          numReqs += 1;
        }
      }

      // 워커를 생성하고 메시지를 Listen한다.
      var numCPUs = require('os').cpus().length;
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      Object.keys(cluster.workers).forEach(function(id) {
        cluster.workers[id].on('message', messageHandler);
      });

    } else {

      // 워커 프로세스에 HTTP 서버가 있다..
      http.Server(function(req, res) {
        res.writeHead(200);
        res.end("hello world\n");

        // 마스터에 'notifyRequest' 메시지를 보낸다.
        process.send({ cmd: 'notifyRequest' });
      }).listen(8000);
    }

### Event: 'online'

`cluster.on('online')` 이벤트와 같지만, 해당 워커의 상태가 변경됐을 때만 발생한다.

    cluster.fork().on('online', function() {
      // Worker is online
    });

### Event: 'listening'

* `address` {Object}

`cluster.on('listening')` 이벤트와 같지만, 해당 워커의 상태가 변경됐을 때만 발생한다.

    cluster.fork().on('listening', function(address) {
      // Worker is listening
    });

### Event: 'disconnect'

`cluster.on('disconnect')` 이벤트와 같지만, 해당 워커의 상태가 변경됐을 때만 발생한다.

    cluster.fork().on('disconnect', function() {
      // Worker has disconnected
    });

### Event: 'exit'

* `code` {Number} 워커의 exit 코드
* `signal` {String} 프로세스를 죽게 만든 시그널의 이름(eg. `'SIGHUP'`)

워커 프로세스가 종료할 때 발생한다. 자세한 건 [child_process Event: 'exit'](child_process.html#child_process_event_exit)을 봐라.

    var worker = cluster.fork();
    worker.on('exit', function(code, signal) {
      if( signal ) {
        console.log("worker was killed by signal: "+signal);
      } else if( code !== 0 ) {
        console.log("worker exited with error code: "+code);
      } else {
        console.log("worker success!");
      }
    });
