# SocketProgramming

## 프로토콜, 소켓의 타입

#### 프로토콜 체계 (Protocol Family)
  1. PF_INET: IPv4
  2. PF_INET6: IPv6

#### 소켓의 타입
  1. 연결지향형 (SOCK_STREAM, IPPROTO_TCP)
     * 데이터가 소멸되지 않음
     * 전송 순서대로 데이터 수신
     * 데이터의 경계가 존재하지 않음 (한 번 전송한 데이터를 여러 번 쪼개어 수신할 수 있음)
     
  3. 비 연결지향형 (SOCK_DGRAM, IPPROTO_UDP)
     * 빠른 전송속도를 지향
     * 데이터 손실 우려
     * 데이터의 경계가 존재함 (두 번 전송한 데이터는 두 번 수신함)
     * 한번에 전송 가능한 데이터의 크기가 제한됨

## IP, Port의 표현
