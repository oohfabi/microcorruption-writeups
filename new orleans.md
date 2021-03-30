# New Orleans

let's have a look at the **`<main>`** function first:

```asm
4438 <main>
4438:  3150 9cff      add	#0xff9c, sp
443c:  b012 7e44      call	#0x447e <create_password>
4440:  3f40 e444      mov	#0x44e4 "Enter the password to continue", r15
4444:  b012 9445      call	#0x4594 <puts>
4448:  0f41           mov	sp, r15
444a:  b012 b244      call	#0x44b2 <get_password>
444e:  0f41           mov	sp, r15
4450:  b012 bc44      call	#0x44bc <check_password>
4454:  0f93           tst	r15
4456:  0520           jnz	#0x4462 <main+0x2a>
4458:  3f40 0345      mov	#0x4503 "Invalid password; try again.", r15
445c:  b012 9445      call	#0x4594 <puts>
4460:  063c           jmp	#0x446e <main+0x36>
4462:  3f40 2045      mov	#0x4520 "Access Granted!", r15
4466:  b012 9445      call	#0x4594 <puts>
446a:  b012 d644      call	#0x44d6 <unlock_door>
446e:  0f43           clr	r15
4470:  3150 6400      add	#0x64, sp
```

it prints some different things on the console like:

"***Enter the password to continue"***

***"Invalid password; try again."***

***"Access Granted!"***

but the **`<create_password>`** looks very interesting:

```asm
447e <create_password>
447e:  3f40 0024      mov	#0x2400, r15
4482:  ff40 6500 0000 mov.b	#0x65, 0x0(r15)
4488:  ff40 3c00 0100 mov.b	#0x3c, 0x1(r15)
448e:  ff40 7900 0200 mov.b	#0x79, 0x2(r15)
4494:  ff40 4d00 0300 mov.b	#0x4d, 0x3(r15)
449a:  ff40 3100 0400 mov.b	#0x31, 0x4(r15)
44a0:  ff40 2100 0500 mov.b	#0x21, 0x5(r15)
44a6:  ff40 5d00 0600 mov.b	#0x5d, 0x6(r15)
44ac:  cf43 0700      mov.b	#0x0, 0x7(r15)
44b0:  3041           ret
```

there are some `mov.b` instructions  (the `.b` suffix means that the instruction moves only 1 byte to the register `r15`)

r15 is located at memory address `2400`

so this function is probably creating our password.

let's start debugging and step trough the function

this is what the address `2400` looks like (after the function `<create_password>`)

`2400: 653c 794d 3121 5d00 e<yM1!].`

so let's try enter `e<yM1!]`

**Success!**

this input worked for me !
