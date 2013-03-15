# Addons

애드온은 동적으로 공유 객체를 연결합니다. 애드온은 C나 C++ 라이브러리에 연결할 수 있다.
API는(현 시점에) 여러 가지 라이브러리에 대한 지식을 포함해서 상당히 복잡하다.

 - V8 자바스트립트, C++ 라이브러리. 자바스크립트와 인터페이스로 연결하는데 사용한다:
   객체를 생성하고 함수를 호출하는 등. `v8.h` 헤더파일(Node 소스트리의
   `deps/v8/include/v8.h`)에 주로 문서화되어 있고
   [online](http://izs.me/v8-docs/main.html)에서도 확인할 수 있다.

 - [libuv](https://github.com/joyent/libuv), C 이벤트루프 라이브러리. 파일 디스크립터가
   읽을 수 있게 되기를 기다린다거나 타이머를 기다리거나 받아야할 신호를 기다려야 할 때는
   언제든지 libuv와 인터페이스로 연결할 필요가 있을 것이다. 즉 어떤 I/O라도 수행한다면
   libuv를 사용해야 한다.

 - 내부 Node 라이브러리. 가장 중요한 것은 `node::ObjectWrap` 클래스이다. 아마
   이 클래스에서 파생하기를 원할 것이다.

 - 그 밖에. 무엇을 사용할 수 있는지 알고 싶으면 `deps/`를 봐라.

Node는 실행가능하도록 모든 의존성을 정적으로 컴파일한다. 모듈을 컴파일할 때 이러한
라이브러리의 연결에 대해서 걱정할 필요가 없다.

아래의 예제는 모두 [download](https://github.com/rvagg/node-addon-examples)에서
다운받을 수 있고 자신만의 애드온을 작성할 때 시작지점으로 사용할 수 있다.

## Hello world

다음 자바스크립트 코드와 동일한 작은 애드온을 C++로 작성하면서 시작해 보자.

    module.exports.hello = function() { return 'world'; };

우선 `hello.cc`파일을 생성한다.

    #include <node.h>
    #include <v8.h>

    using namespace v8;

    Handle<Value> Method(const Arguments& args) {
      HandleScope scope;
      return scope.Close(String::New("world"));
    }

    void init(Handle<Object> exports) {
      exports->Set(String::NewSymbol("hello"),
          FunctionTemplate::New(Method)->GetFunction());
    }
    NODE_MODULE(hello, init)

모든 Node 애드온은 초기화 함수를 외부에 노출해야 한다.

    void Initialize (Handle<Object> exports);
    NODE_MODULE(module_name, Initialize)

함수가 아니므로 `NODE_MODULE`뒤에 세미콜론이 없다.(`node.h`를 봐라.)

`module_name`은 최종 바이너리의 파일명과 일치시켜야 한다.(.node 접미사는 제외하고)

소스코드는 바이너리 애드온인 `hello.node`로 빌드되어야 한다. 이를 위해서
JSON과 유사한 형식으로 모듈의 빌드 설정을 나타내는 `binding.gyp`파일을 생성해야 한다.
이 파일은 [node-gyp](https://github.com/TooTallNate/node-gyp)가 컴파일한다.

    {
      "targets": [
        {
          "target_name": "hello",
          "sources": [ "hello.cc" ]
        }
      ]
    }

다음 과정은 현재 플랫폼에 적절한 프로젝트 빌드 파일을 생성하는 것이다.
빌드파일을 생성하기 위해서 `node-gyp configure`를 사용해라.

이제 `build/` 디렉토리에 `Makefile`(Unix 플랫폼)과 `vcxproj`(Windows 플랫폼)가
있을 것이다. 그 다음 `node-gyp build`명령어를 실행한다.

컴파일된 `.node` 바인딩 파일을 얻었다! 컴파일된 파인딩 파일들은 `build/Release/`에 있다.

최근에 빌드한 `hello.node` 모듈을 `require`함으로써 Node 프로젝트 `hello.js`에서 바이너리
애드온을 사용할 수 있다.

    var addon = require('./build/Release/hello');

    console.log(addon.hello()); // 'world'

더 많은 정보는 아래의 패턴들을 보거나 실사용 예제를 보려면
<https://github.com/pietern/hiredis-node>를 봐라.


## Addon patterns

다음은 애드온 개발을 시작할 때 도움이 될만한 애드폰 패턴들이다. 여러 가지 v8 호출에 대해서는
온라인 [v8 reference](http://izs.me/v8-docs/main.html)를 참고하고 핸들, 범위, 함수 템플릿
등과 같이 사용된 여러가지 개념에 대한 설명은 v8의
[Embedder's Guide](http://code.google.com/apis/v8/embed.html)를 참고해라.

이 예제를 사용하려면 `node-gyp`를 사용해서 컴파일해야 한다.
다음과 같은 `binding.gyp` 파일을 생성해라.

    {
      "targets": [
        {
          "target_name": "addon",
          "sources": [ "addon.cc" ]
        }
      ]
    }

하나 이상의 `.cc` 파일이 있는 경우에 `sources` 배열에 파일명을 다음과 같이 추가해라.

    "sources": ["addon.cc", "myexample.cc"]

`binding.gyp` 파일을 준비했으면 애드온을 설정하고 빌드할 수 있다.

    $ node-gyp configure build


### Function arguments

다음 패턴은 자바스크립트 함수 호출에서 어떻게 아규먼트들을 읽고 결과를 리턴하는 지 보여준다.
다음 파일이 메인파일이고 소스파일인 `addon.cc`만 필요하다.

    #define BUILDING_NODE_EXTENSION
    #include <node.h>

    using namespace v8;

    Handle<Value> Add(const Arguments& args) {
      HandleScope scope;

      if (args.Length() < 2) {
        ThrowException(Exception::TypeError(String::New("Wrong number of arguments")));
        return scope.Close(Undefined());
      }

      if (!args[0]->IsNumber() || !args[1]->IsNumber()) {
        ThrowException(Exception::TypeError(String::New("Wrong arguments")));
        return scope.Close(Undefined());
      }

      Local<Number> num = Number::New(args[0]->NumberValue() +
          args[1]->NumberValue());
      return scope.Close(num);
    }

    void Init(Handle<Object> exports) {
      exports->Set(String::NewSymbol("add"),
          FunctionTemplate::New(Add)->GetFunction());
    }

    NODE_MODULE(addon, Init)

다음 자바스크립트 코드로 이를 테스트할 수 있다:

    var addon = require('./build/Release/addon');

    console.log( 'This should be eight:', addon.add(3,5) );


### Callbacks

C++ 함수에 자바스크립트 함수를 전달해서 C++ 함수에서 자바스크립트 함수를 실행할 수
있다. 다음은 `addon.cc`이다:

    #define BUILDING_NODE_EXTENSION
    #include <node.h>

    using namespace v8;

    Handle<Value> RunCallback(const Arguments& args) {
      HandleScope scope;

      Local<Function> cb = Local<Function>::Cast(args[0]);
      const unsigned argc = 1;
      Local<Value> argv[argc] = { Local<Value>::New(String::New("hello world")) };
      cb->Call(Context::GetCurrent()->Global(), argc, argv);

      return scope.Close(Undefined());
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      module->Set(String::NewSymbol("exports"),
          FunctionTemplate::New(RunCallback)->GetFunction());
    }

    NODE_MODULE(addon, Init)

이 예제는 전체 `module` 객체를 두번째 인자로 받는 두개의 인자를 갖는 형식의 `Init()`를
사용한다. 이는 애드온이 `exports`의 프로퍼티로 함수를 추가하는 대신 하나의 함수로
`exports`를 완전히 덮어쓸 수 있게 한다.

다음 자바스크립트 코드를 실행해서 이를 테스트 할 수 있다:

    var addon = require('./build/Release/addon');

    addon(function(msg){
      console.log(msg); // 'hello world'
    });


### Object factory

`createObject()`에 전달된 문자열을 출력하는 `msg` 프로퍼티를 가진 객체를 리턴하는
이 `addon.cc` 패턴과 함께 C++ 함수내에서 새로운 객체를 생성해서 리턴할 수 있다.

    #define BUILDING_NODE_EXTENSION
    #include <node.h>

    using namespace v8;

    Handle<Value> CreateObject(const Arguments& args) {
      HandleScope scope;

      Local<Object> obj = Object::New();
      obj->Set(String::NewSymbol("msg"), args[0]->ToString());

      return scope.Close(obj);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      module->Set(String::NewSymbol("exports"),
          FunctionTemplate::New(CreateObject)->GetFunction());
    }

    NODE_MODULE(addon, Init)

자바스크립트에서 다음과 같이 테스트한다:

    var addon = require('./build/Release/addon');

    var obj1 = addon('hello');
    var obj2 = addon('world');
    console.log(obj1.msg+' '+obj2.msg); // 'hello world'


### Function factory

이 패턴은 C++ 함수를 감싸는 자바스크립트 함수를 어떻게 생성하고 리턴하는지 보여준다:

    #define BUILDING_NODE_EXTENSION
    #include <node.h>

    using namespace v8;

    Handle<Value> MyFunction(const Arguments& args) {
      HandleScope scope;
      return scope.Close(String::New("hello world"));
    }

    Handle<Value> CreateFunction(const Arguments& args) {
      HandleScope scope;

      Local<FunctionTemplate> tpl = FunctionTemplate::New(MyFunction);
      Local<Function> fn = tpl->GetFunction();
      fn->SetName(String::NewSymbol("theFunction")); // omit this to make it anonymous

      return scope.Close(fn);
    }

    void Init(Handle<Object> exports, Handle<Object> module) {
      module->Set(String::NewSymbol("exports"),
          FunctionTemplate::New(CreateFunction)->GetFunction());
    }

    NODE_MODULE(addon, Init)


다음과 같이 테스트한다:

    var addon = require('./build/Release/addon');

    var fn = addon();
    console.log(fn()); // 'hello world'


### Wrapping C++ objects

`new` 오퍼레이터로 자바스크립트에서 인스턴스화할 수 있는 `MyObject` C++ 객체/클래스에 대한
랩퍼(wrapper)를 생성할 것이다. 우선 메인 모듈 `addon.cc`를 준비하자.

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    void InitAll(Handle<Object> exports) {
      MyObject::Init(exports);
    }

    NODE_MODULE(addon, InitAll)

그 다음 `myobject.h`는 랩퍼가 `node::ObjectWrap`를 상속받도록 한다:

    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init(v8::Handle<v8::Object> exports);

     private:
      MyObject();
      ~MyObject();

      static v8::Handle<v8::Value> New(const v8::Arguments& args);
      static v8::Handle<v8::Value> PlusOne(const v8::Arguments& args);
      double counter_;
    };

    #endif

그리고 `myobject.cc`에서 노출할 다양한 메서드를 구현한다.
여기서 생성자의 프로토타입에 추가해서 `plusOne` 메서드를 노출했다.

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    MyObject::MyObject() {};
    MyObject::~MyObject() {};

    void MyObject::Init(Handle<Object> exports) {
      // Prepare constructor template
      Local<FunctionTemplate> tpl = FunctionTemplate::New(New);
      tpl->SetClassName(String::NewSymbol("MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);
      // Prototype
      tpl->PrototypeTemplate()->Set(String::NewSymbol("plusOne"),
          FunctionTemplate::New(PlusOne)->GetFunction());

      Persistent<Function> constructor = Persistent<Function>::New(tpl->GetFunction());
      exports->Set(String::NewSymbol("MyObject"), constructor);
    }

    Handle<Value> MyObject::New(const Arguments& args) {
      HandleScope scope;

      MyObject* obj = new MyObject();
      obj->counter_ = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      obj->Wrap(args.This());

      return args.This();
    }

    Handle<Value> MyObject::PlusOne(const Arguments& args) {
      HandleScope scope;

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.This());
      obj->counter_ += 1;

      return scope.Close(Number::New(obj->counter_));
    }

다음 코드로 테스트한다:

    var addon = require('./build/Release/addon');

    var obj = new addon.MyObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13


### Factory of wrapped objects

이는 자바스크립트에서 `new` 오퍼레이터로 명시적인 인스턴스화 없이 네이티브 객체를
생성할 수 있도록 하고 싶을 때 유용하다.

    var obj = addon.createObject();
    // 대신에:
    // var obj = new addon.Object();

`addon.cc`에 `createObject` 메서드를 등록하자:

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Handle<Value> CreateObject(const Arguments& args) {
      HandleScope scope;
      return scope.Close(MyObject::NewInstance(args));
    }

    void InitAll(Handle<Object> exports, Handle<Object> module) {
      MyObject::Init();

      module->Set(String::NewSymbol("exports"),
          FunctionTemplate::New(CreateObject)->GetFunction());
    }

    NODE_MODULE(addon, InitAll)

`myobject.h`에서 객체의 인스턴스화를 처리하는 정적 메서드 `NewInstance`를 도입한다.
(예를 들어 자바스크립트에서 `new`가 하는 일이다.)

    #define BUILDING_NODE_EXTENSION
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static v8::Handle<v8::Value> NewInstance(const v8::Arguments& args);

     private:
      MyObject();
      ~MyObject();

      static v8::Persistent<v8::Function> constructor;
      static v8::Handle<v8::Value> New(const v8::Arguments& args);
      static v8::Handle<v8::Value> PlusOne(const v8::Arguments& args);
      double counter_;
    };

    #endif

`myobject.cc`에서 구현체는 위와 유사하다:

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    MyObject::MyObject() {};
    MyObject::~MyObject() {};

    Persistent<Function> MyObject::constructor;

    void MyObject::Init() {
      // 생성자 템플릿 준비
      Local<FunctionTemplate> tpl = FunctionTemplate::New(New);
      tpl->SetClassName(String::NewSymbol("MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);
      // 프로토타입
      tpl->PrototypeTemplate()->Set(String::NewSymbol("plusOne"),
          FunctionTemplate::New(PlusOne)->GetFunction());

      constructor = Persistent<Function>::New(tpl->GetFunction());
    }

    Handle<Value> MyObject::New(const Arguments& args) {
      HandleScope scope;

      MyObject* obj = new MyObject();
      obj->counter_ = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      obj->Wrap(args.This());

      return args.This();
    }

    Handle<Value> MyObject::NewInstance(const Arguments& args) {
      HandleScope scope;

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Object> instance = constructor->NewInstance(argc, argv);

      return scope.Close(instance);
    }

    Handle<Value> MyObject::PlusOne(const Arguments& args) {
      HandleScope scope;

      MyObject* obj = ObjectWrap::Unwrap<MyObject>(args.This());
      obj->counter_ += 1;

      return scope.Close(Number::New(obj->counter_));
    }

다음으로 테스트한다:

    var createObject = require('./build/Release/addon');

    var obj = createObject(10);
    console.log( obj.plusOne() ); // 11
    console.log( obj.plusOne() ); // 12
    console.log( obj.plusOne() ); // 13

    var obj2 = createObject(20);
    console.log( obj2.plusOne() ); // 21
    console.log( obj2.plusOne() ); // 22
    console.log( obj2.plusOne() ); // 23


### Passing wrapped objects around

C++ 객체를 감싸고 리턴하는 부분에 대해서 추가적으로 Node의 `node::ObjectWrap::Unwrap`
헬퍼 함수로 이 객체들을 풀어줌으로써(unwrapping) 전달할 수 있다.
다음 `addon.cc`에서 두 `MyObject` 객체받을 수 있는 `add()` 함수를 도입한다.

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    Handle<Value> CreateObject(const Arguments& args) {
      HandleScope scope;
      return scope.Close(MyObject::NewInstance(args));
    }

    Handle<Value> Add(const Arguments& args) {
      HandleScope scope;

      MyObject* obj1 = node::ObjectWrap::Unwrap<MyObject>(
          args[0]->ToObject());
      MyObject* obj2 = node::ObjectWrap::Unwrap<MyObject>(
          args[1]->ToObject());

      double sum = obj1->Val() + obj2->Val();
      return scope.Close(Number::New(sum));
    }

    void InitAll(Handle<Object> exports) {
      MyObject::Init();

      exports->Set(String::NewSymbol("createObject"),
          FunctionTemplate::New(CreateObject)->GetFunction());

      exports->Set(String::NewSymbol("add"),
          FunctionTemplate::New(Add)->GetFunction());
    }

    NODE_MODULE(addon, InitAll)

흥미롭게 `myobject.h`에서 퍼블릭 메서드를 도입해서 객체를 풀어버린(unwrapping) 후
private 값을 자세히 조사할 수 있다:

    #define BUILDING_NODE_EXTENSION
    #ifndef MYOBJECT_H
    #define MYOBJECT_H

    #include <node.h>

    class MyObject : public node::ObjectWrap {
     public:
      static void Init();
      static v8::Handle<v8::Value> NewInstance(const v8::Arguments& args);
      double Val() const { return val_; }

     private:
      MyObject();
      ~MyObject();

      static v8::Persistent<v8::Function> constructor;
      static v8::Handle<v8::Value> New(const v8::Arguments& args);
      double val_;
    };

    #endif

`myobject.cc`의 구현체는 이전과 유사하다:

    #define BUILDING_NODE_EXTENSION
    #include <node.h>
    #include "myobject.h"

    using namespace v8;

    MyObject::MyObject() {};
    MyObject::~MyObject() {};

    Persistent<Function> MyObject::constructor;

    void MyObject::Init() {
      // 생성자 템플릿 준비
      Local<FunctionTemplate> tpl = FunctionTemplate::New(New);
      tpl->SetClassName(String::NewSymbol("MyObject"));
      tpl->InstanceTemplate()->SetInternalFieldCount(1);

      constructor = Persistent<Function>::New(tpl->GetFunction());
    }

    Handle<Value> MyObject::New(const Arguments& args) {
      HandleScope scope;

      MyObject* obj = new MyObject();
      obj->val_ = args[0]->IsUndefined() ? 0 : args[0]->NumberValue();
      obj->Wrap(args.This());

      return args.This();
    }

    Handle<Value> MyObject::NewInstance(const Arguments& args) {
      HandleScope scope;

      const unsigned argc = 1;
      Handle<Value> argv[argc] = { args[0] };
      Local<Object> instance = constructor->NewInstance(argc, argv);

      return scope.Close(instance);
    }

다음으로 테스트한다:

    var addon = require('./build/Release/addon');

    var obj1 = addon.createObject(10);
    var obj2 = addon.createObject(20);
    var result = addon.add(obj1, obj2);

    console.log(result); // 30
