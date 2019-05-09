# cs2400-arm-classwork-1

CS2400 ARM assembly programming classwork problems.

## C and assembly code

Follow the instructions for each of the following code samples in [Compliler Explorer](https://godbolt.org).

1. [printf](https://godbolt.org/z/y2YKew)
   1. What is the library function that is called?
        The C library function that is being used is printf. Printf is declared in the stdio.h header.

   2. Research the implementation (source code) of this function.
        From the compiler explorer program, the source code for printf is line 5 and 6 which is
            ldr     r0, .L3
            bl      puts
        the programs reads .L3, which reads, which reads the word .LC0, which reads "Hello World" and stores it in R0.

   3. Find out if the program directly executes the output operation or it makes a *system call* to the operating system.
        The program directly executes the output operation. a "system call" to the operating system would require us to ask
        for an operating system service, which is what we are not doing.
   
2. [malloc](https://godbolt.org/z/kAZX7x)
   1. How are the arguments passed to `malloc` and `free`?
        malloc's arguments are being passed by (sizeof(int)) which allocates space off the heap to store an int.
        free's arguments are being passed by the variable p which is not indicated in this program.

   2. Research the implementation (source code) of `malloc` and `free`.
        From the compiler explorer program, the source code for malloc are lines 4, 6, and 7 which are
            movs    r0, #4
            bl      malloc
            mov     r3, r0
            str     r3, [sp, #4]
        The function malloc() will allocate memory that of type int.
        From the compiler explorer program, the source code for free is line 8 and 9 which are
            ldr     r0, [sp, #4]
            bl      free
        you add 4 to the sp, you put the result into ro and load ro. You need to eventually call free exactly once for
        every malloc. If you forget to call free, you'll get a long-running program.

3. [malloc array](https://godbolt.org/z/bBl0zx)
   1. How does this case differ from the previous one?
        In this program you are multiplying (sizeof(int)) with ARRSIZE. That size of the memory block is being explicitly
        casted to a pointer int. all of this is being set to a pointer *iarr of type int.

   2. [**hard**] Write your own tiny `malloc` library by declaring a large `FILL` area and writing a `malloc` and a `free` subroutines that manage allocations to that memory area. 
      1. `malloc` works approximately as follows:
         - it takes as argument the number of bytes requested
         - it finds an appropriate region of the heap that can satisfy the request and marks it as allocated
         - it adds a small *metadata* memory block of known size on top of the region and writes the size of the region in the block
         - returns a pointer to the region, not including the metadata block
         - if the allocation fails for any reason (e.g. running out of memory), it returns `NULL` (0x00000000), which is an invalid address for user programs
         - allocates regions in from-lower-to-higher address orientation
      2. `free` works approximately as follows:
         - it takes the pointer returned by `malloc`
         - it subtracts the size of the metadata block from this address
         - it reads the size of the allocated region
         - it marks the cumulative (metadata block + region) previously allocated area as available
      3. Non-interleaving of calls:
         - assume all the calls to `malloc` precede all the calls to `free`
         - assume the `free` calls will be made in the reverse order of the `malloc` calls, as in 
           ```c
           int *p1 = (int *) malloc(200 * sizeof(int));
           int *p2 = (int *) malloc(300 * sizeof(int));
           int *p3 = (int *) malloc(100 * sizeof(int));
           
           // use memory...
           
           free(p3);
           free(p2);
           free(p1);
           ```
         - this relieves you from the responsibility to manage *holes* in the memory area
      4. Variables you will need:
         - `memstore` (standing for _memory store_) - starting address of the fill area
         - `storeptr` (standing for _store pointer_) - a pointer to the next available memory region
      5. Stack:
         - manage the subroutine (aka function) calls by creating stack frames for them to hold their arguments as well as any local variables that are needed
         - don't forget to manage the register bank and not overwrite register value that are needed by the *caller*
         - if you need to save registers on the stack, you can do it within the frame of the *callee*, correctly expanding the frame to fit them (note that VisUAL does not support `push` and `pop`)
         - note that a function's stack frame only lasts between the start and end of execution of the subroutine, regardless of how many times the subroutine is called
   
4. [arrays](https://godbolt.org/z/lcH006)
   1. Port this code to VisUAL.
      - for an example, refer to the [negate repo](https://github.com/ivogeorg/cs2400-arm-asm-negate-exercise.git)
        1. Open the [Compiler Explorer](https://godbolt.org/)
        2. Select the **ARM gcc 8.2** compiler
        3. Enter the following compiler options in the box on the right: `-fomit-frame-pointer -mcpu=cortex-m3 -mtune=cortex-m3`
        4. Put the code of the [`negate.c`](https://github.com/ivogeorg/cs2400-arm-asm-negate-exercise/blob/master/negate.c) file in the left pane
        5. The compiler will generate the assembly for the long `negate` program, that is not directly executable in VisUAL
        6. The file [`negate_gcc_8_2.S`](https://github.com/ivogeorg/cs2400-arm-asm-negate-exercise/blob/master/negate_gcc_8_2.S) contains the assembly ported to VisUAL
      - special attention on:
        1. Unsupported instructions (Just **Execute** on VisUAL and it will mark them)
           - in particular, `BX` is not supported, so you will need to use a new label or `MOV pc, rl` to return from a subroutine
        2. Memory regions (VisUAL only supports `DCD` and `FILL` directives)
        3. Start of execution (VisUAL starts executing the top line, so you might need to rearrange your code)
           - alternatively, you can define a `_start` label and load it into `pc`
                main
                		sub		sp, sp, #28
                		adr		r3, words
                		mov		r4, sp
                		mov		r5, r3
                		ldmia	r5!, {r0, r1, r2, r3}
                		stmia	r4!, {r0, r1, r2, r3}
                		ldr		r3, [r5]
                		str		r3, [r4]
                		movs	r3, #0
                		str		r3, [sp, #20]
                		b		L4
                L6
                		ldr		r3, [sp, #20]
                		lsls	r3, r3, #2
                		add		r2, sp, #24
                		add		r3, r3, r2
                		ldr		r3, [r3, #-24]
                		cmp		r3, #0
                		bge		L5
                		ldr		r3, [sp, #20]
                		lsls	r3, r3, #2
                		add		r2, sp, #24
                		add		r3, r3, r2
                		ldr		r3, [r3, #-24]
                		mov		r0, r3
                		bl		negate

                L9		mov		r2, r0
                		ldr		r3, [sp, #20]
                		lsls	r3, r3, #2
                		add		r1, sp, #24
                		add		r3, r3, r1
                		str		r2, [r3, #-24]
                L5
                		ldr		r3, [sp, #20]
                		adds	r3, r3, #1
                		str		r3, [sp, #20]
                L4
                		movs	r2, #5
                		ldr		r3, [sp, #20]
                		cmp		r3, r2
                		blt		L6
                		movs	r3, #0
                		mov		r0, r3
                		add		sp, sp, #28
                		end

                negate
                		sub		sp, sp, #8
                		str		r0, [sp, #4]
                		ldr		r3, [sp, #4]
                		rsbs	r3, r3, #0
                		mov		r0, r3
                		add		sp, sp, #8
                		mov		pc, lr

                words	dcd		1, -2, 5, -6, 2

   2. Observe/show that this code writes the local array in reverse order to the `static` global array.
        With static global array, it will run until after main exits, so it will be in reverse compared to the local array.
        Or it could be that it is in series in which a series data is stored in reverse order.
   
5. [2d array](https://godbolt.org/z/Kr-Sn8)
   1. Port this code to VisUAL.
   2. How are nested `for` loops handled in assembly? Are they *"nested"* in assembly?
        They are handles in the way of mov, str, and b instructions. The str instructions are adding 4 bytes to the current address used to access the next value.
        They are nested in assembly. They work similar to the C program. The first one can't be used without the second.
   
6. [2d array with mul](https://godbolt.org/z/cHwSTR)
   1. Port this code to VisUAL. (It's the same as the previous but with multiplicatoin).
   2. Add your 32-bit unsigned integer multiplication algorithm as a subroutine and run the code. Verify its correctness.
        main
        		sub		sp, sp, #8
        		movs	r3, #0
        		str		r3, [sp, #4]
        		b		L2
        L5
        		movs	r3, #0
        		str		r3, [sp]
        		b		L3
        L4
        		ldr		r3, [sp, #4]
        		ldr		r2, [sp]
        numbers	DCD		0b1100101010010001, 0b1111010101000011
        result	FILL	8
        carry	FILL	4
        		ADR		r0, numbers
        		LDR		r1, [r0]
        		LDR		r2, [r0, #4]
        		MOV		r3, #0
        		MOV		r4, #0
        		MOV		r5, #0
        loop	TST		r2, #1
        		BEQ		shift
        		ADDS	r4, r4, r1
        		ADC		r5, r5, r3
        shift	LSR		r2, r2, #1
        		LSL		r3, r3, #1
        		LSLS	r1, r1, #1
        		ADC		r3, r3, #0
        		CMP		r2, #0
        		BGT		loop
        		ADR		r0, result
        		STR		r4, [r0]
        		ADR		r0, carry
        		STR		r5, [r0]
        		END
        		lsls	r1, r3, #1
        		adr		r0, L7
        		ldr		r2, [sp, #4]
        		mov		r3, r2
        		lsls	r3, r3, #2
        		add		r3, r3, r2
        		ldr		r2, [sp]
        		add		r3, r3, r2
        		str		r1, [r0, r3, lsl #2]
        		ldr		r3, [sp]
        		adds	r3, r3, #1
        		str		r3, [sp]
        L3
        		ldr		r3, [sp]
        		cmp		r3, #4
        		ble		L4
        		ldr		r3, [sp, #4]
        		adds	r3, r3, #1
        		str		r3, [sp, #4]
        L2
        		ldr		r3, [sp, #4]
        		cmp		r3, #9
        		ble		L5
        		movs	r3, #0
        		mov		r0, r3
        		add		sp, sp, #8
        		MOV		pc, r0
        L7		DCD		5, 10

## Submission
1. Fork this repository
2. Clone and implement.
3. Commit any modifications.
4. Push your commits to your remote.
5. For research-type questions, answer inline in the [README](README.md).

