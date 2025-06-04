The new thing in this challenge is that you don't have much gadgets, so you have to leverage the code added by the compiler. `__libc_csu_init` is exploited in this scenario because of its structure:

``` asm
; __libc_csu_init
push    r15
push    r14
mov     r15, rdx
push    r13
push    r12
lea     r12, __frame_dummy_init_array_entry
push    rbp
lea     rbp, __do_global_dtors_aux_fini_array_entry
push    rbx
mov     r13d, edi
mov     r14, rsi
sub     rbp, r12
sub     rsp, 8
sar     rbp, 3
call    _init_proc
test    rbp, rbp
jz      short loc_400696
xor     ebx, ebx
nop     dword ptr [rax+rax+00000000h]

mov     rdx, r15            <--- [2]
mov     rsi, r14
mov     edi, r13d
call    [r12+rbx*8]
add     rbx, 1
cmp     rbp, rbx
jnz     short loc_400680

add     rsp, 8
pop     rbx                 <--- [1]
pop     rbp
pop     r12
pop     r13
pop     r14
pop     r15
retn
```

As you can see in the code above, the first gadget \[1\] may be called first to setup 6 registers, followed by the second gadget \[2\] which will setup the 3 registers that are needed to call ret2win. The problem is that the `call` instruction will not call the address of `r12 + rbx*8`, but the address _pointed by_ `r12 + rbx*8`. We therefore either need a write gadget, or an address that already points to an interesting function. In this case, we can simply solve this by calling the `_term_proc` function which does nothing (see code below) and whose address can be found in the program at `0x600e48` in this case. 

```
sub     rsp, 8
add     rsp, 8
retn
```

If we make sure to set `rbx` so that `rbx+1 == rbp`, then we will jump back to our first gadget. As we execute `ret` for the second time, we can now jump to a gadget that will set `rdi` only since that was not completely possible with the `mov edi, r13d` instruction of the second gadget. Finally, after all that is done we can `ret` to `ret2win` and get our flag!