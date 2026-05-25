```c
int menu()
{
  puts("1. create");
  puts("2. read");
  puts("3. update");
  puts("4. delete");
  return printf("> ");
}

unsigned __int64 delete_note()
{
  unsigned int idx; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  printf("idx: ");
  __isoc99_scanf("%d", &idx);
  if ( idx >= 0xA )
    exit(1);
  if ( !chunks[idx].ptr )
    exit(1);
  free((void *)chunks[idx].ptr);
  return v2 - __readfsqword(0x28u);
}

unsigned __int64 update_note()
{
  unsigned int idx; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  printf("idx: ");
  __isoc99_scanf("%d", &idx);
  if ( idx >= 0xA )
    exit(1);
  if ( !chunks[idx].ptr )
    exit(1);
  printf("data: ");
  read(0, (void *)chunks[idx].ptr, chunks[idx].size);
  return v2 - __readfsqword(0x28u);
}


unsigned __int64 read_note()
{
  unsigned int idx; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  printf("idx: ");
  __isoc99_scanf("%d", &idx);
  if ( idx >= 0xA )
    exit(1);
  if ( !chunks[idx].ptr )
    exit(1);
  printf("data: %s\n", (const char *)chunks[idx].ptr);
  return v2 - __readfsqword(0x28u);
}



unsigned __int64 create_note()
{
  unsigned int v0; // ebx
  unsigned int idx; // [rsp+Ch] [rbp-24h] BYREF
  size_t size; // [rsp+10h] [rbp-20h] BYREF
  unsigned __int64 v4; // [rsp+18h] [rbp-18h]

  v4 = __readfsqword(0x28u);
  printf("idx: ");
  __isoc99_scanf("%d", &idx);
  if ( idx >= 0xA )
    exit(1);
  printf("size: ");
  __isoc99_scanf("%lu", &size);
  if ( size > 0x70 )
    exit(1);
  chunks[idx].size = size;
  v0 = idx;
  chunks[v0].ptr = (__int64)calloc(size, 1uLL);
  if ( !chunks[idx].ptr )
    exit(1);
  printf("data: ");
  read(0, (void *)chunks[idx].ptr, size);
  return v4 - __readfsqword(0x28u);
}



__int64 __fastcall main(__int64 a1, char **a2, char **a3)
{
  int num; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v5; // [rsp+8h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  sub_4016BD(a1, a2, a3);
  while ( 1 )
  {
    while ( 1 )
    {
      menu();
      __isoc99_scanf("%d", &num);
      if ( num != 4 )
        break;
      delete_note();
    }
    if ( num > 4 )
      break;
    switch ( num )
    {
      case 3:
        update_note();
        break;
      case 1:
        create_note();
        break;
      case 2:
        read_note();
        break;
      default:
        return 0LL;
    }
  }
  return 0LL;
}
```

---
ida로 본 바이너리 코드

---


```python
from pwn import *

p = process("./chall")
```