# gRPC 개요
* gRPC에서는 클라이언트가 서버 애플리케이션의 함수를 마치 로컬 객체인 것처럼 호출할 수 있다.
* gRPC는 서비스를 정의하고, 파라미터 및 반환값을 사용하여 원격으로 호출할 수 있는 메소드를 제공한다.
* 서버에서는 해당 인터페이스를 구현하고, gRPC 서버를 실행하여 클라이언트의 호출을 처리한다.
* 클라이언트에서는 Stub 객체를 사용해 gRPC 서버와 통신한다.

<img width="530" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/08283939-044a-4093-867e-7f966df15144">

## Protocol Buffers
* Protocol Buffer는 Google이 공개한 데이터 구조로써, 구조화된 데이터를 직렬화하기 위한 데이터 표현 방식이다.
* Protocol Buffer는 특정 언어 혹은 특정 플랫폼에 종속적이지 않은 데이터 표현 방식이다. (JSON과 비슷한 역할)

**Protocol Buffers 코드 구조**
* Protocol Buffer로 작업하기 위해서는 .proto 파일에 직렬화하려는 데이터의 구조를 정의해야 한다.
* Protocol Buffer 데이터는 message로 구성된다. 각 message는 field라고 하는 일련의 이름-값 쌍을 갖는 논리적 단위이다.
* message 정의 시 field 번호를 속성에 할당해야 한다. Protocal Buffer는 속성 이름 대신 field 번호를 사용해 객체 내 필드 값을 식별한다.
* message를 RPC 메소드의 파라미터 또는 반환값으로 사용하여 proto 파일에서 gRPC 서비스를 정의한다.
```kotlin
service Greeter {
  rpc SayHello (Person) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
```
* 하지만, Protocol Buffer는 특정 언어에 종속적이지 않으므로, Java나 Kotlin, Golang 언어에서 직접적으로 사용할 수 없다.
* Protocol Buffer를 언어에서 독립적으로 활용하기 위해서는 이를 기반으로 Client와 Server에서 사용할 수 있는 Stub 클래스를 생성해야 한다.

## Stub
* protocol buffer compiler인 protoc를 사용하여 proto 정의에서 원하는 언어로 데이터 엑세스 클래스를 생성한다. 이를 Stub 클래스라고 한다.
* Stub은 message의 각 필드에 대한 get/set 메소드와 전체 데이터를 raw bytes로 직렬화/역직렬화하기 위한 메소드를 제공한다.

**Stub을 사용해 Client에서 특정 RPC 호출 예제**
* 클라이언트에서 Stub 클래스를 사용하여 서버와 통신하는 코드 예제이다.
* Stub 클래스를 통해 Person Protocol Buffer message 필드값을 채우고, 직렬화/역직렬화한다.
```kotlin
class HelloWorldClient {
  private val channel: String = "hello_world"  // 해당 proto 파일명
  private val stub: GreeterCoroutineStub = GreeterCoroutineStub(channel)

  fun greet(name: String) {
    val request = helloRequest { this.name = name }
    val response = stub.sayHello(request)  // RPC 호출
  }
}
```

