# os

<!--english start-->

    Stability: 4 - API Frozen

Provides a few basic operating-system related utility functions.

Use `require('os')` to access this module.

<!--english end-->

    Stability: 4 - API Frozen

OS 유틸리티 함수를 몇 개 제공한다.

이 모듈은 `require('os')`로 접근한다.

## os.hostname()

<!--english start-->

Returns the hostname of the operating system.

<!--english end-->

OS의 호스트 이름을 리턴한다.

## os.type()

<!--english start-->

Returns the operating system name.

<!--english end-->

OS 이름을 리턴한다(역주, 'Linux').

## os.platform()

<!--english start-->

Returns the operating system platform.

<!--english end-->

OS 플랫폼을 리턴한다(역주, 'linux').

## os.arch()

<!--english start-->

Returns the operating system CPU architecture.

<!--english end-->

OS CPU 아키텍처를 리턴한다(역주, 'x64').

## os.release()

<!--english start-->

Returns the operating system release.

<!--english end-->

OS 버전을 리턴한다(역주, '3.2.0-26-generic').

## os.uptime()

<!--english start-->

Returns the system uptime in seconds.

<!--english end-->

시스템 구동시간을 초 단위로 리턴한다(역주, 50515.673530518).

## os.loadavg()

<!--english start-->

Returns an array containing the 1, 5, and 15 minute load averages.

<!--english end-->

1, 5, 15 분 로드 평균값을 배열에 담아 리턴한다.

## os.totalmem()

<!--english start-->

Returns the total amount of system memory in bytes.

<!--english end-->

시스템 메모리의 총량을 바이트 단위로 리턴한다.

## os.freemem()

<!--english start-->

Returns the amount of free system memory in bytes.

<!--english end-->

시스템의 여유 메모리를 바이트 단위로 리턴한다.

## os.cpus()

<!--english start-->

Returns an array of objects containing information about each CPU/core installed: model, speed (in MHz), and times (an object containing the number of CPU ticks spent in: user, nice, sys, idle, and irq).

Example inspection of os.cpus:

    [ { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 252020,
           nice: 0,
           sys: 30340,
           idle: 1070356870,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 306960,
           nice: 0,
           sys: 26980,
           idle: 1071569080,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 248450,
           nice: 0,
           sys: 21750,
           idle: 1070919370,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 256880,
           nice: 0,
           sys: 19430,
           idle: 1070905480,
           irq: 20 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 511580,
           nice: 20,
           sys: 40900,
           idle: 1070842510,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 291660,
           nice: 0,
           sys: 34360,
           idle: 1070888000,
           irq: 10 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 308260,
           nice: 0,
           sys: 55410,
           idle: 1071129970,
           irq: 880 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 266450,
           nice: 1480,
           sys: 34920,
           idle: 1072572010,
           irq: 30 } } ]

<!--english end-->

모든 CPU/코어에 대한 정보를 배열에 담아서 그 배열을 리턴한다. CPU/코어에 대한 정보는 model, speed(MHz 단위), times(user, nide, sys, idle, irq로 분류해서 각각 사용한 CPU 타임(CPU 틱의 수.)이다.

os.cpus의 결과:

    [ { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 252020,
           nice: 0,
           sys: 30340,
           idle: 1070356870,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 306960,
           nice: 0,
           sys: 26980,
           idle: 1071569080,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 248450,
           nice: 0,
           sys: 21750,
           idle: 1070919370,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 256880,
           nice: 0,
           sys: 19430,
           idle: 1070905480,
           irq: 20 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 511580,
           nice: 20,
           sys: 40900,
           idle: 1070842510,
           irq: 0 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 291660,
           nice: 0,
           sys: 34360,
           idle: 1070888000,
           irq: 10 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 308260,
           nice: 0,
           sys: 55410,
           idle: 1071129970,
           irq: 880 } },
      { model: 'Intel(R) Core(TM) i7 CPU         860  @ 2.80GHz',
        speed: 2926,
        times:
         { user: 266450,
           nice: 1480,
           sys: 34920,
           idle: 1072572010,
           irq: 30 } } ]

## os.networkInterfaces()

<!--english start-->

Get a list of network interfaces:

    { lo0: 
       [ { address: '::1', family: 'IPv6', internal: true },
         { address: 'fe80::1', family: 'IPv6', internal: true },
         { address: '127.0.0.1', family: 'IPv4', internal: true } ],
      en1: 
       [ { address: 'fe80::cabc:c8ff:feef:f996', family: 'IPv6',
           internal: false },
         { address: '10.0.1.123', family: 'IPv4', internal: false } ],
      vmnet1: [ { address: '10.99.99.254', family: 'IPv4', internal: false } ],
      vmnet8: [ { address: '10.88.88.1', family: 'IPv4', internal: false } ],
      ppp0: [ { address: '10.2.0.231', family: 'IPv4', internal: false } ] }


<!--english end-->

네트워크 인터페이스의 목록을 리턴한다:

    { lo0: 
       [ { address: '::1', family: 'IPv6', internal: true },
         { address: 'fe80::1', family: 'IPv6', internal: true },
         { address: '127.0.0.1', family: 'IPv4', internal: true } ],
      en1: 
       [ { address: 'fe80::cabc:c8ff:feef:f996', family: 'IPv6',
           internal: false },
         { address: '10.0.1.123', family: 'IPv4', internal: false } ],
      vmnet1: [ { address: '10.99.99.254', family: 'IPv4', internal: false } ],
      vmnet8: [ { address: '10.88.88.1', family: 'IPv4', internal: false } ],
      ppp0: [ { address: '10.2.0.231', family: 'IPv4', internal: false } ] }

