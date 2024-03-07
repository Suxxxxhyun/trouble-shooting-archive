## Oauth개념 및 동작방식 이해하기
- 웹 서핑을 하다보면 google과 kakao와 같이 외부 소셜 계정을 기반으로 간편히 회원가입 및 로그인할 수 있는 웹 어플리케이션을 쉽게 찾아볼 수 있다.
- 예를 들어, google로 로그인하면 API를 통해 연동된 계정 정보를 가져와 로그인이 간편하게 진행된다.
- 이때 사용되는 프로토콜이 OAuth이다. 


> *OAuth는 인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여할 수 있는 공통적인 수단으로서 사용되는, 접근 위임을 위한 개방형 표준이다. (위키백과)*

- '원티드라 서비스'는 사용자 인증을 위해 kakao, naver, facebook, google등의 사용자 인증 방식을 사용한다고 가정하자.
- 이때, Oauth를 바탕으로 제 3자 서비스(원티드라)는 외부 서비스(kakao, naver, facebook, google등)의 특정 자원을 접근 및 사용할 수 있는 권한을 인가받게 된다.

- 펫트리서비스에서 사용한 것을 빗대어 보면, **펫트리 서비스가 외부 서비스(kakao)의 특정 자원을 접근 및 사용할 수 있는 권한을 인가받게 된다.**

## Oauth참여자
Oauth동작에 관여하는 참여자는 크게 세가지로 구분할 수 있다.

- Resource Server
  - Client가 제어하고자 하는 자원을 보유하고 있는 서버(kakao, google, naver, facebook)
- Resource Owner 
  - 자원의 소유자 (로그인하려는 실제 사용자)
  - Client가 제공하는 서비스를 통해 로그인하는 실제 유저가 이에 속한다.
- **Client(외부 어플리케이션)**
  - Resource Server에 접속해서 정보를 가져오고자 하는 클라이언트(웹 어플리케이션) (ex: 원티드, 펫트리)

## Oauth Flow
- kakao계정을 바탕으로 웹 어플리케이션에서 로그인하고, kakao에서 제공하는 여러 API기능을 외부 어플리케이션에서 사용해보도록 하자.

1) Client등록

    - Client가 Resource Server를 이용하기 위해서는 자신의 서비스를 등록함으로써 사전 승인을 받아야한다. 
    ex) kakao developer에서 petree웹 어플리케이션을 등록하였다. 이 과정이 이에 해당한다.
    
    - 등록 절차를 통해 세 가지 정보를 부여받게 된다.
      - ClientID : 클라이언트 웹 어플리케이션을 구별할 수 있는 식별자
      - Client Secret : Client ID에 대한 비밀키
      - Authorized Redirect URL : Authorization Code를 전달받을 주소
    - Google 등 외부 서비스를 통해 인증을 마치면, 클라이언트를 명시된 주소로 리다이렉트 시키는데, 이때 QueryString으로 특별한 Code가 전달된다.
    - 클라이언트는 인가Code와 Client ID, Client Secret을 Resource Server에 보내,
    - Resource Server의 자원을 사용할 수 있는 AT를 발급받는다.
      - 등록되지 않은 리다이렉트 URL을 사용하는 경우, Resource Server가 인증을 거부한다.
2) Resource Owner의 승인
   - Resource Owner(로그인하려는 실제 사용자 = 자원의 소유자)는 Client의 웹 어플리케이션을 이용하다가, 해당 주소로 연결되는 소셜 로그인 버튼을 클릭한다.
   - Resource Owner는 Resource Server에 접속하여 로그인을 수행한다.
   - **로그인이 완료되면 Resource Server는 QueryString으로 넘어온 파라미터들 등을 통해 Client를 검사한다.**
     - 파라미터로 전달된 **Client ID와 동일한 ID값이 존재하는지 확인한다.**
     - 해당 Client ID에 해당하는 **Redirect URL이 파라미터로 전달된 Redirect URL과 같은지 확인한다.**
   - **검증이 마무리되면, Resource Server는 Resource Owner에 다음과 같은 질의를 보낸다.
     - **권한을 Client에게 정말로 부여할거니?**
   - 허용한다면, 최종적으로 Resource Owner가 Resource Server에게 Client의 접근을 승인하게 된다.
3) Resource Server의 승인
   - Resource Owner의 승인이 마무리 되면, 명시된 Redirect URL로 Client를 리다이렉트 시킨다.
   - 이때, Resource Server는 Client가 자신의 자원을 사용할 수 있는 AT를 발급하기 전에, 임시 암호인 Authorization Code를 함께 발급한다.
   - Client는 Client ID와 Secret, Code를 Resource Owner를 거치지 않고 Resource Server에 직접 전달한다.
   - Resource Server는 정보를 검사한 후, 유효한 요청이라면 AT를 발급하게 된다.
   - Client는 해당 토큰을 서버에 저장해두고, Resource Server의 자원을 사용하기 위한 API호출시, 해당 토큰을 헤더에 담아 보낸다.
4) API호출
   - 이후 AT를 헤더에 담아 Kakao API를 호출하면, 해당 계정과 연동된 Resource Serve의 풍부한 자원 및 기능들을
   - 내가 만든 웹 어플리케이션(펫트리)에서 사용할 수 있다.
5) Refresh Token발급
   - AT는 만료기간이 있으며, 만료된 AT로 API를 요청하면 401에러가 발생한다.
   - AT가 만료되어 재발급받을 때마다, 서비스 이용자가 재로그인하는 것은 다소 번거롭다.
   - 보통 Resource Server는 AT를 발급할 때 RT를 함께 발급한다.
   - Client는 AT, RT를 모두 저장해두고, Resource Server의 API를 호출할때 AT를 사용한다.
   - AT가 만료되어 401에러가 발생하면, Client는 보관중이던 RT를 보내 새로운 AT를 발급받는다.

- 참조블로그
    - https://tecoble.techcourse.co.kr/post/2021-07-10-understanding-oauth/