```c
PNG
00 01 02 03 /04 05 06 07
89 50 4E 47 /0D 0A 1A 0A

  
49 45 4E 44 AE 42 60 82 -> 파일 끝

magic       CRLF + EOF markers
+08: IHDR chunk → 길이(4B) + "IHDR"(4B) + width(4B) + height(4B) + bi
```

```c
JPEG
+00 +01 +02 +03 +04 +09~
FF D8 FF E0 len JFIF\x00 (5B) SOI APP0 marker


끝은 FF D9 -> 파일끝
 (EOI)
```

```c
ZIP
+00 +01 +02 +03 +04~05 +06~
50 4B 03 04
ver flags/compression/...
Local File Header sig "PK"


Central Dir: PK\x01\x02 / EOCD: PK\x05\x06 -> 파일끝 == 50 4B 05 06
```

```c
ELF
+00~03 +04 +05 +06 +10~11 +12~13
7F 45 4C 46 01/02 01/02 01
               e_type
                      e_mach
.ELF magic
                             32/64bit
                                     LE/BE
ELF ver
2=EXEC, 3=SO, 4=CORE
```

```c
PE
+00~01 +3C PE sig (+e_lfanew) Machine
/4D 5A "MZ" / → PE offset 50 45 00 00 "PE\0\0" 8664/14C
DOS header e_lfanew
COFF header 시작
x86-64 / x86
```

```c
PDF
25 50 44 46 2D "%PDF-"

25 25 45 4F 46 -> 파일끝
ver (1.x)
```

```c
GIF
/47 49 46 38 /"GIF8" 37/39 "7a/9a" width(2B) height(2B)
magic 87a/89a ver logical screen (LE)

3B -> 파일끝

```


---

```c
png 파일기준 리눅스 추출방법

# 파일 기본 확인
file suspicious.png
strings suspicious.png | less         # 읽을 수 있는 문자열
exiftool suspicious.png               # 메타데이터 (GPS, 작성자, 코멘트 등)
binwalk suspicious.png                # 내부에 숨겨진 파일 시그니처 탐지
binwalk -e suspicious.png             # 자동 추출
foremost suspicious.png               # 카빙 (파일 경계 기반)
```

```c
suspicious.png (디스크)
│
├─ file        → 헤더 1회 읽기 → magic DB 매칭 → 포맷 출력
├─ strings     → 전체 선형 스캔 → ASCII 시퀀스만 stdout
├─ exiftool    → 포맷별 메타데이터 파싱 → 구조화된 필드 출력
│
├─ binwalk     → 슬라이딩 윈도우 스캔 → 시그니처 오프셋 테이블 출력
├─ binwalk -e  → 위 + 각 오프셋에서 추출기 실행 → _extracted/ 생성
│
└─ foremost    → header/footer 쌍 카빙 → output/ 디렉토리 생성
```

