# Collection Pattern

### Collection Pattern
* 특정 리소스(ex. 회원정보, 게시물, 상품 등등)들을 각각의 그룹으로 묶어서 표현할 때 사용.
* 그룹명을 표시할 때, 복수형을 사용하기 때문에 URI 상에서 하나의 리소스를 지정할 때도 복수형으로 써야 함.
* 특정 리소스에 의존적인 리소스(ex. 임의의 게시물에 해당하는 댓글)라면, 디렉터리처럼 구성 가능함.
* 예시
  * /posts -> 모든 게시물을 하나의 URI로 표현(일반적으로 복수형으로 사용).
  * /posts/{postId} -> postId에 해당하는 게시물을 표현.
  * /posts/{postId}/comments -> postId에 해당하는 게시물에 대한 댓글 목록 표현.
    * /comments?postId={postId} 형태로 사용해도 됨.
    얼핏 봤을 때 전하는 의미는 다르지만, 수행이 이루어지는 로직 차이가 없으므로 동일하게 봐도 무방함.
  * /posts/{postId}/comments/{commentId} -> postId에 해당하는 게시물에 있는 댓글 중 commentId에 해당하는 댓글 표현.
    * 인기 댓글, 댓글 삭제 등등의 댓글에 대한 간단한 작업 요청이라면 /comments/{commentId}로 구성하는게 좋을 수 있음.
  * /posts/{postId}/edit -> postId에 해당하는 게시물 수정시에 사용.
    * /edit?postId={postId} 형태로 사용해도 되지만, 게시물에 해당하는 댓글 수정과 구분해서 사용할 때 혼동을 줄 수 있음.
* 세션과 같이 특정 리소스가 그룹이 아닌 경우, 단수형으로 표현할 수 있음.
  * 예시
    * /session -> 세션은 오직 하나만 유지되므로, 단수형으로 표현함.

<br>

#### 참고
  * https://learn.microsoft.com/ko-kr/azure/architecture/best-practices/api-design#organize-the-api-design-around-resources
