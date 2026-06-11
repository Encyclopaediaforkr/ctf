```c

int menu()
{
  puts("Menu:");
  puts("0. Make note");
  puts("1. Edit note");
  puts("2. Copy note");
  puts("3. Delete note");
  puts("4. Exit");
  return printf("Choice: ");
}


unsigned __int64 sub_1269()
{
  int v0; // ebx
  unsigned int idx; // [rsp+0h] [rbp-30h] BYREF
  unsigned int size; // [rsp+4h] [rbp-2Ch] BYREF
  _QWORD *v4; // [rsp+8h] [rbp-28h]
  __int64 v5; // [rsp+10h] [rbp-20h]
  unsigned __int64 v6; // [rsp+18h] [rbp-18h]

  v6 = __readfsqword(0x28u);
  printf("Index and size: ");
  __isoc99_scanf("%d %d", &idx, &size);
  if ( idx >= 8 )
  {
    puts("Invalid index");
    exit(1);
  }
  size = (size + 8) & 0xFFFFFFF8;
  v0 = idx;
  *((_QWORD *)&qword_4060 + v0) = malloc((int)size + 24LL);
  if ( !*((_QWORD *)&qword_4060 + (int)idx) )
  {
    puts("Memory allocation failed");
    exit(1);
  }
  **((_QWORD **)&qword_4060 + (int)idx) = (int)size;
  v4 = (_QWORD *)(*((_QWORD *)&qword_4060 + (int)idx) + 8LL);
  v5 = (int)size;
  v4 = (_QWORD *)((char *)v4 + (int)size);
  *v4 = __readfsqword(0x28u);
  printf("Note %d created\n", idx);
  return v6 - __readfsqword(0x28u);
}

unsigned __int64 edit_note()
{
  unsigned int idx; // [rsp+8h] [rbp-28h] BYREF
  int size; // [rsp+Ch] [rbp-24h] BYREF
  unsigned __int64 v3; // [rsp+28h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  printf("Index and size: ");
  __isoc99_scanf("%d %d", &idx, &size);
  if ( idx >= 8 )
  {
    puts("Invalid index");
    exit(1);
  }
  printf("Data: ");
  read(0, (void *)(qword_4060[idx] + 8LL), size - 1);
  *(_BYTE *)(qword_4060[idx] + size - 1 + 8LL) = 0;
  return v3 - __readfsqword(0x28u);
}

unsigned __int64 __fastcall copy_note(void *a1)
{
  unsigned int idx; // [rsp+14h] [rbp-Ch] BYREF
  unsigned __int64 v3; // [rsp+18h] [rbp-8h]

  v3 = __readfsqword(0x28u);
  printf("Index: ");
  __isoc99_scanf("%d", &idx);
  if ( idx > 7 || !qword_4060[idx] )
  {
    puts("Invalid index");
    exit(1);
  }
  memcpy(a1, (const void *)(qword_4060[idx] + 8LL), *(_QWORD *)qword_4060[idx]);
  *((_BYTE *)a1 + *(_QWORD *)qword_4060[idx] - 1) = 0;
  return v3 - __readfsqword(0x28u);
}


unsigned __int64 delete_note()
{
  unsigned int idx; // [rsp+4h] [rbp-Ch] BYREF
  unsigned __int64 v2; // [rsp+8h] [rbp-8h]

  v2 = __readfsqword(0x28u);
  printf("Index: ");
  __isoc99_scanf("%d", &idx);
  if ( idx > 7 || !qword_4060[idx] )
  {
    puts("Invalid index");
    exit(1);
  }
  free((void *)qword_4060[idx]);
  qword_4060[idx] = 0LL;
  printf("Note %d deleted\n", idx);
  return v2 - __readfsqword(0x28u);
}


__int64 __fastcall main(int a1, char **a2, char **a3)
{
  __int64 result; // rax
  int v4; // [rsp+Ch] [rbp-114h] BYREF
  char v5[264]; // [rsp+10h] [rbp-110h] BYREF
  unsigned __int64 v6; // [rsp+118h] [rbp-8h]

  v6 = __readfsqword(0x28u);
  setvbuf(stdin, 0LL, 2, 0LL);
  setvbuf(stdout, 0LL, 2, 0LL);
  setvbuf(stderr, 0LL, 2, 0LL);
  while ( 2 )
  {
    menu();
    if ( (unsigned int)__isoc99_scanf("%d", &v4) != 1 )
      return 0LL;
    switch ( v4 )
    {
      case 0:
        create_note();
        continue;
      case 1:
        edit_note();
        continue;
      case 2:
        copy_note(v5);
        printf("Note: %s\n", v5);
        continue;
      case 3:
        delete_note();
        continue;
      case 4:
        result = 0LL;
        break;
      default:
        puts("Invalid command");
        result = 1LL;
        break;
    }
    break;
  }
  return result;
}
```

---
```c
   0x5570cc0798cc:	lea    -0x110(%rbp),%rax
   0x5570cc0798d3:	mov    %rax,%rdi
   0x5570cc0798d6:	call   0x5570cc07954a
   0x5570cc0798db:	lea    -0x110(%rbp),%rax
   0x5570cc0798e2:	mov    %rax,%rsi
   0x5570cc0798e5:	lea    0x7d7(%rip),%rax        # 0x5570cc07a0c3
   0x5570cc0798ec:	mov    %rax,%rdi
   0x5570cc0798ef:	mov    $0x0,%eax
   0x5570cc0798f4:	call   0x5570cc079110 <printf@plt>

```

v5 == a1 offset rbp-0x110 겟또다제
272 만큼이 offset 265만큼 넣어야함

