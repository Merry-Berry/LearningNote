### Stack Buffer Overflow
읽기 함수의 취약점을 통해 stack영역을 침범하여 중요 데이터를 유출, 변조하거나 셸코드를 삽입하여 원하는 동작을 발생시키는 기법

#### Calling Convention
함수 호출 규약은 함수의 호출과 반환의 방법에 대한 약속이다. 함수 호출 시 인자를 넘기는 방법과 호출자로 다시 복귀하는 방법을 정의하였고, 아키텍처마다 다양한 호출 규약이 존재한다.  
컴파일 시 적용되는 호출 규약은 컴파일러에게 결정되어 있다.
1. x86  
  a. cdecl  
  b. fastcall  
  c. thiscall  
2. x86-64  
  a. SYSV ABI (MSVC)  
  b. MS ABI (gcc)  

#### x86 calling convention: cdecl
* caller는 callee에게 인자를 스택으로 마지막 인자부터 첫 번째 인자까지 push하여 전달한다.
* callee가 종료하면, caller는 인자를 전달하기 위해 사용한 스택을 정리한다.

#### x86-64 calling convention: SYSV ABI
* caller는 callee에게 인자를 레지스터로 마지막 인자부터 첫 번째 인자까지 저장하여 전달한다. 만약 레지스터 수 보다 인자 수가 많을 경우, 마지막 인자부터 일부 인자들은 스택을 통해 전달된다.
* callee를 call할 경우, 복귀 주소(caller가 callee를 호출한 이후의 명령 주소)가 push된다. callee가 종료되었을 경우 해당 주소를 이용하여 복귀한다.
* 함수의 도입부(prologue)에는 rbp(SFP, Stack Frame Pointer)를 지정하고, 기존의 rbp를 push하여 함수가 종료될 때 해당 값을 pop하여 기존의 스택 프레임으로 복구할 수 있다.
```nasm
push rbp
mov rbp, rsp
```
* 스택 프레임을 형성한 후, rsp의 값을 빼서 callee의 지역 변수 공간을 할당하며 새로운 스택 프레임을 형성한다.
* callee의 반환 값을 rax 레지스터에 저장한다.
* leave를 통해 rbp를 pop하고 기존 스택 프레임을 복구한다. 또한 ret을 이용하여 rip에 복귀 주소를 pop하여 복귀한다.

![image](https://user-images.githubusercontent.com/55453184/175956686-99482a23-a3b8-4ead-9c13-ce20d7f4005b.png)

#### Return Address Overwrite
스택의 뒤에 위치한 함수의 반환 주소를 툭정 함수의 주소로 변조하여 원하는 함수가 실행되게 하는 공격 기법. 프로그램에 입력할 페이로드(공격을 위해 프로그램에게 전달하는 데이터)를 구성하여 공격한다.
* 페이로드의 구성  
![image](https://user-images.githubusercontent.com/55453184/175957925-64e263ac-e033-4329-b525-4aabc66c2230.png)

#### Patch RAO Vulnerability
![image](https://user-images.githubusercontent.com/55453184/175958456-1073cca3-b511-4b47-9c5e-9688fd1c42ac.png)
