https://pwnable.kr/play/1


```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
	if(argc<2){
		printf("pass argv[1] a number\n");
		return 0;
	}
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
	if(!strcmp("LETMEWIN\n", buf)){
		printf("good job :)\n");
		setregid(getegid(), getegid());
		system("/bin/cat flag");
		exit(0);
	}
	printf("learn about Linux file IO\n");
	return 0;

}
```

### 🔎 코드 분석

```c
if(!strcmp("LETMEWIN\n", buf)){
```

buf에 들어있는 값이 LETMEWIN이면 flag를 구할 수 있음

버퍼 입력받는 부분을 보자

```c
	int fd = atoi( argv[1] ) - 0x1234;
	int len = 0;
	len = read(fd, buf, 32);
```

실행 시 받은 인자에서 0x1234를 뺀 값을 file descriptor로 사용하여 값을 읽는다

우리가 임의로 정할 수 있는 값은 file descriptor의 값 뿐이다

### ✨ 핵심 개념

**❓ 파일 디스크립터**

리눅스 계열에서 프로세스가 파일을 다룰 때 사용

프로세스가 파일을 접근할 때 디스크립터라는 개념을 이용

0, 1, 2번은 선점되어있고 그 이후로 하나씩 번호가 매겨짐

- 0번 - 표준 입력
- 1번 - 표준 출력
- 2번 - 표준 에러

### 🗡️ 익스플로잇

파일 디스크립터를 표준 입력 0으로 만들고 표준 입력으로 LETMEWIN을 입력하면 된다

파일 디스크립터로 입력받는 값이 첫 번째 인자 - 0x1234가 되므로

```c
int fd = atoi( argv[1] ) - 0x1234;
```

0x1234의 10진수 값을 입력해주면 된다

