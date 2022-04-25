#### 2022년 4월 25일 월
* 멀티플렉싱 서버 구현하기
  * fd_set형 변수
    파일 디스크립터들의 정보를 등록하고, 그들의 상태를 비트 단위로 관찰하는 변수
    ![image](https://user-images.githubusercontent.com/55453184/165032393-ad28879c-70b8-40d5-84c9-6cdbfe2a2121.png)
    ```C
    \\fd_set형 변수를 조작하는 메크로 함수
    FD_ZERO(fd_set* fdset); //fd_set형 변수의 모든 비트를 0으로 초기화
    FD_SET(int fd, fd_set* fdset); //fd_set형 변수에 파일 디스크립터 fd 정보를 등록
    FD_CLR(int fd, fd_set* fdset); //fd_set형 변수에 파일 디스크립터 fd 정보를 삭제
    FD_ISSET(int fd, fd_set* fdset); //fd_set형 변수에 파일 디스크립터 fd의 정보가 있으면 양수 반환
    ```
  * select 함수
    ```C
    #include <sys/select.h>
    #include <sys/time.h>
    
    int select(int maxfd, fd_set* readset, fd_set* writeset, fd_set* exceptset, const struct timeval* timeout);
    ```
    1. maxfd: 가장 큰 파일 디스크립터에 1을 더한 값
    2. readset: fd_set형 변수에서 수신된 데이터의 존재 여부를 검사할 때 전달하는 매개변수
    3. writeset: fd_set형 변수에서 데이터 전송 가능 여부를 검사할 때 전달하는 매개변수
    4. exceptset: fd_set형 변수에서 예외사항의 발생 여부를 검사할 때 전달하는 매개변수
    5. timeout: select함수가 검사 과정에서 변하는 파일 디스크립터가 존재할 때까지 기다리는 시간.
                시간이 다 지나면(Time out 시) 반환 값은 0이 되고, 시간 내에 임의의 파일 디스크립터의 상태가 변하면 변한 개수만큼 반환.
                timeout시간을 설정하지 않으면 파일 디스크립터의 상태가 변할 때까지 해당 함수는 Blocking 상태에 놓인다.
