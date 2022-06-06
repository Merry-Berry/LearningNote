### x86-64 Architecture
* 명령어 집합 구조 (Instruction Set Architecture, ISA)  
  CPU가 해석하는 명령어의 집합
* 범용 레지스터
  1. rax: 함수의 반환 값
  2. rbx: 주된 용도 x
  3. rcx: 반복문의 반복 횟수, 연산 시행 횟수
  4. rdx: 주된 용도 x
  5. rsi: 데이터를 옮길 때 원본을 가리키는 포인터
  6. rdi: 데이터를 옮길 때 목적지를 가리키는 포인터
  7. rsp: 사용중인 스택의 위치를 가리키느 포인터
  8. rbp: 스택 바닥을 가리키는 포인터
* 세그먼트 레지스터(cs, ss, ds, es, fs, gs)  
  16비트이고 세그먼트 영역을 가리킬 때 사용. cs, ds, ss는 각각 코드, 데이터, 스택 영역을 가리킨다.
* 명령어 포인터 레지스터(rip)  
  CPU가 다음에 실행할 명령어를 가리키는 포인터
* 플래그 레지스터
  1. CF(Carry Flag): 부호 없는 수의 연산 결과가 비트의 범위를 넘을 경우 설정
  2. ZF(Zero Flag): 연산의 결과가 0일 경우 설정
  3. SF(Sign Flag): 연산의 결과가 음수일 경우 설정
  4. OF(Overflow Flag): 부호 있는 수의 연산 결과가 비트 범위를 넘을 경우 설정

### 스택과 프로시저
#### 스택 push/pop
* push val
  1. rsp -= 8
  2. mov [rsp], val
* pop reg
  1. rsp += 8
  2. reg = [rsp - 8]

#### 프로시저 호출
* call addr: addr 프로시저 호출
  1. push return_address :return_address는 복귀 후 실행할 다음 명령어 위치
  2. jmp addr
* leave: 스택 프레임 정리
  1. mov rsp, rbp
  2. pop rbp
* ret: return address로 반환
  1. pop rip

#### 프로시저 호출, 반환 시 스택 관리
1. call로 프로시저 호출하면서, push return_addr
2. 기존 스택 프레임을 저장하기 위해, push rbp
3. 새로운 스택 프레임을 만들기 위해, mov rbp, rsp
4. 새로운 스택 프레임의 공간을 확장하기 위해, sub rsp, 0x(size)
5. 새로운 공간에 지역변수 할당 후 로직 동작
6. 스택 프레임의 시작 부분으로 돌아가기 위해, leave (mov rsp, rbp ; pop rbp)
7. 다음 명령어로 복귀하기 위해, ret (pop rip)

### 시스템 콜
유저 모드에서 커널 모드의 시스템에게 특정 동작을 요청하기 위해 사용
#### syscall
* 요청: rax
* 인자 순서: rdi->rsi->rdx->rcx>r8->r9->stack
* 호출: syscall
* 반환: rax
