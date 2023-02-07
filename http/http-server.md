# HTTP Server

* Java ServerSocket
  * 네트워크 상에서 소켓 통신을 위해 Java에서 제공해주는 클래스.

* Blocking vs Non-Blocking
  * Blocking
    * 다른 주체가 수행한 작업으로 인해 기존에 수행된 작업의 제어권이 넘어가는 현상.
    * 제어권이 넘어가는 동안에는 기존에 수행된 작업 주체에서 다른 작업을 수행할 수 없음.
  * Non-Blocking
    * 다른 주체에서 수행되는 작업에 관계없이 기존에 수행된 로직의 제어권이 유지되는 현상.
    * 제어권이 그대로 유지되므로, 기존에 수행된 작업 주체에서 다른 작업 수행 가능.
  * 비효율성을 막기 위해 Callback, Listener 등등을 이용한 비동기 프로그래밍과 이벤트 기반 프로그래밍이 필요함.

<figure><img src="./images/blocking-non-blocking.png" alt=""></figure>

* Java HttpServer
  * Java에서 간단한 HTTP Server를 구현할 때 사용하는 클래스.

* Java NIO(New Input/Output)
  * 채널을 사용하여 양방향 버퍼를 통해 데이터 통신을 진행할 수 있도록 해주는 클래스들의 모음.
  * Non-Blocking 방식의 IO 사용 가능함.

* 참고 링크
  * https://homoefficio.github.io/2017/02/19/Blocking-NonBlocking-Synchronous-Asynchronous/
