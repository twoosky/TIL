# gRPC 개요
* gRPC에서는 클라이언트가 서버 애플리케이션의 함수를 마치 로컬 객체인 것처럼 호출할 수 있다.
* gRPC는 서비스를 정의하고, 파라미터 및 반환값을 사용하여 원격으로 호출할 수 있는 메소드를 제공한다.
* 서버에서는 해당 인터페이스를 구현하고, gRPC 서버를 실행하여 클라이언트의 호출을 처리한다.
* 클라이언트에서는 Stub 객체를 사용해 gRPC 서버와 통신한다.

<img width="530" alt="image" src="https://github.com/twoosky/TIL/assets/50009240/08283939-044a-4093-867e-7f966df15144">

## Protocol Buffers
