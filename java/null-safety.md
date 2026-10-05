# Null 안전성(JSpecify)

### NullPointerException 문제
- Java에서는 모든 참조 타입 변수에 null이 들어갈 수 있으므로, 어떤 값이 null일 수 있는지 타입만 보고는 알 수 없음
  - 확인하지 않고 사용하면 실행 중에 NullPointerException이 발생함
  - 반대로 불안해서 모든 곳에 null 검사를 넣으면 코드가 지저분해짐
- 해결하기 위한 기존 방법들
  - Optional: 값이 없을 수 있는 반환값을 표현함. 하지만 필드나 매개변수에 쓰는 것은 권장되지 않고, 기존 API 전체에 적용할 수 없음
  - 애노테이션: 라이브러리마다 javax.annotation.Nullable, org.springframework.lang.Nullable, JetBrains의 @Nullable 등 서로 다른 애노테이션을 사용하여 도구마다 해석이 달랐음

<br>

### JSpecify
- Google, JetBrains, Spring 팀 등이 함께 만든 Java의 null 관련 표준 애노테이션으로, 2024년 1.0이 출시됨
- 주요 애노테이션
  - @NullMarked: 선언한 범위(패키지, 클래스, 메서드) 안의 타입은 기본적으로 null이 아님(Non-null)을 의미함
  - @Nullable: null일 수 있는 타입에 붙임
  - @NonNull: @NullMarked 범위 밖에서 null이 아님을 명시할 때 사용함
  - @NullUnmarked: @NullMarked 범위 안에서 일부를 다시 해제함
- package-info.java에 @NullMarked를 선언하고, null이 될 수 있는 곳에만 @Nullable을 붙이는 방식이 일반적임
  - 대부분의 값은 null이 아니므로, 예외적인 곳만 표시하면 되어 애노테이션이 적어짐
```
build.gradle

dependencies {
    implementation 'org.jspecify:jspecify:1.0.0'
}
```
```
src/main/java/com/example/order/package-info.java

@NullMarked
package com.example.order;

import org.jspecify.annotations.NullMarked;
```
```
public class OrderService {

    public Order getOrder(Long id) {                       // null을 반환하지 않음
        return repository.findById(id).orElseThrow();
    }

    public @Nullable Coupon findCoupon(String code) {      // null을 반환할 수 있음
        return couponRepository.findByCode(code);
    }

}
```

<br>

### 타입 사용(Type-Use) 애노테이션
- JSpecify 애노테이션은 타입의 일부로 붙는 Type-Use 애노테이션이므로, 위치에 따라 의미가 달라짐
  - List<@Nullable String>: 요소가 null일 수 있는 List
  - @Nullable List<String>: List 자체가 null일 수 있음
  - String @Nullable []: 배열 자체가 null일 수 있음
  - @Nullable String[]: 배열의 요소가 null일 수 있음
- 중첩 타입은 Map.@Nullable Entry처럼 바깥 타입 이름 뒤에 붙여야 함

<br>

### Spring Framework 7의 Null 안전성
- Spring Framework 7과 Spring Boot 4.x는 전체 코드에 JSpecify 애노테이션을 적용함
  - 이전에 사용하던 org.springframework.lang.Nullable 등은 Deprecated 처리됨
  - Spring Data, Spring Security, Spring AI 등 다른 프로젝트들도 함께 적용함
- Spring API를 호출할 때 반환값이 null일 수 있는지 IDE가 바로 알려주므로, 불필요한 null 검사와 빠뜨린 null 검사를 줄일 수 있음
- Kotlin은 JSpecify 애노테이션을 읽어 Spring API의 반환 타입을 null 가능 타입과 불가능 타입으로 정확히 구분함

<br>

### NullAway로 컴파일 시점에 검사하기
- 애노테이션만으로는 경고에 그치므로, 빌드할 때 검사하여 위반하면 실패하게 만들어야 효과가 있음
- NullAway는 Uber가 만든 Error Prone 플러그인으로, @NullMarked 범위의 코드에서 null 관련 오류를 컴파일 오류로 만듦
  - null일 수 있는 값을 확인 없이 사용하는 경우
  - Non-null 매개변수에 null일 수 있는 값을 넘기는 경우
  - Non-null 반환 타입에서 null을 반환하는 경우
- Gradle에서는 Error Prone 플러그인과 함께 설정함
```
build.gradle

import net.ltgt.gradle.errorprone.CheckSeverity

plugins {
    id 'net.ltgt.errorprone' version '<버전>'
}

dependencies {
    implementation 'org.jspecify:jspecify:1.0.0'
    errorprone 'com.google.errorprone:error_prone_core:<버전>'
    errorprone 'com.uber.nullaway:nullaway:<버전>'
}

tasks.withType(JavaCompile).configureEach {
    options.errorprone {
        check('NullAway', CheckSeverity.ERROR)
        option('NullAway:OnlyNullMarked', 'true')
    }
}
```
- 기존 프로젝트는 패키지 단위로 @NullMarked를 하나씩 붙여가며 점진적으로 적용함

<br>

### Optional과 함께 사용하기
- 값이 없을 수 있는 반환값을 Optional로 표현하는 방식은 여전히 유효함
  - Spring Data의 findById처럼 조회 결과가 없을 수 있다는 것을 호출하는 쪽에 강제할 때 사용함
- 필드, 매개변수, 컬렉션 요소처럼 Optional이 어울리지 않는 곳은 @Nullable로 표현함
- Optional을 반환하는 메서드에서 null을 반환하지 않도록 주의해야 함

<br>

#### 참고
- JSpecify 공식문서 <User Guide> - https://jspecify.dev/docs/user-guide/
- Spring Framework Reference Documentation <Null-safety> - https://docs.spring.io/spring-framework/reference/core/null-safety.html
- GitHub <uber/NullAway> - https://github.com/uber/NullAway
- heise <Spring Framework 7 brings new concept for null safety> - https://heise.de/-11078745

#### 배워가는 것들
- 모든 곳에 null 가능성을 표시하는 대신, 기본을 Non-null로 두고 예외만 @Nullable로 표시하는 방식이 훨씬 현실적이라는 것을 알게 되었다.
- 애노테이션은 경고에 그치므로 NullAway 같은 도구로 빌드를 실패시켜야 실제로 NPE를 막을 수 있다는 점이 중요했다.
- Spring Framework 7이 전체 API에 JSpecify를 적용하면서, Spring API의 반환값이 null일 수 있는지 문서를 찾아보지 않아도 IDE에서 바로 알 수 있게 되었다.
