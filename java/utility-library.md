# 유틸리티 라이브러리

### 유틸리티 클래스(Utility Class)
- 별도의 상태를 가지지 않고 정적(static) 메서드만 제공하는 클래스
- 인스턴스를 만들 필요가 없으므로, 보통 생성자를 private으로 선언하여 생성을 막아둠
- 여러 곳에서 반복되는 처리를 한곳에 모아둠으로써 중복을 줄이고 일관성을 확보할 수 있음

<br>

### Apache Commons Lang3
- Java 표준 라이브러리에서 부족한 기능을 보완해주는 유틸리티 라이브러리
- org.apache.commons.lang3 패키지를 사용하며, Java 생태계에서 가장 널리 쓰이는 유틸리티 라이브러리 중 하나임
- 다른 라이브러리의 전이 의존성(Transitive Dependency)으로 딸려오는 경우가 많지만, 직접 사용한다면 명시적으로 의존성을 선언해줘야 함
```
build.gradle

dependencies {
    implementation 'org.apache.commons:commons-lang3:3.20.0'
}
```

<br>

### StringUtils
- 문자열 처리 시 사용하는 유틸리티 클래스이며, Commons Lang3에서 가장 많이 사용하는 클래스라 할 수 있음
- 핵심은 null 안전성. 인자로 null이 들어와도 예외를 발생시키지 않고 정해진 값을 반환함
  - String이 제공하는 메서드는 인스턴스 메서드이므로, 대상이 null이면 NullPointerException이 발생함
  - 정적 메서드로 null을 처리해주기 때문에, null 검사와 빈 문자열 검사를 한 줄로 줄일 수 있음
```
str.isBlank()                 // str이 null이면 NullPointerException 발생
StringUtils.isBlank(str)      // str이 null이면 true 반환
```
- 자주 사용하는 메서드
  - isEmpty / isNotEmpty: 문자열이 null이거나 길이가 0인지 판단함. 공백 문자열(" ")은 empty로 보지 않음
  - isBlank / isNotBlank: isEmpty의 조건에 더해, 공백 문자로만 이루어진 경우까지 판단함
  - defaultString / defaultIfBlank: null이거나 비어있는 경우 지정한 기본값을 반환함
  - trimToEmpty / trimToNull: 앞뒤 공백을 제거하며, 결과가 비어있는 경우 각각 빈 문자열과 null을 반환함
  - equals / equalsIgnoreCase: null 안전 비교를 수행함
  - join / split: 구분자를 이용하여 여러 문자열을 합치거나 하나의 문자열을 나눔
  - leftPad / rightPad: 지정한 길이가 되도록 특정 문자로 앞뒤를 채움
  - capitalize / uncapitalize: 첫 글자의 대소문자를 변환함

<br>

### ObjectUtils
- 객체 전반을 다룰 때 사용하는 유틸리티 클래스이며, StringUtils에 비해 사용 빈도는 낮은 편
- 자주 사용하는 메서드
  - defaultIfNull: 대상이 null인 경우 지정한 기본값을 반환함
  - firstNonNull: 전달받은 값들 중 null이 아닌 첫번째 값을 반환함
  - isEmpty / isNotEmpty: null 여부뿐만 아니라 빈 문자열, 빈 배열, 빈 컬렉션까지 함께 판단함
  - allNotNull / anyNull: 전달받은 여러 값의 null 여부를 한번에 판단함
  - compare: null 안전 비교를 수행함

<br>

### 표준 라이브러리와 Spring이 제공하는 대체 기능
- Java Version이 올라가고 Spring이 자체 유틸리티를 제공하면서, Commons Lang3가 담당하던 영역 상당수가 대체됨
- Java 표준 라이브러리
  - String.join(): 구분자를 이용한 문자열 결합(Java 8)
  - Objects.equals(), Objects.hash(): null 안전 비교 및 해시 생성
  - Objects.requireNonNullElse(): null인 경우 기본값 반환(Java 9)
  - String.isBlank(), String.strip(), String.repeat(): 공백 판단 및 처리(Java 11)
  - Optional: null 처리 흐름 자체를 타입으로 표현
- Spring
  - org.springframework.util.StringUtils: hasText(), hasLength() 등을 제공함
  - org.springframework.util.ObjectUtils: isEmpty() 등을 제공함
  - org.springframework.util.CollectionUtils: 컬렉션 관련 처리를 제공함
- 신규 구현 시에는 표준 라이브러리와 Spring이 제공하는 기능을 먼저 확인하고, 그것으로 해결되지 않는 null 안전 처리에 Commons Lang3를 사용하는 것이 좋음
  - 레거시가 존재하는 프로젝트에서는 여전히 사용 빈도가 높으므로, 코드를 읽기 위해서라도 알아둘 필요가 있음

<br>

### 이름 충돌 주의
- StringUtils와 ObjectUtils는 Apache Commons Lang3와 Spring 양쪽에 같은 이름으로 존재하며, 같은 이름의 메서드라도 동작이 서로 다름
- IDE의 자동 Import 기능에 의존하는 경우 의도하지 않은 쪽이 들어올 수 있으므로, import 구문을 반드시 확인해야 함
```
org.apache.commons.lang3.StringUtils.isEmpty(" ")   // false. 공백 문자열은 empty로 보지 않음
org.apache.commons.lang3.StringUtils.isBlank(" ")   // true

org.springframework.util.StringUtils.hasLength(" ") // true. 길이만 판단함
org.springframework.util.StringUtils.hasText(" ")   // false. 공백 문자열은 텍스트가 없는 것으로 봄
```
- org.springframework.util.StringUtils의 isEmpty(Object)는 Spring 5.3부터 Deprecated 처리되었으므로, hasText()나 hasLength() 또는 ObjectUtils.isEmpty(Object)를 사용해야 함

<br>

#### 참고
- Apache Commons Lang <StringUtils> - https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html
- Apache Commons Lang <ObjectUtils> - https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html

#### 배워가는 것들
- 유틸리티 라이브러리를 쓰는 이유가 단순한 편의성이 아니라 null 안전성에 있다는 것을 정리할 수 있었다. 같은 기능이 표준 라이브러리에 있더라도 null 처리 방식이 다르다는 점을 기억해야겠다.
- 표준 라이브러리와 Spring이 제공하는 기능이 늘어나면서 대체 가능한 영역이 많아졌다는 것을 알게 되었다. 습관적으로 의존성을 추가하기 전에 이미 쓸 수 있는 것이 있는지 먼저 확인해야 할 것이다.
- 같은 이름의 클래스가 여러 라이브러리에 존재할 수 있다는 점이 인상적이었다. 자동 Import에 의존하다 보면 동작이 다른 쪽을 쓰게 될 수 있으므로, 이름이 겹치는 클래스는 특히 주의해서 확인해야 한다.
