

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
![image](https://user-images.githubusercontent.com/55453184/164879633-79713ba8-2cd4-462d-8151-64281962da23.png)
