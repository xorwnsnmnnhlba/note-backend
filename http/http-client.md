# HTTP Client

* 인터넷 프로토콜 스위트(Internet Protocol Suite)
  * 인터넷 상에서 호스트들이 서로 통신할 때 쓰이는 규약(프로토콜)들의 집합.

* TCP/IP(Transmission Control Protocol/Internet Protocol)
  * 인터넷 상에서 호스트들이 서로 통신할 때 쓰이는 가장 기본적인 프로토콜.
  * 데이터를 주고받을 때 의도한대로 통신이 이루어질 수 있도록 보장해줌.

* TCP와 UDP
  * 전송 계층의 대표적인 프로토콜
  * TCP(Transmission Control Protocol)
    * 연결성 프로토콜로써, 통신하려는 호스트들이 서로 연결을 맺은 후 데이터를 주고받음.
    * 데이터와 패킷 전달 순서가 보장됨.
    * TCP 프로토콜을 이용한 통신절차
      * 준비 단계
        * 서버가 클라이언트의 접속요청을 받을 수 있도록 소켓(대기 소켓)을 열어놓음. -> Listen
        * 클라이언트는 소켓을 열어놓은 후, 서버쪽으로 접속 시도. -> Connect
        * 서버는 클라이언트가 시도한 접속 요청을 받아서 통신을 진행할 소켓(송수신 소켓) 구성. -> Accept
      * 실행 단계
        * 서버는 송수신 소켓을 이용하여 클라이언트와 통신 진행. -> Send & Receive
        * 클라이언트에서 데이터 통신 시 패킷으로 분할 후 전송하며, 수신지까지 올바르게 전송되는지 확인함.
        * 서버에서는 패킷으로 전송받은 데이터를 보낸 순서에 맞춰 재설정 후, 그 데이터들을 다시 조립함.
      * 종료 단계
        * 통신 수행 후, 소켓을 닫음. -> Close
  * UDP(User Datagram Protocol)
    * 비연결성 프로토콜로써, 데이터를 보낼 때 연결을 맺는 작업을 수행하지 않음.
    * 데이터와 패킷 전달 순서가 보장되지 않음.

* 소켓(Socket)
  * 프로세스 간 통신(Inter Process Communication, IPC)의 종착점을 의미함.
  * 호스트 간 네트워크 통신 창구 역할을 수행.
  * 기본적으로 파일과 유사하게 다룰 수 있음.

* 버클리 소켓(Berkeley Sockets, BSD)
  * 프로세스 간 통신에 사용되는 인터넷 소켓, 유닉스 도메인 소켓을 위한 API.

* Java에서의 소켓 활용
  * 데이터가 전송되는 통로라 할 수 있는 스트림(Stream)을 이용하여 프로세스 간 통신 수행.
    * InputStream/OutputStream: 데이터를 바이트 단위로 읽어들이고 기록할 때 활용.
    * Reader/Writer: 데이터를 문자열 단위로 읽어들이고 기록할 때 활용.
      * InputStreamReader: 바이트 단위 데이터를 문자 단위로 변환하여 읽을 때 사용.
      * OutputStreamWriter: 문자 단위의 데이터를 바이트 단위로 변환하여 기록할 때 사용.

* Java try-with-resources
  * 리소스를 자동으로 해제하도록 제공해주는 구문.
  * Java 1.7부터 지원되는 기능.
  * 해당 리소스가 AutoCloseable을 구현한 경우, close()를 명시적으로 호출하지 않아도 
  try 블록에서 오픈된 리소스는 정상적인 경우나 예외가 발생한 경우 모두 자동으로 close()가 호출됨.

* Java text blocks
  * 큰따옴표 세개(""")를 이용하여 여러 줄 문자열 초기화를 가능하게 해주는 문법.
  * Java 15부터 정식으로 지원됨.

* URI, URL, URN
  * URI(Uniform Resource Identifier, 통합 자원 식별자): 인터넷의 자원을 식별할 수 있는 문자열을 의미함. 하위 개념으로 URL, URN이 있음.
    * URL(Uniform Resource Locator): 네트워크 상에서 웹 페이지, 이미지, 동영상 등의 파일이 위치한 경로(Path)를 나타냄.
    * URN(Uniform Resource Name): URI의 표준 포맷 중 하나로, 이름으로 리소스를 특정함.
  * 이름만으로는 리소스에 대한 접근이 어렵기 때문에, 요즘에는 URN을 거의 사용하지 않는다.
  * 이에 따라 URL을 URI라 통칭하기도 하지만, 엄밀히 따지면 차이가 있다.
    * URI 형식은 아래와 같다. 여기서 scheme://host:port/path까지 URL이라 볼 수 있다.
      * 서로 다른 쿼리 파라미터를 가지고 요청이 들어오는 경우, 같은 URL이 포함된 서로 다른 URI라 볼 수 있다.
    * <pre class="language-ini"><code class="lang-ini">scheme://host:port/path?query#fragment</code></pre>

<figure><img src="./images/uri-url-urn.png" alt=""></figure>

* 호스트(Host)
  * 네트워크 상에 연결되어 통신 가능한 장치를 의미. 식별 가능한 하나 이상의 IP 주소를 가지고 있음.
  * IP 주소(IP Address)
    * 호스트 간 통신을 위해 네트워크 상에서 사용되는 고유 주소.
    * 호스트 별로 고정 IP를 지정하여 이용하거나 DHCP(Dynamic Host Configuration Protocol)를 이용하여 동적으로 할당받아 이용할 수 있음.
    * IPv4 주소
      * 현재(2023년) 기준 일반적으로 사용하는 IP 주소로써, 총 32비트로 구성되어 있음.
      * .(dot)을 이용하여 8비트씩 총 네개로 구분하여 나타냄.
        * xxx.xxx.xxx.xxx 형태이며, 각각의 자리는 0부터 255(2^8 - 1)의 숫자로 구성되어 있음.
        * 최대 약 42억(2^32)개의 주소를 부여할 수 있으며, 일부 주소들은 특수한 목적으로 사용하고 있음.
        * 특수 목적으로 사용되는 IPv4 예시
          * localhost: 127.0.0.1
          * 사설 망에서 사용하는 IP: 10.0.0.0/8(10.0.0.0 ~ 10.255.255.255), 172.16.0.0/12(172.16.0.0 ~ 172.31.255.255), 192.168.0.0 ~ 192.168.255.255(192.168.0.0/16)
    * IPv6 주소
      * 32비트로 구성된 IPv4 주소가 고갈되는 문제를 해결하기 위해 나온 총 128비트 크기의 IP 주소.
      * :(colon)을 이용하여 16비트씩 총 여덟개로 구분하여 나타냄.
        * xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx:xxxx 형태이며, 각각의 자리는 0000부터 ffff까지의 16진수로 구성되어 있음.
    * 도메인(Domain)
      * IP 주소를 사람이 인지하기 쉽게 문자열 형태로 바꾼 주소.
      * 영어 소문자, 숫자, -(hyphen)의 조합으로만 표현되어야 하며, -(hyphen)으로 시작하거나 끝날 수 없음.
    * DNS(Domain Name System)
      * 사람이 인지할 수 있는 도메인을 네트워크상의 호스트가 인지할 수 있는 IP 주소로 상호 변환하는 시스템.
      * DDNS(Dynamic DNS)
        * DNS에 매칭된 IP 주소가 변경되어도 해당 도메인으로 접속 가능하도록 제공해주는 시스템.
  
* 포트(Port)
  * IP 내에서 프로세스 간 통신 구분을 위해 사용하는 번호. 0부터 65535(2^32 - 1)까지 할당 가능함.
  * 잘 알려진 포트(Well-known Port)
    * 0 ~ 1023까지의 포트이며, IANA(Internet Assigned Numbers Authority, 인터넷 할당번호 관리기관)에 의해 통제되어 관리가 이루어짐.
      * ex. SSH: 22/TCP, HTTP: 80/TCP, HTTPS: 443/TCP, syslog: 514/TCP 등등.
  * 등록된 포트(Registered Port)
    * 1024 ~ 49151까지의 포트이며, IANA에서 할당하지만 통제되지는 않음.
    * 임의의 프로그램에서 여러 통신을 위해 등록되며, 통제되지 않기 때문에 다른 용도로 사용 가능함.
      * ex. MySQL: 3306/TCP, Zookeeper: 2181/TCP, ElasticSearch: 9200/TCP, Kibana: 5601/TCP, Grafana: 3000/TCP 등등.
  * 동적 포트(Dynamix Port)
    * 49152 ~ 65535까지의 포트이며, 임시 포트이기 때문에 중복 사용이 없다는 전제하에 아무 프로세스에서나 사용 가능함.

* 참고
  * https://stackoverflow.com/questions/176264/what-is-the-difference-between-a-uri-a-url-and-a-urn
