```c
NTAS \ [root] \ config
```

여기서 하이브파일을 모조리 추출해야함

- **SYSTEM**: 악성코드가 시스템 '서비스'로 등록되지는 않았는가?
- **SOFTWARE**: 모든 사용자를 대상으로 하는 '자동 실행 프로그램'으로 설치되지는 않았는가?
- **SAM**: 공격자가 새로운 '관리자 계정'을 만들지는 않았는가?
- **SECURITY**: 시스템의 '보안 정책'을 변경하지는 않았는가?
- **NTUSER.DAT**: 특정 사용자가 '로그인했을 때만' 악성 행위를 하도록 숨어있지는 않은가


dirty clean시 RLA 프로그램 여기서 다운
https://ericzimmerman.github.io/#forensic-tools