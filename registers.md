	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# Description of PUxx registers

	%0 thru %15: 16/32/64 bits general purpose registers.
		They are also referred as %gpr.
		kernelmode and usermode each have their own
		set of dedicated gpr.
		The following system instructions are used to copy
		gpr values from/to kernelmode to/from usermode:
		setkgpr %gpr1 %gpr2:
			Set the kernelmode gpr indexed
			by %gpr1, with the value of the
			usermode gpr indexed by %gpr2.
		setugpr %gpr1 %gpr2:
			Set the usermode gpr indexed
			by %gpr1, with the value of the
			kernelmode gpr indexed by %gpr2.

	%ip: 16/32/64 bits instruction pointer register.
		Its value is the address of
		the instruction being decoded.
		Its value can be read using
		the instruction jl or rli.

	The following registers are accessible
	only when the pu is in kernelmode.

	%ksl: Write only. 16/32/64 bits wide.
		KernelSpaceLimit value.
		When running in kernelspace, a 1-to-1 mapping
		is always done regardless of TLB entries if
		the memory access address is >= 0x1000 and < %ksl .
		When running in userspace, The TLB is never
		ignored and this register is ignored.
		When the TLB is not used, memory accesses within
		the range >= 0x1000 and < %ksl are cached and
		un-cached when outside of the same memory range.

	%sysopcode: Read/Write. 16bits wide.
		When an interrupt occurs, this register
		get set with the instruction opcode
		at which the interrupt was triggered.
		The opcode is stored in such a way
		that the first byte of the instruction
		occupy the least significant byte
		of this register.
		opcode OPNOTAVAIL (|8|000|0000|0000|) can be
		returned for ExtIntr, TimerIntr, ExecFaultIntr.
		Its value is read using the instruction:
		getsysopcode %gpr			|5|000|rrrr|0000|
			The value of %sysopcode is
			written in the 16lsb of %gpr.

	%faultreason: Read only. 3bits wide.
		When an interrupt occurs,
		the pu set this register with the reason.
		The following describe the meaning of each value:
		0: ReadFaultIntr.
		1: WriteFaultIntr.
		2: ExecFaultIntr.
		3: AlignFaultIntr.
		4: ExtIntr.
		5: SysOpIntr.
		6: TimerIntr.
		7: PreemptIntr.
		Its value is read using the instruction:
		getfaultreason %gpr			|5|011|rrrr|0000|
			Write the value of %faultreason
			in the 3lsb of %gpr.

	%faultaddr: Read only. 16/32/64 bits wide.
		When a fault occur, the pu set this register
		with the offending 16/32/64 bits address.
		Its value is read using the instruction:
		getfaultaddr %gpr			|5|010|rrrr|0000|
			Write the value of
			%faultaddr in %gpr.

	%ksysopfaulthdlr: Write only. 16/32/64 bits wide.
		When a ksysopfault fault occurs, the pu jumps to
		the address in this register (adjusted on whether
		from kernelmode/usermode) and set %sysopcode with
		the instruction opcode that triggered the fault.
		If ksysopfault occurred in usermode, the pu also
		switches to kernelmode and set %faultaddr with the
		physical address of the address in usermode:%gpr2.
		The adjusted address the pu jumps to is computed as follow:
		- From usermode: %ksysopfaulthdlr
		- From kernelmode: (%ksysopfaulthdlr + 4)
		Its value is written using following instruction:
		setksysopfaulthdlr %gpr			|7|000|rrrr|0000|
			%ksysopfaulthdlr = %gpr;

	%clkcyclecnt: Read only. 32/64/128 bits wide.
		Counter that increment every clock cycle
		when the pu rst input is not active.
		Its value is read using the instructions:
		getclkcyclecnt %gpr			|5|100|rrrr|0000|
			%gpr = %clkcyclecnt[15/31/63:0];
		getclkcyclecnth %gpr			|2|001|rrrr|0000|
			%gpr = %clkcyclecnt[31/63/127:16/32/64];

	%timer: Write only. 16/32/64 bits wide.
		Holds timer value.
		settimer %gpr				|7|111|rrrr|0000|
			%timer = %gpr;

	%asid: Write only. 13bits wide.
		12lsb holds ASID (Adresss Space ID),
		with bit12 indicating whether usermode is running
		in kernelspace (when 0) or userspace (when 1).
		setasid %gpr				|7|100|rrrr|0000|
			%asid = %gpr;

	%flags: Write only. 16 bits wide.
		Hold flags that determine which kernelmode
		capabilities are allowed in usermode:
		0x0001: setasid
		0x0002: settimer
		0x0004: settlb
		0x0008: clrtlb
		0x0010: getclkcyclecnt
		0x0020: getclkfreq getcap getver
		0x0040: gettlbsize
		0x0080: getcachesize
		0x0100: getcoreid
		0x0200: cacherst
		0x0400: gettlb
		0x0800: setflags
		0x1000: disExtIntr
		0x2000: disTimerIntr
		0x4000: disPreemptIntr
		0x8000: halt
		Its value is written using the instruction:
		setflags %gpr				|7|110|rrrr|0000|
			%flags = %gpr;

	%tlb: Write only.
		Contains TLB (Translation Lookaside Buffer) entries.
		The instruction gettlbsize returns the number of TLB entries.
		Each TLB entry is set using the instruction settlb.
