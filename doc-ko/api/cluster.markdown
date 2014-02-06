# Cluster

    Stability: 1 - Experimental

Node 프로세스 하나는 스레드 하나로 동작한다. 멀티 코어 시스템을 이용해서 부하를 처리하려면
Node 프로세스를 여러 개 띄울 필요가 있다.

이 cluster 모듈은 서버 포트를 공유하는 프로세스들의 네트워크를 쉽게 만들 수 있게 해준다.

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

이 프로그램에서 워커는 8000번 포트를 공유한다.

    % NODE_DEBUG=cluster node server.js
    23521,Master Worker 23524 online
    23521,Master Worker 23526 online
    23521,Master Worker 23523 online
    23521,Master Worker 23528 online


이 기능은 최근에 추가돼서 앞으로 변경될 수 있다. 해보고 안되면 피드백을 주길 바란다.

Windows에서는 이름있는 파이프 서버를 만들 수 없으므로 주의해야 한다.

## How It Works

<!--type=misc-->

워커 프로세스는 `child_process.fork` 메소드로 생성(spawn)하기 때문에 IPC로 부모 프로세스와
통신하고 서버 핸들을 서로 주고받을 수 있다.

워커에서 `server.listen(...)`를 호출하면 아규먼트를 직열화하고 요청을 마스터 프로세스에 보낸다.
마스터 프로세스에 있는 리스닝 서버가 워커의 요구사항에 맞으면 그 핸들을 워커에 넘긴다. 만약 워커에
필요한 리스닝 서버가 없으면 하나 만들어서 워커에 핸들을 넘긴다.

그래서 극단적인 상황에서(edge case) 다음과 같은 행동을 보인다.:

1. `server.listen({fd: 7})` 메시지를 마스터에 보내기 때문에 부모에서 7번 파일 디스크립터를
   Listen하기 시작하고 그 핸들이 워커로 넘겨진다. 7번 파일 디스크립터가 무엇인지 워커에 묻거나
   하는 것이 아니다.
2. `server.listen(handle)` 핸들에 직접 Listen하면 워커는 넘긴 핸들을 사용한다.
   마스터에 메시지를 보내지 않는다. 워커가 이미 핸들을 가지고 있으면 이렇게 호출하는 사람은
   자신이 무슨 짓을 하는 것인지 알고 있다고 간주한다.
3. `server.listen(0)` 이렇게 호출하면 서버는 랜덤 포트에 Listen한다. 하지만, cluster에서는
   워커가 `listen(0)`를 호출할 때마다 같은 랜덤 포트를 받는다. 사실 처음에만 랜덤이지
   그 다음부터는 예측할 수 있다. 워커마다 고유한(unique) 포트를 사용하려면 워커 ID를 이용해서
   포트 넘버를 직접 생성하라.

같은 리소스에 `accept()`하는 프로세스가 여러 개이면 운영체제는 매우 효율적으로 Load-balance를
한다. Node.js에는 라우팅 로직이 없고 워커끼리 상태 정보도 공유하지도 않는다. 그러므로 로그인이나
세션 정보 같은 것을 메모리에 너무 과도하게 상주하지 않도록 프로그램을 설계하는 것이 중요하다.

워커는 서로 독립적인 프로세스이므로 프로그램의 필요에 따라 죽이거나(kill) 재생성(re-spawn)할 수
있다. 워커가 살아 있는 한 새 연결을 계속 수락한다. Node는 워커를 관리해주지 않는다. 애플리케이션에
필요한대로 워커 풀을 관리하는 것은 개발자 책임이다.

## cluster.settings

* {Object}
* `execArgv` {Array} 노드 실행파일에 전달할 문자열 인자 목록
  (기본값=`process.execArgv`)
* `exec` {String} 워커 파일의 경로. (기본값=`process.argv[1]`)
  * `args` {Array} 워커에 넘겨지는 스트링 아규먼트.
    (기본값=`process.argv.slice(2)`)
  * `silent` {Boolean} 워커의 output을 부모의 stdio로 보낼지 말지 여부
    (기본값=`false`)

`.setupMaster()`를 한번만 호출할 수 있으므로 설정된 뒤에는 효과적으로 값이 고정된다.

이 객체를 직접 수정하지 말아야 한다.

## cluster.isMaster

* {Boolean}

프로세스가 마스터이면 true를 리턴한다. 이 프로퍼티는 `process.env.NODE_UNIQUE_ID` 값을
이용하는데 `process.env.NODE_UNIQUE_ID`이 undefined이면 `isMaster`는 true이다.

## cluster.isWorker

* {Boolean}

프로세스가 마스터가 아니면 true이다.(`cluster.isMaster`의 반대다.)

## Event: 'fork'

* `worker` {Worker 객체}

워커가 하나 새로 포크 되면 cluster 모듈은 'fork' 이벤트를 발생(emit)시킨다. 워커의 액티비티
로그를 남기거나 자신만의 타임아웃을 생성하는 데 활용된다.

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

워커를 포크 하면 워커는 '온라인' 메시지를 보낸다. 마스터가 그 '온라인' 메시지를 받으면 이 이벤트를
발생한다. 'fork' 이벤트와 'online' 이벤트의 차이는 간단하다. 'fork' 이벤트는 마스터가
워커 프로세스를 포크 할 때 발생하는 것이고 'online' 이벤트는 워커가 실행 중일 때 발생한다.

    cluster.on('online', function(worker) {
      console.log("Yay, the worker responded after it was forked");
    });

## Event: 'listening'

* `worker` {Worker 객체}
* `address` {Object}

워커에서 `listen()`을 호출한 뒤에 서버에서 'listening' 이벤트가 발생한 경우 마스터의
`cluster`에서 listening 이벤트도 발생할 것이다.

이벤트 핸들러의 아규먼트는 두 개다. `worker`에는 해당 워커 객체가 넘어오고 `address`에는
`address`, `port`, `addressType` 프로퍼티가 있는 `address` 객체가 넘어온다.
이 이벤트는 워커가 하나 이상의 주소를 Listen할 때 유용하다.

    cluster.on('listening', function(worker, address) {
      console.log("A worker is now connected to " + address.address + ":" + address.port);
    });

`addressType`는 다음 값 중 하나이다.

* `4` (TCPv4)
* `6` (TCPv6)
* `-1` (유닉스 도메인 소켓)
* `"udp4"`, `"udp6"` (UDP v4, v6)

## Event: 'disconnect'

* `worker` {Worker 객체}

워커의 IPC 채널의 연결이 끊긴 뒤에 발생한다. 이 이벤트는 워커가 안전하게 종료되거나 죽거나 수동으로
연결이 끊긴(`worker.disconnect()`등으로) 경우에 발생할 수 있다.

`disconnect`와 `exit`이벤트 사이에는 약간의 지연이 있을 수 있다. 프로세스를 정리하면서 멈췄는지
아니면 오래 살아있는 연결이 있는지 검사할 때 이러한 이벤트를 사용할 수 있다.

    cluster.on('disconnect', function(worker) {
      console.log('The worker #' + worker.id + ' has disconnected');
    });

## Event: 'exit'

* `worker` {Worker 객체}
* `code` {Number} 워커의 exit 코드
* `signal` {String} 프로세스를 죽게 만든 신호의 이름(eg. `'SIGHUP'`)

워커가 죽으면 cluster 모듈에 'exit' 이벤트가 발생한다.

그래서 워커가 죽으면 `fork()`를 호출해서 워커를 다시 띄울 수 있다.

    cluster.on('exit', function(worker, code, signal) {
      console.log('worker %d died (%s). restarting...',
        worker.process.pid, signal || code);
      cluster.fork();
    });

[child_process event: 'exit'](child_process.html#child_process_event_exit)를 봐라.

## Event: 'setup'

`.setupMaster()`가 처음 호출될 때 발생한다.

## cluster.setupMaster([settings])

* `settings` {Object}
  * `exec` {String} 워커 파일의 경로.  (기본값=`process.argv[1]`)
  * `args` {Array} 워커에 넘겨지는 스트링 아규먼트.
    (기본값=`process.argv.slice(2)`)
  * `silent` {Boolean} 워커의 output을 부모의 stdio로 보낼지 말지 여부.
    (기본값=`false`)

`setupMaster`는 'fork'의 기본 행동을 수정하는 데 사용한다. 일당 호출되면
`cluster.settings`에서 설정정보가 유지될 것이다.

Note that:

* `.setupMaster()`를 처음 호출할 때만 효과가 있고 이어진 `.setupMaster()` 호출은 무시된다.
* 위의 이유로 워커마다 커스터마이징하는 *유일한* 속성은 `.fork()`함수에서 `env`로 전달해야 한다.
* 기본값을 구성하기 위해서 `.fork()`는 내부적으로 `.setupMaster()`를 호출하므로 반드시
  `.fork()`를 호출하기 *전에* `.setupMaster()`를 호출해야 적용된다.

예제:

    var cluster = require("cluster");
    cluster.setupMaster({
      exec : "worker.js",
      args : ["--use", "https"],
      silent : true
    });
    cluster.fork();

이 함수는 마스터 프로세스에서만 호출할 수 있다.

## cluster.fork([env])

* `env` {Object} 워커 프로세스의 환경변수에 추가할 키/값 쌍.
* return {Worker 객체}

워커 프로세스를 하나 만든다(spawn).

이 함수는 마스터 프로세스에서만 호출할 수 있다.

## cluster.disconnect([callback])

* `callback` {Function} 모든 워커가 Disconnect되고 핸들러가 닫히면 호출되는 함수

`cluster.workers`의 워커마다 `.disconnect()`를 호출한다.

워커의 연결이 끊길 때 내부의 모든 핸들이 닫으면서 대기 중인 다른 이벤트가 없다면
마스터 프로세스를 안전하게 종료할 수 있게 한다.

콜백을 인자로 넘기면 끝날 때 호출된다.

이 함수는 마스터 프로세스에서만 호출할 수 있다.

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

살아있는 워커 객체가 저장되는 해시로 `id`필드가 키다. 모든 워커를 쉽게 순회할 수 있다.
이는 마스터 프로세스에서만 사용할 수 있다.

`'disconnect'`나 `'exit'` 이벤트가 발생하기 바로 직전에 cluster.workers에서 워커를
제거한다.

    // 모든 워커에 적용한다.
    function eachWorker(callback) {
      for (var id in cluster.workers) {
        callback(cluster.workers[id]);
      }
    }
    eachWorker(function(worker) {
      worker.send('big announcement to all workers');
    });

통신으로 워커 참조를 주고받아야 하는 상황이라면 워커 id를 주고받는 것이 가장 좋다.

    socket.on('data', function(id) {
      var worker = cluster.workers[id];
    });

## Class: Worker

워커에 대한 Public 정보와 메소드는 워커 객체에 들어 있다. 마스터에서는
`cluster.workers`로 워커 객체에 접근하고 워커에서는 `cluster.worker`로 접근한다.

### worker.id

* {String}

모든 워커에는 고유한 id가 부여되고 그 값은 id 프로퍼티에 저장된다.

이 id가 clsuter.workers 프로퍼티에서 해당 워커 객체의 인덱스다.
워커가 살아 있는 동안에만 사용할 수 있다.

### worker.process

* {ChildProcess 객체}

워커 프로세스는 `child_process.fork()`로 생성하는 데 이 함수가 반환한 객체는
`.process`에 저장된다. 워커에서 전역 `process`를 저장한다.

참고: [Child Process module](
child_process.html#child_process_child_process_fork_modulepath_args_options)

`process`에서 `'disconnect'`이벤트가 발생하거나 `.suicide`가 `true`가 아니면 해당 워커는
`process.exit(0)`를 호출할 것이다. 이는 의도치 않은 연결종료를 막아준다.

### worker.suicide

* {Boolean}

`.kill()`나 `.disconnect()`를 호출해서 설정한다. 설정되기 전에는 `undefined`다.

불리언 값인 `worker.suicide`는 의도적인 종료와 실수로 종료하는 걸 구분해 준다.
마스터는 이 값에 기반을 둬서 워커를 다시 생성하지 않을지를 선택할 것이다.

    cluster.on('exit', function(worker, code, signal) {
      if (worker.suicide === true) {
        console.log('Oh, it was just suicide\' – no need to worry').
      }
    });

    // 워커를 죽인다
    worker.kill();

`.kill()`를 호출하고 나서 해당 워커가 죽으면 true가 할당되고 `.disconnect()`를 호출하면
즉시 true가 할당된다. 그때까지는 `undefined`이다.

### worker.send(message, [sendHandle])

* `message` {Object}
* `sendHandle` {Handle 객체}

이 함수는 `child_process.fork()`로 생기는 send 메소드와 같다. 마스터에서 특정 워커에
메시지를 보낼 때는 이 함수를 사용한다.

워커에서 `process.send(message)`를 사용할 수도 있지만 사실 같은 함수다.

다음은 마스터가 워커에 보낸 매시지를 다시 그대로 반환하는 echo 예제다.

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

이 함수로 워커를 죽인다. 마스터에서는 `worker.process`의 연결을 종료해서 이를 수행하고 일단
연결이 종료되면 `signal`로 죽인다. 워커에서는 채널의 연결이 종료되고 `0` 코드와 함께 죽는다.

이 함수는 `.suicide`가 설정되도록 한다.

이 메서드는 하위 호환성을 위한 `worker.destroy()`라는 별칭이 존재한다.

워커에 `process.kill()`가 존재하지만 `process.kill()`는
이 함수가 아니고 [kill](process.html#process_process_kill_pid_signal)이다.

### worker.disconnect()

워커에서 이 함수는 모든 서버를 닫고 각 서버의 'close' 이벤트를 기다린 후 IPC 채널의 연결을 종료한다.

마스터에서 내부적인 메시지를 워커에 보내서 워커가 `.disconnect()`를 호출하도록 한다.

이 함수는 `.suicide`가 설정되도록 한다.

서버가 닫힌 뒤에는 더는 새로운 연결을 받지 않지만 동작 중인 다른 워커가 연결을 받을 수도 있다.
기존에 존재하는 연결은 평소처럼 닫힐 수 있는 상태가 된다. 더는 연결이 존재하지 않을 때
([server.close()](net.html#net_event_close) 참고) 워커에 대한 IPC 채널은 워커가
안전하게 죽을 수 있도록 닫힐 것이다.

위의 내용은 서버 *연결에만* 적용되고 클라이언트 연결은 워커가 자동으로 닫지 않고
disconnect는 종료하기 전에 닫기 위해서 기다리지 않는다.

워커에 `process.disconnect`가 존재하지만 `process.disconnect`는 이 함수가 아니고
[disconnect](child_process.html#child_process_child_disconnect)이다.

오래 살아있는 서버 연결이 워커의 연결 종료를 막을 수 있으므로(메시지를 보내는 데 유용하다) 연결을
닫기 위해 애플리케이션의 특정 동작을 추가할 수 있다. 일정 시간 후에 `disconnect` 이벤트가
발생하지 않는다면 워커를 죽이도록 타임아웃을 구현하는데도 유용하다.

    if (cluster.isMaster) {
      var worker = cluster.fork();
      var timeout;

      worker.on('listening', function(address) {
        worker.send('shutdown');
        worker.disconnect();
        timeout = setTimeout(function() {
          worker.kill();
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

      process.on('message', function(msg) {
        if(msg === 'shutdown') {
          // initiate graceful close of any connections to server
        }
      });
    }

### Event: 'message'

* `message` {Object}

이 이벤트는 `child_process.fork()`의 것과 같다.

워커에서는 `process.on('message')`를 사용할 수도 있다.

다음은 마스터 프로세스에서 총 요청 수를 세는 예제다. 메시지 시스템을 사용해서 구현한다.

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

`cluster.on('online')` 이벤트와 유사하지만, 해당 워커에만 적용된다.

    cluster.fork().on('online', function() {
      // Worker is online
    });

이는 워커에서 발생하지 않는다.

### Event: 'listening'

* `address` {Object}

`cluster.on('listening')` 이벤트와 유사하지만, 해당 워커에만 적용된다.

    cluster.fork().on('listening', function(address) {
      // Worker is listening
    });

이는 워커에서 발생하지 않는다.

### Event: 'disconnect'

`cluster.on('disconnect')` 이벤트와 유사하지만, 해당 워커에만 적용된다.

    cluster.fork().on('disconnect', function() {
      // Worker has disconnected
    });

### Event: 'exit'

* `code` {Number} 워커의 exit 코드
* `signal` {String} 프로세스를 죽게 만든 신호의 이름(eg. `'SIGHUP'`)

`cluster.on('exit')` 이벤트와 유사하지만, 해당 워커에만 적용된다.

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

### Event: 'error'

이 이벤트는 `child_process.fork()`의 것과 같다.

워커에서는 `process.on('error')`를 사용할 수도 있다.
