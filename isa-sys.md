	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# PUxx System Instructions

	sysret					|0|000|0000|0000|
		Switch to usermode.

	halt					|0|011|0000|0000|
		Halt execution until a timer
		or external interrupt occurs.
		Actual halting where execution
		stop happens only in usermode.

	icacherst				|0|100|0000|0000|
		Invalidate the instruction cache.

	dcacherst				|0|101|0000|0000|
		Invalidate the data cache.

	ksysret					|0|111|0000|0000|
		Return from ksysopfault.

	setksysopfaulthdlr %gpr			|7|000|rrrr|0000|
		%ksysopfaulthdlr = %gpr;
		The lsb of %gpr is ignored,
		automatically aligning to 16 bits
		the address set in %ksysopfaulthdlr.

	setksl %gpr				|7|001|rrrr|0000|
		%ksl = %gpr;

	settlb %gpr1, %gpr2			|7|010|rrrr|rrrr|
		Set a TLB entry.
		Bit format expected from the value
		of %gpr1 and %gpr2 respectively:
		|ppn: 4/20/52|xxx: 7|user: 1|cached: 1|readable: 1|writable: 1|executable: 1|
		|vpn: 4/20/52|asid: 12|
		The index of the TLB entry being set come from
		the least-significant-bits of the field "vpn";
		the total count of those bits is the clog2()
		of the number of TLB entries as returned
		by the instruction gettlbsize.
		The field "ppn" is the physical page number to which
			the virtual-page-number correspond to.
		The field "xxx" are don't-care-bits.
		The field "user" determine whether the page can be used in userspace,
			where bit12 of %asid[%curctx] indicates whether running
			in kernelspace (when 0) or userspace (when 1).
		The field "cached" determine whether memory access
			to the page is cached.
		The fields "readable", "writable" and "executable" are
			respectively the read, write and executable permissions
			of the virtual page.
		The field "vpn" is the virtual-page-number.
		The field "asid" is the adresss-space-id which allow
			multiple threads to share the TLB.

	clrtlb %gpr1, %gpr2			|7|011|rrrr|rrrr|
		Clear a TLB entry matching the value in %gpr2.
		Bit format expected from the value of %gpr2:
		|vpn: 4/20/52|asid: 12|
		The value of %gpr1 is a mask of bits to match;
		ie: when %gpr1 is null, the TLB entry is cleared
		regardless of whether it matches the value in %gpr2.

	setasid %gpr				|7|100|rrrr|0000|
		%asid = %gpr;
		Bit12 indicates whether usermode is running
		in kernelspace (when 0) or userspace (when 1).
		%sr is expected to have the address
		of the Page-Global-Directory.

	setuip %gpr				|7|101|rrrr|0000|
		%uip = %gpr;

	setflags %gpr				|7|110|rrrr|0000|
		%flags = %gpr;

	settimer %gpr				|7|111|rrrr|0000|
		%timer = %gpr;

	setkgpr %gpr1 %gpr2			|15|001|rrrr|rrrr|
		%gpr1 = usermode:%gpr2;

	setugpr %gpr1 %gpr2			|15|010|rrrr|rrrr|
		usermode:%gpr1 = %gpr2;

	setgpr %gpr1 %gpr2			|15|011|rrrr|rrrr|
		usermode:%gpr1 = usermode:%gpr2;

	getsysopcode %gpr			|5|000|rrrr|0000|
		The value of %sysopcode is
		written in the 16lsb of %gpr.

	getuip %gpr				|5|001|rrrr|0000|
		%gpr = usermode:%ip;

	getfaultaddr %gpr			|5|010|rrrr|0000|
		Write the value of
		%faultaddr in %gpr.

	getfaultreason %gpr			|5|011|rrrr|0000|
		Write the value of %faultreason
		in the 3lsb of %gpr.

	getclkcyclecnt %gpr			|5|100|rrrr|0000|
		%gpr = %clkcyclecnt[15/31/63:0];

	getclkcyclecnth %gpr			|5|101|rrrr|0000|
		%gpr = %clkcyclecnt[32/63/127:16/32/64];

	gettlbsize %gpr				|5|110|rrrr|0000|
		Return in %gpr a powerof2
		value that is the number
		of TLB entries.

	geticachesize %gpr			|5|111|rrrr|0000|
		Return in %gpr, the size
		of the instruction cache.
		The value returned must be multiplied
		by (ARCHBITSZ/8) to be the actual
		instruction cache size in bytes.

	getcoreid %gpr				|2|000|rrrr|0000|
		Set %gpr using the core-id.

	getclkfreq %gpr				|2|001|rrrr|0000|
		Return in %gpr, the pu
		clock frequency in Hz.

	getdcachesize %gpr			|2|010|rrrr|0000|
		Return in %gpr, the size
		of the data cache.
		The value returned must be multiplied
		by (ARCHBITSZ/8) to be the actual
		data cache size in bytes.

	gettlb %gpr1, %gpr2			|2|011|rrrr|rrrr|
		Lookup a TLB entry using %gpr2
		and return the result in %gpr1.
		Null is returned in %gpr1 if there is not a match.
		Bit format expected from the value of %gpr2:
		|vpn: 4/20/52|asid: 12|
		The index of the TLB entry returned comes from
		the least-significant-bits of the field "vpn";
		Bit format of the value returned in %gpr1 when there is match:
		|ppn: 4/20/52|xxx: 7|user: 1|cached: 1|readable: 1|writable: 1|executable: 1|
