welcome
===

# Description

> 이 문제는 서버에서 작동하고 있는 서비스(welcome)의 바이너리와 소스 코드가 주어집니다.
> "접속 정보 보기"를 눌러 서비스 정보를 얻은 후 플래그를 획득하세요.
> 서버로부터 얻은 플래그의 내용을 워게임 사이트에 인증하면 점수를 획득할 수 있습니다.
> 플래그의 형식은 DH{…} 입니다.

# Source code

~~~
#include <stdio.h>
int main(void) {
FILE *fp;
char buf[0x80] = {};
size_t flag_len = 0;
printf("Welcome To DreamHack Wargame!\n");
fp = fopen("/flag", "r");
fseek(fp, 0, SEEK_END);
flag_len = ftell(fp);
fseek(fp, 0, SEEK_SET);
fread(buf, 1, flag_len, fp);
fclose(fp);
printf("FLAG : ");
fwrite(buf, 1, flag_len, stdout);
}
~~~

네트워크 연결 하면 플래그를 바로 보여주기에 소스코드에서 공략할 취약점이 없다.

# Result

~~~
stella@ubuntu:~/Desktop$ nc host1.dreamhack.games 8247
Welcome To DreamHack Wargame!
FLAG : DH{5cc72596cba7104569abb37f71b8ccf3}
~~~

# Learned

>  nc {주소} {포트} :이 명령어를 통해 네트워크에 접속 할 수 있다.