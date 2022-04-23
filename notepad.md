

## 프로토콜, 소켓의 타입

#### 프로토콜 체계 (Protocol Family)
  1. PF_INET: IPv4
  2. PF_INET6: IPv6

#### 소켓의 타입
  1. 연결지향형 (SOCK_STREAM, IPPROTO_TCP)
     * 데이터가 소멸되지 않음
     * 전송 순서대로 데이터 수신
     * 데이터의 경계가 존재하지 않음 (한 번 전송한 데이터를 여러 번 쪼개어 수신할 수 있음)
     
  2. 비 연결지향형 (SOCK_DGRAM, IPPROTO_UDP)
     * 빠른 전송속도를 지향
     * 데이터 손실 우려
     * 데이터의 경계가 존재함 (두 번 전송한 데이터는 두 번 수신함)
     * 한번에 전송 가능한 데이터의 크기가 제한됨


## IP, Port의 표현

#### IPv4 기반 주소 정보를 저장하는 구조체
```
  struct sockaddr_in{
    sa_family_t    sin_family;    //Address Family (AF_INET, AF_INET6)
    uint16_t       sin_port;      //16bit TCP, UDP port
    struct in_addr sin_addr;      //32bit IP
    char           sin_zero[8];   //사용 안함. 반드시 memset()을 사용하여 해당 구조체 초기화
  }
```
```
  struct in_addr{
    in_addr_t     s_addr;         //32bit IPv4
  }
```

#### 바이트 순서
  1. 빅 엔디안(Big Endian): 상위 바이트의 값을 작은 번지수에 저장 → 네트워크 바이트 순서(Network Byte Order)
  2. 리틀 엔디안(Little Endian): 상위 바이트의 값을 큰 번지수에 저장 → 호스트 바이트 순서(Host Byte Order)

#### 바이트 순서 변환
  1. 16bit, 32bit의 호스트 ↔ 네트워크 바이트 순서 변환
```
#include <arpa/inet.h>

unsigned short htons(unsigned short);
unsigned short ntohs(unsigned short);
unsigned long htonl(unsigned long);
unsigned long ntohl(unsigned long);
```

  2. IP의 문자열 정보 ↔ 네트워크 바이트 순서의 정수 변환
```
#include <arpa/inet.h>

in_addr_t inet_addr(const char* string);
int inet_aton(const char* string, struct in_addr* addr);
char* inet_ntoa(struct in_addr* addr);
```

#### 인터넷 주소의 초기화
```
struct sockaddr_in addr;
memset(&addr, 0, sizeof(addr)); //sin_zero[8]을 0으로 초기화
addr.sin_family = AF_INET;
addr.sin_addr.s_addr = inet_addr(char_ip);
//addr.sin_addr.s_addr = htonl(INADDR_ANY); //자동으로 IP주소 할당
addr.sin_port = htons(atoi(char_port));
```

#### 소켓 함수 호출순서
  1. 서버 소켓
```
#include <sys/socket.h>
#include <unistd.h>

int socket(int domain, int type, int protocol);
int bind(int sockfd, struct sockaddr* myaddr, socklen_t addrlen);
int listen(int sock, int backlog);
int accept(int sock, struct sockaddr* addr, socklen_t* addrlen);
//ssize_t write(int fd, const void* buf, size_t nbytes);
//ssize_t read(int fd, void* buf, size-t nbytes);
int close(int fd);
```

  2. 클라이언트 소켓
```
#include <sys/socket.h>
#include <unistd.h>

int socket(int domain, int type, int protocol);
int connect(int sock, struct sockaddr* servaddr, socklen_t addrlen);
//ssize_t write(int fd, const void* buf, size_t nbytes);
//ssize_t read(int fd, void* buf, size-t nbytes);
int close(int fd);
```


## TCP/IP

#### TCP/IP Protocol Stack
![image](https://user-images.githubusercontent.com/55453184/164879633-79713ba8-2cd4-462d-8151-64281962da23.png)
  * Link 계층: 물리적 영역의 표준(WAN, MAN, LAN)
  * IP 계층: 비 연결지향형 프로토콜. 목적지로 데이터를 전송하기 위해 경로를 탐색하며, 데이터 손실의 우려가 있다.
  * TCP/UDP 계층: 대량의 패킷을 송수신하기 위한 프로토콜. IP가 탐색한 경로를 기준으로 데이터의 송수신을 담당한다.
  * Application 계층: 프로그래머가 정의한 서버-클라이언트 간의 프로토콜

#### TCP 이론
  1. 연결 성립 (Three-way handshaking)
  ![image](https://user-images.githubusercontent.com/55453184/164881756-49b93f86-dede-4758-abcb-e3f26ad8d122.png)
    1. [SYN] SEQ: 1000, ACK: -   
    2. [SYN + ACK] SEQ: 2000, ACK: 1001 (ACK = 1의 SEQ + 1 = 상대가 다음에 전송할 SEQ)   
    3. [ACK] SEQ: 1001, ACK: 2001 (SEQ = 이전에 보낸 SEQ + 1, ACK = 2의 SEQ + 1)   
    
  2. 데이터 송수신
    1. ACK = SEQ + 수신한 바이트 크기
    2. 일정 시간 내에 대상으로부터 패킷을 받지 못하면 Time out이 발생하고 이전에 보냈던 패킷을 재전송한다.
    ![image](https://user-images.githubusercontent.com/55453184/164882684-409de9a2-2d15-4dc1-9b0f-ad7560895113.png)

