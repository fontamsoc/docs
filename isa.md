	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# PUxx Instruction Set

	All instructions are 16 bits except for the instructions
	li16/inc16/rli16, li32/inc32/rli32, li64/inc64/rli64
	which are respectively 32, 48, 80 bits.
	All instructions must be 16 bits aligned.


## Notations

	%ip is the instruction pointer register which hold
	the address of the current instruction, or in other words,
	the address of the instruction being decoded.

	ipnext refers to the address of the next instruction.

	%gpr refers to one of the general purpose registers %0 thru %15.
	%gpr1 and %gpr2 are used when an instruction makes use of two
	general purpose registers.

	msb stands for most significant bit(s).
	lsb stands for least significant bit(s).

	An example of binary representation
	of an instruction is as follow:
	|op|010|rrrr|iiii|
	'op' is a 5bits value in decimal; is
		4bits in binary for li8/inc8.
	'0' represent a bit with a value 0.
	'1' represent a bit with a value 1.
	'r' represent a bit of a gpr index;
		when eight of them are used,
		the first four correspond
		to %gpr1, and the second four
		correspond to %gpr2.
	'i' represent a bit of an immediate value.


## General Purpose Instructions

### Arithmetic

	add %gpr1, %gpr2			|23|000|rrrr|rrrr|
		%gpr1 += %gpr2;

	sub %gpr1, %gpr2			|23|001|rrrr|rrrr|
		%gpr1 -= %gpr2;

	mul %gpr1, %gpr2			|25|010|rrrr|rrrr|
		Signed low multiplication.
		%gpr1 *= %gpr2;
		the result of the integer multiplication
		is 32/64/128 bits, but only the 16/32/64 lsb of the result
		get stored in %gpr1, while the 16/32/64 msb can be
		retrieved using the instruction mulh.

	mulu %gpr1, %gpr2			|25|000|rrrr|rrrr|
		Unsigned low multiplication.
		%gpr1 *= %gpr2;
		the result of the integer multiplication
		is 32/64/128 bits, but only the 16/32/64 lsb of the result
		get stored in %gpr1, while the 16/32/64 msb can be
		retrieved using the instruction mulhu.

	mulh %gpr1, %gpr2			|25|011|rrrr|rrrr|
		Signed high multiplication.
		%gpr1 *= %gpr2;
		the result of the integer multiplication
		is 32/64/128 bits, but only the 16/32/64 msb of the result
		get stored in %gpr1, while the 16/32/64 lsb can be
		retrieved using the instruction mul.

	mulhu %gpr1, %gpr2			|25|001|rrrr|rrrr|
		Unsigned high multiplication.
		%gpr1 *= %gpr2;
		the result of the integer multiplication
		is 32/64/128 bits, but only the 16/32/64 msb of the result
		get stored in %gpr1, while the 16/32/64 lsb can be
		retrieved using the instruction mulu.

	div %gpr1, %gpr2			|25|110|rrrr|rrrr|
		Signed integer division.
		%gpr1 /= %gpr2;
		the sign of the quotient is positive if
		the dividend and divisor have the same sign
		otherwise it is negative.

	mod %gpr1, %gpr2			|25|111|rrrr|rrrr|
		Signed integer modulo.
		%gpr1 %= %gpr2;
		the sign of the remainder is the same as
		the sign of the dividend.

	divu %gpr1, %gpr2			|25|100|rrrr|rrrr|
		Unsigned integer division.
		%gpr1 /= %gpr2;

	modu %gpr1, %gpr2			|25|101|rrrr|rrrr|
		Unsigned integer modulo.
		%gpr1 %= %gpr2;


### Bitwise

	and %gpr1, %gpr2			|24|011|rrrr|rrrr|
		%gpr1 &= %gpr2;

	or %gpr1, %gpr2				|24|100|rrrr|rrrr|
		%gpr1 |= %gpr2;

	xor %gpr1, %gpr2			|24|101|rrrr|rrrr|
		%gpr1 ^= %gpr2;

	not %gpr1, %gpr2			|24|110|rrrr|rrrr|
		%gpr1 = ~%gpr2;

	cpy %gpr1, %gpr2			|24|111|rrrr|rrrr|
		%gpr1 = %gpr2;
		"cpy %0, %0" is the official
		NOP instruction.

	sll %gpr1, %gpr2			|24|000|rrrr|rrrr|
		Logical left shift.
		%gpr1 <<= %gpr2[5/4/3:0];

	srl %gpr1, %gpr2			|24|001|rrrr|rrrr|
		Logical right shift.
		%gpr1 >>= %gpr2[5/4/3:0];

	sra %gpr1, %gpr2			|24|010|rrrr|rrrr|
		Arithmetic right shift.
		%gpr1 >>= %gpr2[5/4/3:0];


### Floating point

	fadd %gpr1, %gpr2			|27|000|rrrr|rrrr|
		%gpr1 += %gpr2;

	fsub %gpr1, %gpr2			|27|001|rrrr|rrrr|
		%gpr1 -= %gpr2;

	fmul %gpr1, %gpr2			|27|010|rrrr|rrrr|
		%gpr1 *= %gpr2;

	fdiv %gpr1, %gpr2			|27|011|rrrr|rrrr|
		%gpr1 /= %gpr2;


### Load Immediate

	inc8 %gpr, imm				|1001|iiii|rrrr|iiii|
		%gpr += imm;
		Increment %gpr by the sign extended
		8bits immediate.
		Instruction "inc8 %0, 0" preempts usermode.

	inc16 %gpr1, %gpr2, imm			|20|001|rrrr|rrrr|
						|iiiiiiiiiiiiiiii|
		%gpr1 = %gpr2 + imm;
		Increment %gpr by the sign extended
		16bits little-endian immediate.

	inc32 %gpr1, %gpr2, imm			|20|010|rrrr|rrrr|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		%gpr1 = %gpr2 + imm;
		Increment %gpr by the sign extended
		32bits little-endian immediate.

	inc64 %gpr1, %gpr2, imm			|20|011|rrrr|rrrr|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		%gpr1 = %gpr2 + imm;
		Increment %gpr by the sign extended
		64bits little-endian immediate.

	li8 %gpr, imm				|1000|iiii|rrrr|iiii|
		Set %gpr using the sign extend
		8bits immediate.

	li16 %gpr, imm				|21|001|rrrr|0000|
						|iiiiiiiiiiiiiiii|
		Set %gpr using the sign extended
		16bits little-endian immediate.

	li32 %gpr, imm				|21|010|rrrr|0000|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		Set %gpr using the sign extended
		32bits little-endian immediate.

	li64 %gpr, imm				|21|011|rrrr|0000|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		Set %gpr using the sign extended
		64bits little-endian immediate.


### Test

	seq %gpr1, %gpr2			|23|010|rrrr|rrrr|
		%gpr1 = (%gpr1 == %gpr2);

	sne %gpr1, %gpr2			|23|011|rrrr|rrrr|
		%gpr1 = (%gpr1 != %gpr2);

	slt %gpr1, %gpr2			|23|100|rrrr|rrrr|
		Signed comparison.
		%gpr1 = (%gpr1 < %gpr2);

	slte %gpr1, %gpr2			|23|101|rrrr|rrrr|
		Signed comparison.
		%gpr1 = (%gpr1 <= %gpr2);

	sltu %gpr1, %gpr2			|23|110|rrrr|rrrr|
		Unsigned comparison.
		%gpr1 = (%gpr1 < %gpr2);

	slteu %gpr1, %gpr2			|23|111|rrrr|rrrr|
		Unsigned comparison.
		%gpr1 = (%gpr1 <= %gpr2);

	sgt %gpr1, %gpr2			|22|000|rrrr|rrrr|
		Signed comparison.
		%gpr1 = (%gpr1 > %gpr2);

	sgte %gpr1, %gpr2			|22|001|rrrr|rrrr|
		Signed comparison.
		%gpr1 = (%gpr1 >= %gpr2);

	sgtu %gpr1, %gpr2			|22|010|rrrr|rrrr|
		Unsigned comparison.
		%gpr1 = (%gpr1 > %gpr2);

	sgteu %gpr1, %gpr2			|22|011|rrrr|rrrr|
		Unsigned comparison.
		%gpr1 = (%gpr1 >= %gpr2);


### Branching & Relative Addressing

	jz %gpr1 %gpr2				|26|000|rrrr|rrrr|
		if (!%gpr1) %ip = %gpr2;
		The lsb of %gpr2 is ignored,
		automatically aligning to 16 bits.
		Hence, instruction alignfault
		never occurs.

	jnz %gpr1 %gpr2				|26|001|rrrr|rrrr|
		if (%gpr1) %ip = %gpr2;
		The lsb of %gpr2 is ignored,
		automatically aligning to 16 bits.
		Hence, instruction alignfault
		never occurs.

	jl %gpr1 %gpr2				|26|010|rrrr|rrrr|
		Jump-and-link.
		%gpr1 = ipnext; %ip = %gpr2;
		The lsb of %gpr2 is ignored,
		automatically aligning to 16 bits.
		Hence, instruction alignfault
		never occurs.

	rli8 %gpr, imm				|1110|iiii|rrrr|iiii|
		Relative load-immediate.
		Set %gpr using the sign extend
		8bits immediate added to ipnext.

	rli16 %gpr, imm				|21|101|rrrr|0000|
						|iiiiiiiiiiiiiiii|
		Relative load-immediate.
		Set %gpr using the sign extended
		16bits little-endian immediate
		added to ipnext.

	rli32 %gpr, imm				|21|110|rrrr|0000|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		Relative load-immediate.
		Set %gpr using the sign extended
		32bits little-endian immediate
		added to ipnext.

	rli64 %gpr, imm				|21|111|rrrr|0000|
						|iiiiiiiiiiiiiiii| 2 least-significant-bytes.
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii|
						|iiiiiiiiiiiiiiii| 2 most-significant-bytes.
		Relative load-immediate.
		Set %gpr using the sign extended
		64bits little-endian immediate
		added to ipnext.


### Memory Access

	ld8 %gpr1, %gpr2			|30|100|rrrr|rrrr|
		Load an 8 bits value from
		the memory location in %gpr2, and
		save it in %gpr1, zero extending %gpr1.

	ld16 %gpr1, %gpr2			|30|101|rrrr|rrrr|
		Load a 16 bits value from
		the memory location in %gpr2, and
		save it in %gpr1, zero extending %gpr1.
		In usermode an alignfault occurs
		if the lsb of %gpr2 is non-null, but
		in kernelmode the lsb of %gpr2 is ignored
		automatically aligning to 16 bits.

	ld32 %gpr1, %gpr2			|30|110|rrrr|rrrr|
		Load a 32 bits value from
		the memory location in %gpr2,
		and save it in %gpr1.
		In usermode an alignfault occurs
		if either of the 2 lsb of %gpr2 are non-null,
		but in kernelmode the 2 lsb of %gpr2 are ignored
		automatically aligning to 32 bits.

	ld64 %gpr1, %gpr2			|30|111|rrrr|rrrr|
		Load a 64 bits value from
		the memory location in %gpr2,
		and save it in %gpr1.
		In usermode an alignfault occurs
		if either of the 3 lsb of %gpr2 are non-null,
		but in kernelmode the 3 lsb of %gpr2 are ignored
		automatically aligning to 64 bits.

	st8 %gpr1, %gpr2			|30|000|rrrr|rrrr|
		Store the 8 lsb of %gpr1 to
		the memory location in %gpr2.

	st16 %gpr1, %gpr2			|30|001|rrrr|rrrr|
		Store the 16 lsb of %gpr1 to
		the memory location in %gpr2.
		In usermode an alignfault occurs
		if the lsb of %gpr2 is non-null, but
		in kernelmode the lsb of %gpr2 is ignored
		automatically aligning to 16 bits.

	st32 %gpr1, %gpr2			|30|010|rrrr|rrrr|
		Store the value of %gpr1 to
		the memory location in %gpr2.
		In usermode an alignfault occurs
		if either of the 2 lsb of %gpr2 are non-null,
		but in kernelmode the 2 lsb of %gpr2 are ignored
		automatically aligning to 32 bits.

	st64 %gpr1, %gpr2			|30|011|rrrr|rrrr|
		Store the value of %gpr1 to
		the memory location in %gpr2.
		In usermode an alignfault occurs
		if either of the 3 lsb of %gpr2 are non-null,
		but in kernelmode the 3 lsb of %gpr2 are ignored
		automatically aligning to 64 bits.

	ld8v %gpr1, %gpr2			|14|100|rrrr|rrrr|
		Volatile ld8.
		Bypass and update l1-dcache.

	ld16v %gpr1, %gpr2			|14|101|rrrr|rrrr|
		Volatile ld16.
		Bypass and update l1-dcache.

	ld32v %gpr1, %gpr2			|14|110|rrrr|rrrr|
		Volatile ld32.
		Bypass and update l1-dcache.

	ld64v %gpr1, %gpr2			|14|111|rrrr|rrrr|
		Volatile ld64.
		Bypass and update l1-dcache.

	st8v %gpr1, %gpr2			|14|000|rrrr|rrrr|
		Volatile st8.
		Bypass and update l1-dcache.

	st16v %gpr1, %gpr2			|14|001|rrrr|rrrr|
		Volatile st16.
		Bypass and update l1-dcache.

	st32v %gpr1, %gpr2			|14|010|rrrr|rrrr|
		Volatile st32.
		Bypass and update l1-dcache.

	st64v %gpr1, %gpr2			|14|011|rrrr|rrrr|
		Volatile st64.
		Bypass and update l1-dcache.

	ldst8 %gpr1, %gpr2			|31|000|rrrr|rrrr|
		Atomically swap the 8 bits value
		of the memory location in %gpr2
		with %gpr1, zero extending %gpr1.

	ldst16 %gpr1, %gpr2			|31|001|rrrr|rrrr|
		Atomically swap the 16 bits value
		of the memory location in %gpr2
		with %gpr1, zero extending %gpr1.
		In usermode an alignfault occurs
		if the lsb of %gpr2 is non-null, but
		in kernelmode the lsb of %gpr2 is ignored
		automatically aligning to 16 bits.

	ldst32 %gpr1, %gpr2			|31|010|rrrr|rrrr|
		Atomically swap the 32 bits value
		of the memory location in %gpr2
		with the value of %gpr1.
		In usermode an alignfault occurs
		if either of the 2 lsb of %gpr2 are non-null,
		but in kernelmode the 2 lsb of %gpr2 are ignored
		automatically aligning to 32 bits.

	ldst64 %gpr1, %gpr2			|31|011|rrrr|rrrr|
		Atomically swap the 64 bits value
		of the memory location in %gpr2
		with the value of %gpr1.
		In usermode an alignfault occurs
		if either of the 3 lsb of %gpr2 are non-null,
		but in kernelmode the 3 lsb of %gpr2 are ignored
		automatically aligning to 64 bits.

	cldst8 %gpr1, %gpr2			|31|100|rrrr|rrrr|
		Atomically perform following:
		- Load 8 bits value from memory
		location in %gpr2, zero-extending it.
		- Compare %sr with loaded value on whether equal.
		- If above comparison is true, swap 8 bits value
		of memory location in %gpr2 with %gpr1 value,
		otherwise only save loaded value in %gpr1.

	cldst16 %gpr1, %gpr2			|31|101|rrrr|rrrr|
		Atomically perform following:
		- Load 16 bits value from memory
		location in %gpr2, zero-extending it.
		- Compare %sr with loaded value on whether equal.
		- If above comparison is true, swap 16 bits value
		of memory location in %gpr2 with %gpr1 value,
		otherwise only save loaded value in %gpr1.

	cldst32 %gpr1, %gpr2			|31|110|rrrr|rrrr|
		Atomically perform following:
		- Load 32 bits value from memory
		location in %gpr2.
		- Compare %sr with loaded value on whether equal.
		- If above comparison is true, swap 32 bits value
		of memory location in %gpr2 with %gpr1 value,
		otherwise only save loaded value in %gpr1.

	cldst64 %gpr1, %gpr2			|31|111|rrrr|rrrr|
		Atomically perform following:
		- Load 64 bits value from memory
		location in %gpr2.
		- Compare %sr with loaded value on whether equal.
		- If above comparison is true, swap 64 bits value
		of memory location in %gpr2 with %gpr1 value,
		otherwise only save loaded value in %gpr1.


## Pseudo-Instructions

	nop
		cpy %0, %0;

	inc16 %gpr1, imm
		inc16 %gpr1, %gpr1, imm

	inc32 %gpr1, imm
		inc32 %gpr1, %gpr1, imm

	inc64 %gpr1, imm
		inc64 %gpr1, %gpr1, imm

	preemptctx
		inc8 %0, 0;
		Triggers PreemptIntr when in usermode
		and %flags.disPreemptIntr is not set.

	j %gpr
		jnz %gpr, %gpr;
		Jump to address 0 is done using:
			jz %gpr, %gpr;

	syscall					|0|001|0000|0000|
		System-call instruction.
		When used in kernelmode, it is a hypercall.
		By convention, the system-call number
		must be set in %sr.

	brk					|0|010|0000|0000|
		Software breakpoint instruction.
