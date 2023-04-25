# File Upload

### HTTP multipart/form-data
* HTML form을 통해 파일을 전송할 때 사용하는 옵션값. form 태그 내에서 enctype 옵션으로 지정할 수 있음.
  * 위의 form을 통해 POST 방식으로 이루어진 요청의 경우, Header에 있는 Content-type 값으로 multipart/form-data가 전송됨.
* Spring Web MVC를 이용하여 파일을 업로드할 때, DispatcherServlet의 MultipartResolver 인터페이스(구현체: StandardServletMultipartResolver)가 해당 역할을 수행함.
  * HttpServletRequest를 MultipartHttpServletRequest로 변환해주어 요청이 담고 있는 File을 꺼낼 수 있는 API를 제공함.

<br>

### @ModelAttribute
* 메서드 파라미터를 바인딩(Binding)할 때 사용하는 애노테이션.
* @RequestMapping 애노테이션이 달려있는 컨트롤러 클래스와 함께 사용되며, consumes 속성으로 특정 타입의 데이터만 받을 수 있음.
  * JSON 데이터를 받는다면 MediaType.APPLICATION_JSON_VALUE를 사용하며, 파일을 받는다면 MediaType.MULTIPART_FORM_DATA_VALUE를 사용함.
* 여러 곳(URI Path, 요청 매개변수, 세션 등등)에 있는 단순 타입 데이터를 복합 타입 객체로 받아오거나 해당 객체를 새로 만들 때 사용할 수 있으며, 생략 가능함.
* 값을 바인딩할 수 없는 경우, BindException 발생(400 Error).
  * 바인딩 에러를 직접 다루고 싶은 경우, 해당 메서드 파라미터에 BindingResult 인스턴스를 추가하여 다룰 수 있음.
  * 바인딩 이후 검증 작업을 추가로 하고 싶은 경우, @Valid 또는 @Validated 애노테이션을 사용하여 속성에 대한 검증을 수행할 수 있음.

<br>

### MultipartFile, MockMultipartFile
* MultipartFile
  * 파일 업로드 시 사용하는 메서드 아규먼트.
  * record를 이용하여 DTO의 필드로 넣어서 사용 가능.
  * MultipartResolver 빈이 설정되어 있어야 사용할 수 있음(스프링 부트의 경우, 자동으로 설정됨).
  * POST multipart/form-data 요청으로부터 오는 파일을 참조할 수 있으며, List를 이용하여 여러 파일 참조도 가능함.
* MockMultipartFile
  * 파일 업로드 관련 테스트 코드 작성 시 사용하는 클래스.
  * 아래와 같이 이름, 업로드할 파일명, 파일 형식, 파일 스트림을 가지고 인스턴스를 생성한 후, multipart(url) 메서드를 이용하여 테스트 가능.
```
@Test
@DisplayName("POST /images - with file")
void createWithFile() throws Exception {
    String filename = "src/test/resources/files/test.jpg";

    MockMultipartFile file = new MockMultipartFile("image", "test.jpg",
            "image/jpeg", new FileInputStream(filename));

    mockMvc.perform(multipart("/images")
                    .file(file))
            .andExpect(status().isCreated());
}
```

<br>

### Cloudinary
* 클라우드 환경에 이미지와 영상을 업로드할 수 있는 도구.
* 제한된 수준에서 무료로 사용 가능.
* Cloudinary Dashboard에 있는 API ENV를 프로그램 ENV로 지정하여 구성 가능.
* 연동을 위해 build.gradle에 Cloudinary SDK 의존성을 추가해야 함.

<figure><img src="./images/cloudinary-dashboard.png" alt=""></figure>

```
build.gradle

implementation 'com.cloudinary:cloudinary-http44:1.32.2'


application.yml -> 실제 값은 프로그램 ENV로 지정하여 구성.

cloudinary:
  url: ${CLOUDINARY_URL:cloudinary://key:secret@cloud}

```

<br>

#### 참고
* 인프런 <스프링 웹 MVC> - 백기선

<br>

#### 배워가는 것들
* Spring Boot 내부에서 돌아가는 MultipartFile 동작원리.
* Cloudinary에 대한 이해.
* 정적(static) Web 리소스(ex. 이미지, 동영상) 관리에 대한 고찰.
  * 이미지 크기 조정이 필요한 경우 -> Cloudinary
  * 크기 조정이 필요없는 이미지나 짧은 동영상의 경우 -> Nginx
  * 길이가 긴 동영상의 경우 -> 별도의 Streaming 서버 연동 필요.
