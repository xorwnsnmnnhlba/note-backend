# DTO

* DTO(Data Transfer Object)
  * 프로세스 사이에서 도메인 모델을 대신하여 데이터를 전송하는 객체를 의미함.
  * 특정 도메인 모델 데이터를 보낼 때 불필요한 데이터를 피하여 전송하거나, 여러 도메인 모델의 속성들을 하나의 객체로 전송해야 할 때 사용함.

<br>

* 프로세스 간 통신(IPC, Inter-Process Communication)
  * 실행 중인 프로세스 간 데이터를 주고받는 행위를 의미함.
  * 하나의 호스트에서 진행하는 서로 다른 프로세스 간 통신뿐만 아니라, 서로 다른 호스트 간 프로세스와의 통신도 포함됨.
  * Backend와 Frontend 간 통신을 위해 필수적으로 IPC를 수행해야 함.
  * IPC를 수행할 때 사용하는 기술
    * 파일(File) - 가장 기본적인 접근방법이지만, 여러개의 프로세스 혹은 쓰레드에서 동시에 접근할 때 문제가 발생할 수 있기 때문에 원격 환경에서 사용하기는 어려움.
    * 소켓(Socket) - Network를 통해 데이터를 스트림으로 변환하여 전송함. 원격 환경에서도 사용 가능.
      * 정해진 규칙에 따라 동작하는 HTTP 같은 프로토콜을 활용하여 용이하게 전송 가능함.

<br>

* 무기력한 도메인 모델(Anemic Domain Model)
  * 속성(필드)들을 단순히 나열만 하고, 그에 따른 행위(메서드)를 올바르게 설계 및 구현하지 않은 도메인 모델.
  * 안티 패턴인 이유
    * 속성에 대한 행위 구현을 제대로 하지 못하면서, 그에 따른 활용도가 현저하게 제한됨.
    * 유효성 검사나 비즈니스적인 의존성 처리 등등을 제대로 수행하지 못함으로써, 도메인 모델의 응집도를 떨어뜨림.

<br>

* 자바빈즈(JavaBeans)
  * Java에서 작성된 소프트웨어 컴포넌트.
  * 구성요건
    * Java에서 속성값을 나타내는 필드와 속성값 저장 및 불러오는 동작을 수행하는 Getter/Setter 메서드를 갖춰야 함.
    * 필드는 private으로 선언되어야 하며, Getter/Setter 메서드는 public으로 선언되어야 함.
    * 직렬화에 필요한 Serializable 인터페이스를 구현해야 함.
    * 인자가 없는 기본 생성자를 가지고있어야 함.

<br>

* EJB(Enterprise JavaBeans)
  * 기업 환경에서 주로 사용하는 Java EE(Enterprise Edition)를 위한 서버사이드 컴포넌트.

<br>

* Java record
  * Java 16부터 정식으로 추가된 클래스로써, 변하지 않는(immutable) 데이터 인스턴스를 생성할 때 유용하게 사용할 수 있음.
  * DTO 구현 시, 자주 사용하는 Getter, equals, hashCode, toString 등등의 메서드들에 대한 번거로운 구현을 줄일 수 있게 됨.
  * 클래스 변수 선언 가능하며, 필요에 따라 메서드 오버라이딩도 가능함.
  * 매개변수로 넣은 필드들이 모두 변하지 않는 final로 생성되기 때문에, Setter는 제공하지 않음.
  * Getter는 getXXX() 형식이 아니라, 필드명과 유사하게 생성됨.
  * 선언한 필드에 대한 제약조건을 걸어서 예외처리가 가능하도록 설정할 수 있음.

```
public record Account(String email, Integer birthYear) {
    public Account {
        if ( birthYear < 0 ) {
            throw new CustomException();
        }
    }
}
```

<br>

* DAO(Data Access Object)
  * 영속성을 가진 모듈인 DB에 접근하여 데이터를 처리하는 객체를 의미함.
  * JPA에서는 DB에 직접 접근하는 Repository 인스턴스들을 DAO라 볼 수 있음.

<br>

* ORM(Object-Relational Mapping)
  * 객체와 관계형 데이터베이스(RDBMS)를 매핑하는 것.
  * Java 애플리케이션의 클래스와 SQL DB 테이블 사이의 매핑 정보를 기술한 메타데이터를 사용하여,  
    Java 애플리케이션의 인스턴스를 SQL DB 테이블에 자동으로 영속화해주는 기술이라 할 수 있음.  
  * 객체를 DB에 저장/수정/삭제할 때 SQL insert/update/delete문을 직접 작성하지 않고,  
    Java에서 인스턴스를 List, Map 등등에 저장하듯이 ORM 프레임워크에 저장하여 진행할 수 있음.
  * RDBMS를 이용하더라도 객체 모델링에 집중할 수 있어서 효율적인 객체지향 개발이 가능해짐.
  * 특정 벤더에 대한 의존성(Dependency)이 낮음.
  * 위와 같은 장점 때문에 학습곡선(Learning Curve)이 다소 높더라도 많은 개발자들이 애용하고 있음.

<br>

* ORM 패러다임 불일치
  * 객체를 RDBMS에 매핑시키면서 발생하는 문제.
  * 객체와 RDBMS는 지향하는 목적이 서로 다르므로, 기능과 표현방법에 있어 차이를 보임.
  * 객체 모델링을 정교하게 진행할수록 문제가 커질 수 있음.
  * Java에서는 이러한 불일치를 해결하기 위해 JPA(Java Persistence API)가 등장함.

<br>

* 참고
  * 인프런 <스프링 Data JPA> - 백기선
  * <자바 ORM 표준 JPA 프로그래밍> - 김영한
