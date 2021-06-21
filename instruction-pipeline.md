	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# Instruction Pipeline (Single Issue)

	IF  : Instruction Fetch
	ID  : Instruction Decode
	EX  : Execute
	MEM : Memory Access
	WB  : Register Write Back

	IF | ID   EX    WB | : Pipeline with single-cycle instructions; ID EX WB are same stage.
	IF | ID | EX  | WB | : Pipeline with multi-cycle instructions.
	IF | ID | MEM | WB | : Pipeline with memory-access instructions.

# Notes

	The icache and itlb are used at the IF stage.
	The dcache and dtlb are used at the MEM stage.
