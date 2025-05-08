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
-      extern 리턴_타입 함수이름(매개변수(이름 없음));
- 소스 파일
-      #include "해더_파일이름.h"
       리턴_타입 함수이름(매개변수){
        // 내용 
       }
- makefile
-      실행 파일 : 목적어_파일
       (tab)gcc -o 실행_파일이름 목적어_파일
 
       목적어_파일 : 소스_파일 해더_파일
       (tab)gcc -c 소스_파일
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

## 전체적인 흐름 (소스 파일을 같이 볼 것)
- main함수
1. cmd 라인과 카운트를 선언한다.
2. setbuf함수를 통해 출력, 에러 버퍼를 제거한다. (즉시 출력함)
3. help를 통해 명령어 리스트 출력
4. for문(무한 반복)
5. 명령 프롬프트를 형식에 맞게 출력
6. if의 조건문 안쪽에 fgets를 통해 cmd_line에 저장
7. 만일 NULL인 경우 탈출
8. 다음 if의 경우 get_argv_optv(cmd_line)를 통해 유효성 확인
9. NULL이 아니면 proc_cmd를 통해 명령어를 처리하고 cmd_count증가
10. 4로 이동↑

- get_argv_optv함수
1. cmd_line의 시작 주소를 입력 받는다.
2. 잘라낸 문자열의 시작 주소를 담을 tok와 전역 변수 argc와 optc를 0으로 초기화
3. if의 조건문에서 전역변수cmd에 strtok를 통해 (공백||탭||엔터)로 구분한 첫단어를 대입
4. 그냥 엔터만 친 경우 NULL리턴 
5. for의 종료 조건은 잘라낸 단어가 NULL이 아닐때 즉 끝까지 자른다.
6. 만일 잘라낸 단어의 첫글자(tok[0])가 '-'면 optv[optc]에 현재주소(tok)를 저장하고 optc 1증가
7. 그 외의 경우 argv[argc]에 저장 및 argc 1증가
8. 반복문 탈출후 첫단어(cmd) 주소 반환

- progcmd함수
1. 변수k 선언
2. for문은 num_cmd만큼 실행됨 num_cmd는 전역 변수로 cmd_tbl의 원소개수 임
3. 만일 cmd와 cmd_tbl[k].cmd중 하나라도 일치하지 않는다면 지원되지 않음 출력
4. 일치하는 경우 check_arg와 check_opt를 통해 하나라도 잘못되면 print_usage에서 사용법 출력 및 종료
5. 그 외의 경우(정상) 해당하는 함수 호출 및 종료
