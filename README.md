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

-       typedef struct{
            char *cmd;          // 첫단어
            void (*func)(void); // 함수 이름
            int argc;           // 인자 개수
            char *opt;          // 옵션 문자
            char *arg;          // 사용법 출력시 출력할 내용
       } cmd_tbl_t;
    
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

### 전역 변수

|변수|역활|
|:---:|:---:|
|*cmd|입력한 라인의 첫단어|
|*argv[100]|입력한 라인의 두번째 단어 부터의 배열|
|*optv[10]|옵션을 저장하는 배열|
|argc|단어 카운터|
|optc|옵션 카운터|
|cur_work_dir[SZ_STR_BUF]|현재 작업 디렉토리 이름을 저장|
|num_cmd|총 명령어 개수|

### 메크로 상수 & 함수

|변수 or 함수|역활|
|:---:|:---:|
|SZ_STR_BUF|256만큼의 문자열 길이|
|PRINT_ERR_RET()|perror()를 통해 에러 출력|
|EQUAL(_s1, _s2)|strcmp를 통해 같으면 true|
|NOT_EQUAL(_s1, _s2)|strcmp를 통해 다르면 true|
|AC_LESS_1|-1 명령어 인자 개수가 0 또는 1인 경우|
|AC_ANY|-100 명령어 인자 개수가 제한 없는 경우(echo)|

### main함수
1. cmd_line 과 카운터를 선언한다.
2. setbuf함수를 통해 출력, 에러 버퍼를 제거한다. (즉시 출력함)
3. help를 통해 명령어 리스트 출력
4. for문(무한 반복)
5. 명령 프롬프트를 형식에 맞게 출력
6. if의 조건문 안쪽에 fgets를 통해 cmd_line에 저장
7. 만일 NULL인 경우 탈출
8. 다음 if의 경우 get_argv_optv(cmd_line)를 통해 단어를 자름
9. NULL이 아니면 proc_cmd를 통해 명령어를 처리하고 cmd_count증가
10. 4로 이동↑

|변수|역활|
|:---:|:---:|
|cmd_line[SZ_STR_BUF]|SZ_STR_BUF만큼 명령어 라인 저장|
|cmd_count|입력한 명령어 카운트|

### get_argv_optv함수 (cmd_line자르기)
1. cmd_line의 시작 주소를 받는다.
2. 잘라낸 문자열의 시작 주소를 담을 tok와 전역 변수 argc와 optc를 0으로 초기화
3. if의 조건문에서 전역변수cmd에 strtok를 통해 (공백||탭||엔터)로 구분한 첫단어를 대입
4. 그냥 엔터만 친 경우 NULL리턴 
5. for의 종료 조건은 잘라낸 단어가 NULL이 아닐때 즉 끝까지 자른다.
6. 만일 잘라낸 단어의 첫글자(tok[0])가 '-'면 optv[optc]에 현재주소(tok)를 저장하고 optc 1증가
7. 그 외의 경우 argv[argc]에 저장 및 argc 1증가
8. 반복문 탈출후 첫단어(cmd) 주소 반환

|변수|역활|
|:---:|:---:|
|*tok|첫단어|

### progcmd함수 (유효성 확인)
1. 변수k 선언
2. for문은 num_cmd만큼 실행됨 num_cmd는 전역 변수로 cmd_tbl의 원소개수 임
3. 만일 cmd와 cmd_tbl[k].cmd중 하나라도 일치하지 않는다면 지원되지 않음 출력
4. 일치하는 경우 check_arg와 check_opt를 통해 하나라도 잘못되면 print_usage에서 사용법 출력 및 종료
5. 그 외의 경우(정상) 해당하는 함수 호출 및 종료

|변수|역활|
|:---:|:---:|
|k|for문 제어용 변수|

# 나올 만한 문제 (각 함수별로 1개씩)

- <span style="color:red">정답은 끝에 있음</span>

1. cd함수에 들어갈 내용은?
-     pwp = getpwuid(????);
2. changemod함수는 이미 ???이라는 유닉스 함수가 존재하기 때문에 이름을 바꿨다.
3. date함수에 들어갈 내용은?
-     char stm[128];
      time_t ttm = time(NULL);
      struct tm *??? = localtime(&ttm);
4. echo함수에 들어갈 내용은?
-     for(int i=0; i<????; i++) printf("%s ", argv[i]);
      printf("\n");
5. hostname함수에 들어갈 내용은?
-     char hostname[SZ_STR_BUF];
      ??????(hostname, SZ_STR_BUF);
      printf("%s\n", hostname);
6. id함수에 들어갈 내용은?
-     struct ????? *pwp;
      struct ????? *grp;
7. ln함수에 들어갈 내용은?
-     if( ((???? != 0) ? symlink(argv[0], argv[1]) : link(argv[0], argv[1])) < 0)
		PRINT_ERR_RET();
8. ls함수에 들어갈 내용은?
-     // 디렉토리 이름을 주지 않았다면 현재 디렉토리 설정
      path = (???? == 0)? ".": argv[0];
9. makedir함수에 들어갈 내용은?
-     if((link(argv[0], argv[1]) < 0) || (unlink(argv[0]) < 0))
		???????????;
10. mv함수에 들어갈 내용은?
-     if((link(argv[0], argv[1]) < 0) || (???????(argv[0]) < 0))
		PRINT_ERR_RET();
11. pwd함수에 들어갈 내용은?
-     printf("%s\n", ?????);
12. exit함수는 이미 존재 한다. 그러므로 ???을 함수 이름으로 정의한다.
13. rm함수에 들어갈 내용은?
-     struct ???? buf;
14. removedir함수에 들어갈 내용은?
-     if(?????(argv[0]) < 0)
		PRINT_ERR_RET();
15. unixname함수에 들어갈 내용은?
-     struct utsname un;
      uname(&un);

      printf("%s", ?????????);
16. whoami함수에 들어갈 내용은?
-     struct passwd *pwp;
      pwp = ??????(getuid());
17. help함수에 들어갈 내용은?
-     int  k;

      printf("현재 지원되는 명령어 종류입니다.\n");
      for (k = 0; k < ???????; ++k)
		print_usage("  ", cmd_tbl[k].cmd, cmd_tbl[k].opt, cmd_tbl[k].arg);
      printf("\n");

## 정답

1. getuid()
2. chmod()
3. ltm
4. argc
5. gethostname
6. passwd, group
7. optc
8. argc
9. PRINT_ERR_RET();
10. unlink
11. cur_work_dir
12. quit
13. stat
14. rmdir
15. un.sysname
16. getpwuid
17. num_cmd
