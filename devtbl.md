	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# Description of devtbl

	The device-table is a set of entries;
	each entry is two ARCHBITSZ bits which represent
	a single device; the format of each entry is as follow:
	- DevID: ARCHBITSZ bits.
		Unique device identifier.
	- DevMapSz: ADDRBITSZ bits.
		Count of (ARCHBITSZ/8) bytes that the device
		mapping occupies in the address space.
	- Reserved: (clog2(ARCHBITSZ/8)-1) bits; must be 0.
	- DevUseIntr: 1 bit; When 1 the device
		uses interrupt, otherwise it is 0.

	Scanning through the device-table is to
	stop at the entry with DevMapSz null.

	List of currently assigned DevID:
	-1	: Invalid
	0	: Memory gap.
	1	: RAM device.
	2	: DMA engine.
	3	: Interrupt controller.
	4	: Block device.
	5	: Character device.
	6	: GPIO device.
	7	: DevTbl.
	8	: SPI-Master device.
	9	: PDM device.

	devtbl device also implement commands issued through PIRWOP,
	and ARCHBITSZ bits aligned. The ARCHBITSZ bits aligned offset
	within its memory mapping identifies the command.
	The value to write is the argument of the command,
	while the value read is the return value of the command.
	The commands are:
	INFO: Byte offset 0:
		0: SOCVERSION: returns version of the SoC.
		1: RAMCACHESZ: returns size of the RAM cache in (ARCHBITSZ/8) bytes.
		2: BOOTORIGIN: returns from which event the SoC booted:
			0: Hardware CRESET
			1: PWROFF
			2: WRESET
			3: Software CRESET
		3: PRELDRADDR: returns address of pre-loader.
	ACTION: Byte offset (ARCHBITSZ/8):
		0: PWROFF: poweroff SoC; returns 0 on success, otherwise non-null.
		1: WRESET: warm-reset SoC; returns 0 on success, otherwise non-null.
		2: CRESET: cold-reset SoC; returns 0 on success, otherwise non-null.
		3: RRESET: RAMCACHE reset; returns 0 on success; otherwise non-null.
