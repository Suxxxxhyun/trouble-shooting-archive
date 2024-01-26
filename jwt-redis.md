## petree의 jwt흐름도

<img src="../petree/image/jwt-stream.PNG">

## Refresh Token을 Redis에 저장한 이유?
- Redis는 key-value쌍으로 데이터를 관리할 수 있는 data storage이다. DB가 아닌 data storage라고 표현하지 않은 이유는 기본적으로 Redis는 in-memory로 데이터를 관리하므로, 저장된 데이터가 영속적이지 않기 때문이다.
  - 이때 in-memory란?
  <img src="../petree/image/disk-inmemory.PNG">
    - DISK DBMS의 경우 
      - 디스크에 저장된 내용을 메인 메모리로 로딩을 해야하며 실행을 위해서는 cpu로 해당 데이터를 재전송해야한다. 하지만 메모리가격보다 저렴하다.
    - in-memory DBMS의 경우
      - 메인 메모리에서 바로 cpu로 데이터를 전송만 하면 되므로 구조상 간단하다. 하지만 비용이 비싸다.
- 이러한 Redis에 Refresh Token을 저장하기에 적합하다고 생각한 이유는 다음과 같다.
  1) **RT를 RDB등에 저장하면, 스케줄러등을 이용해 주기적으로 만료된 토큰을 만료처리하거나 제거해야한다.** 하지만, **Redis는 기본적으로 데이터의 유효시간을 지정**할 수 있다.
  2) 물론 jwt와 같은 클레임 기반 토큰을 사용하면, RT를 서버에 저장할 필요가 없지만, 사용자 강제 로그아웃 기능, **유저 차단, 토큰 탈취시 대응을 해야한다는 가정**으로 Redis에 RT를 저장하도록 구현하였다.


## Petree서비스의 자세한 구현내용은 다음과 같다.

- 사용자가 로그인에 성공하면, (key:유저아이디, value:RT)를 기반으로 Redis에 저장하도록 하였다. 이후, 사용자는 유효한 AT를 기반으로 API를 요청하도록 하였다.
- 사용자가 만료된 AT를 기반으로 API를 요청하는 경우, RT를 이용하여 AT를 새롭게 발급하도록 구현하였다.
- 사용자가 로그아웃에 성공하면, AT를 Redis의 블랙리스트에 저장하도록 구현하였다.
  - 혹여나 클라이언트가 블랙리스트에 있는 토큰을 요청한다면 인증실패하도록 구현하였다.

- 참조블로그
  - https://blog.naver.com/gkenq/10183400845
  - https://velog.io/@rnqhstlr2297/JWT-Redis%EB%A5%BC-%EC%9D%B4%EC%9A%A9%ED%95%9C-%EB%A1%9C%EA%B7%B8%EC%95%84%EC%9B%83