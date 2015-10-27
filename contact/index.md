---
layout: page
title: Contact Me
---

Want to contact me? Find my email by assembling, linking, and running the [NASM](http://www.nasm.us) code below:

```nasm
; Want to know my email?
; Solve this puzzle!

extern printf

SECTION .data

email db 61h,73h,73h,69h,6ch,6bh
      db 73h,69h,6bh,73h,69h
      db '@gmail.com'
      db 10h,0

SECTION .text
global main

main:
    push rbp

    mov rdi, email
    mov rax, 0
    call printf

    pop rbp
    mov rax, 0
    ret
```

Too challenging? This should be clear if you know Python, or if you use a bit of common sense:

```python
email = 'moc.liamg@iskisklissa'
print(email[::-1]) # Reverse the string
```

Still can't get my email? Use the Twitter link in the sidebar!
