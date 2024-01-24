## XSS(Cross Site Scripting)
- 악의적인 사용자가 공격하려는 사이트에 스크립트를 넣는 기법으로
- XSS를 통해 사용자의 쿠키나 세션등의 민감한 정보를 탈취할 수 있다.

<img src="../image/Stored XSS.PNG">

### Stored XSS

- 악의적인 사용자가 어떤 사이트에서 게시글을 작성할 때, 해당 게시글을 조회한 회원의 쿠키 정보를 탈취하는 악성 스크립트를 작성해서 저장했다고 가정하자.
- 이후, 사용자가, 악성 스크립트가 저장되어있는 게시글을 클릭하면, 게시글이 열리면서 스크립트 코드가 실행이 될 것이다.
- 이렇게 되면, 스크립트 코드를 통해서 사용자의 쿠키정보를 악의적인 사용자한테 전달할 것이다.
- 악의적인 사용자는 해당 쿠키를 통해서 서버의 사용자인척하여 악의적인 행동을 할 것이다.
- 만약 탈취한 쿠키가 관리자 회원의 쿠키였다면, 더욱 심각한 상황이 발생할 것이다.

### XSS를 방어하는 방법
- 오픈소스 "lucy-xss-servlet-filter"를 이용하여 서버로 들어오는 파라미터에 대해서 필터링을 하는 역할을 한다.
- 예를 들어, 스크립트 코드를 작성할 때 사용하는 '<'문자는 '$lt'로 변환을 해준다.
- 장점 ) 이렇게 되면, 사용자가 게시글을 열어도 서버에 저장된 문자가 치환되어 저장되기 때문에 JS코드가 실행되지 않을 것이다.
- 단점 ) "lucy-xss-servlet-filter"는 application/x-www-form-urlencoded"에 대해서만 적용되고 http request body에 담긴 json데이터에 대해 처리해주지 못한다.
- 따라서, json데이터를 필터 처리해주는 MappingJackson2HttpMessageConverter를 Bean으로 등록한다.
- 참조블로그
  - https://jojoldu.tistory.com/470
- 오픈소스 "lucy-xss-servlet-filter"
  - https://github.com/naver/lucy-xss-servlet-filter
