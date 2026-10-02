
### 🔎 코드 분석

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}

int main(int argc, char* argv[]){
	if(argc<2){
		printf("usage : %s [passcode]\n", argv[0]);
		return 0;
	}
	if(strlen(argv[1]) != 20){
		printf("passcode length should be 20 bytes\n");
		return 0;
	}

	if(hashcode == check_password( argv[1] )){
		setregid(getegid(), getegid());
		system("/bin/cat flag");
		return 0;
	}
	else
		printf("wrong passcode.\n");
	return 0;
}

```

fd 문제와 유사하게 인자로 받은 값을 처리하는 코드

```c
if(hashcode == check_password( argv[1] )){
		setregid(getegid(), getegid());
		system("/bin/cat flag");
		return 0;
	}
```

인자를 check_password 함수에 넣은 결과가 hashcode (0x21DD09EC)와 동일하게 만들면 된다

```c
unsigned long check_password(const char* p){
	int* ip = (int*)p;
	int i;
	int res=0;
	for(i=0; i<5; i++){
		res += ip[i];
	}
	return res;
}
```

check_password 함수는 받아온 인자를 int(4바이트)씩 쪼개서 모두 더한 결과를 return한다

### 🗡️ 익스플로잇

0x21DD09EC(= 568,134,124)를 5로 나누면 안 나눠떨어짐

113,626,824 * 4 + 113,626,828 로 나눌 수 있음

0x06C5CEC8 * 4 + 0x06C5CECC

```c
./col $(python3 -c "import sys; sys.stdout.buffer.write(b'\xc8\xce\xc5\x06'*4 + b'\xcc\xce\xc5\x06')")
```