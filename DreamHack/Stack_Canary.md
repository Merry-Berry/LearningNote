### Stack Canary
스택 영역의 변조를 막기 위한 보호 기법. 커널이 canary라는 무작위 값을 생성하여 스택 프레임의 전반부(prologue)에 저장한 후, 스택 프레임의 후반부(epilogue)에 canary가 변조되었는지 검사한다.

#### Push & Check Canary
* fs segment에서 생성된 canary를 rax에 저장하고 스택에 push한다.
* fs가 가리키는 것은 TLS로, 프로세스가 시작할 때 canary를 전역 변수로 저장한다. fs의 값은 gdb의 명령으로 알 수 없고, 특정 시스템 콜을 통해서만 파악 가능하다.
```nasm
mov    rax,QWORD PTR fs:0x28
mov    QWORD PTR [rbp-0x8],rax
```
* 전반부에 저장한 canary를 rcx에 저장하고, 원본 canary 값과 xor연산을 진행한다. 만약 두 값이 다를 경우, 스택에 저장된 canary가 변조된 것이므로 __stack_chk_fail@plt로 jump한다.
```nasm
mov    rcx,QWORD PTR [rbp-0x8]
xor    rcx,QWORD PTR fs:0x28
je     0x6f0 <main+70>
call   __stack_chk_fail@plt
```

#### Canary Leak & Exploit
* 읽기/쓰기 버퍼와 canary의 위치를 통해 canary leak을 할 수 있다. canary는 항상 \x00으로 끝나므로, 리틀 엔디언 환경에서 \x00~으로 저장된다. 따라서 입력 버퍼의 크기 + 1byte만큼 입력하고, 해당 버퍼를 출력할 경우 canary도 같이 출력된다.
* canary을 파악하면, stack buffer overflow를 진행한다. 이때 페이로드는 (shellcode)+(dummy)+(canary)+(dummy for SFP) + (address of shellcode)로 구성한다.
![image](https://user-images.githubusercontent.com/55453184/175966795-39ec4ccd-6211-445d-adc9-ca1d540d25b7.png)
