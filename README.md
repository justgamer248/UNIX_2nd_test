Makefile~cmd(date)까지

# Makefile (makefile)
- 소스 파일이 여러 개이고 실행 파일도 여러 개라면 일일이 컴파일 하기 불편함. 그럴 때
간편하게 make만 치면 해결

- $ vi Makefile에서
-     test1 : test1.c
      (tab)gcc -o test1 test1.c
- 입력하면 이후 test1.c에 변경 사항 있을 시 세로 컴파일
## 소스파일에서 바로 실행파일을 생성하고자 할 때
-     all : 실행_파일이름들 (하나인 경우 딱히 없어도 무방)
      실행_파일이름 : 소스_파일이름들_나열(소스가 여러 개일 경우 모두 나열)
      (tab)gcc –o 실행_파일이름 소스_파일이름들_나열
##  소스파일->목적어파일->실행파일을 생성하고자 할 때
-     실행_파일_이름 : 목적어_파일이름들_나열
      (tab)gcc –o 실행_파일이름 목적어_파일이름들_나열

      목적어_파일_이름 : 소스_파일이름
      (tab)gcc –c 소스_파일이름
## 자신이 작성한 해더 파일을 포함 시키고자 하면
- 해더 파일
-      extern 리턴 타입 함수 이름(매개변수(이름 없음));
- 소스 파일
-      #include "해더 파일 이름.h"
       리턴 타입 함수이름(매개변수){
        // 내용 
       }
- makefile
-      실행 파일 : 목적어 파일
       (tab)gcc -o 실행 파일 이름 목적어 파일
 
       목적어 파일 : 소스 파일 해더 파일
       (tab)gcc -c 소스 파일
## clean 룰 && 매크로 상수 
- $ make clean을 사용하여 rm실행
- Makefile 최하단에
-       clean:
        (tab)rm –f *.o test1 test2 t1 t2 // 모든 실행파일 이름을 기술할 것
- 매크로 상수 : 기존 Makefile의 맨 앞에 상수를 정의
-       (최상단)이름 = 파일들
- 사용시 : $(이름)
# cmd  
- 우선 지금까지 구현한 명령어들은 하나의 구조체에 배열의 원소로써 저장됨
- 즉 cmd_tbl_t 이라는 구조체의 배열은 각각의 정보를 담고있는 원소를 가짐

|이름|처리함수이름|인자개수|옵션|
|:---:|:---:|:---:|:---:|
|cd|cd|AC_LESS_1|NULL|
|chmod|changemod|2|NULL|
|cp|cp|2|NULL|
|date|date|0|NULL|
|echo|echo|AC_ANY|NULL|
|exit|quit|0|NULL|
|help|help|0|NULL|
|hostname|hostname|0|NULL|
|id|id|AC_LESS_1|NULL|
|ln|ln|2|-s|
|ls|ls|AC_LESS_1|-l|
|mkdir|makedir|1|NULL|
|mv|mv|2|NULL|
|pwd|pwd|0|NULL|
|rm|rm|1|NULL|
|rmdir|removedir|1|NULL|
|uname|unixname|0|-a|
|whoami|whoami|0|NULL|

- 각 명령어에 대한 설명은 cmd.c 파일을 다운로드하여 AI에게 질문할것

## 전체적인 흐름
- 







