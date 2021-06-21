	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# PerInt (PERipheral INTerconnect) v1 description

	ARCHBITSZ is either 16, 32 or 64.
	ADDRBITSZ == (ARCHBITSZ - clog2(ARCHBITSZ/8)).

## PerInt signals

	Master ports:
		output [2]           op_o
		output [ADDRBITSZ]   addr_o
		output [ARCHBITSZ/8] sel_o
		output [ARCHBITSZ]   data_o
		input  [ARCHBITSZ]   data_i
		input                rdy_i

	Slave ports:
		input  [2]           op_i
		input  [ADDRBITSZ]   addr_i
		input  [ARCHBITSZ/8] sel_i
		input  [ARCHBITSZ]   data_i
		output [ARCHBITSZ]   data_o
		output               rdy_o

## General mode of operation

	A master sets the operation to perform on its outputs
	op_o, addr_o, data_o, sel_o; when its input rdy_i becomes 1,
	the result from the previous operation is set on its input
	data_i and on the next clock edge, the operation sets on
	its outputs op_o, addr_o, data_o, sel_o is started.

	A slave receives the operation to perform on its inputs
	op_i, addr_i, data_i, sel_i; when it has completed the previous
	operation, it sets the result on its output data_o, sets its
	output rdy_o to 1, and on the next clock edge, it starts executing
	the operation set on its inputs op_i, addr_i, data_i, sel_i.

	When a master device has no memory operation to execute,
	it sets its output op_o to PINOOP; when a slave device is not ready
	to start executing the memory operation on its inputs op_i, addr_i,
	data_i, sel_i, it sets its ouput rdy_o to 0.

## Master PerInt signals description

	output [2] op_o
		Memory operation to start executing on
		the next clock edge if the input rdy_i is 1.
		Valid values are:
		PINOOP	0b00: No operation.
		PIWROP	0b01: Write operation.
		PIRDOP	0b10: Read operation.
		PIRWOP	0b11: Atomic read write operation.
		This signal can change, to the value for the next
		operation to perform, while the input rdy_i is 0.

	output [ADDRBITSZ] addr_o
		Physical address which will be used by the memory
		operation set on the output op_o.
		It is to be an ARCHBITSZ bits address for which
		the clog2(ARCHBITSZ/8) least significant bits have
		been discarded.
		This signal can change, to the value for the next
		operation to perform, while the input rdy_i is 0.

	output [ARCHBITSZ/8] sel_o
		The value of this signal is used to select the bytes
		of data_o/data_i on which the memory operation set
		on the output op_o should act upon.
		The least significant bit corresponds to the least
		significant byte, while the most significant bit
		corresponds to the most significant byte.
		Valid values are:
		0b11111111: 64bits at byte offset 0 from addr_o.
		0b00001111: 32bits at byte offset 0 from addr_o.
		0b11110000: 32bits at byte offset 4 from addr_o.
		0b00000011: 16bits at byte offset 0 from addr_o.
		0b00001100: 16bits at byte offset 2 from addr_o.
		0b00110000: 16bits at byte offset 4 from addr_o.
		0b11000000: 16bits at byte offset 6 from addr_o.
		0b00000001: 8bits at byte offset 0 from addr_o.
		0b00000010: 8bits at byte offset 1 from addr_o.
		0b00000100: 8bits at byte offset 2 from addr_o.
		0b00001000: 8bits at byte offset 3 from addr_o.
		0b00010000: 8bits at byte offset 4 from addr_o.
		0b00100000: 8bits at byte offset 5 from addr_o.
		0b01000000: 8bits at byte offset 6 from addr_o.
		0b10000000: 8bits at byte offset 7 from addr_o.
		This signal can change, to the value for the next
		operation to perform, while the input rdy_i is 0.

	output [ARCHBITSZ] data_o
		Data which will be used by PIWROP and PIRWOP.
		This signal can change, to the value for the next
		operation to perform, while the input rdy_i is 0.

	input [ARCHBITSZ] data_i
		Data which will be used by PIRDOP and PIRWOP.
		The value of this signal is valid only when the input rdy_i
		is 1, and must be retrieved immediately on the next clock
		edge after the input rdy_i has become 1.
		Note that when valid, the value of this signal is the result
		of the previous memory operation, not the memory operation
		currently set on the output op_o.
		This signal is a don't-care when the input rdy_i is 0.

	input rdy_i
		When 0, the previously started memory operation
		has not yet completed. When 1, the previously
		started memory operation has completed and the memory
		operation set on the output op_o will start executing
		on the next clock edge; if there was any data expected
		from the just completed memory operation, the value
		of the input data_i must be retrieved immediately
		on the next clock edge.

## Slave PerInt signals description

	input [2] op_i
		Memory operation to start executing on
		the next clock edge if the output rdy_o is 1.
		Valid values are:
		PINOOP		0b00: No operation.
		PIWROP	0b01: Write operation.
		PIRDOP	0b10: Read operation.
		PIRWOP	0b11: Atomic read write operation.
		This signal is a don't-care when the output rdy_o is 0.

	input [ADDRBITSZ] addr_i
		Offset within the slave device, which is to be used
		by the memory operation set on the input op_i.
		It is to be an ARCHBITSZ bits offset for which
		the clog2(ARCHBITSZ/8) least significant bits have
		been discarded.
		This signal is a don't-care when the output rdy_o is 0.

	input [ARCHBITSZ/8] sel_i
		The value of this signal specifies the bytes
		of data_o/data_i on which the memory operation set
		on the input op_i should act upon.
		The least significant bit corresponds to the least
		significant byte, while the most significant bit
		corresponds to the most significant byte.
		Valid values are:
		0b11111111: 64bits at byte offset 0 from addr_i.
		0b00001111: 32bits at byte offset 0 from addr_i.
		0b11110000: 32bits at byte offset 4 from addr_i.
		0b00000011: 16bits at byte offset 0 from addr_i.
		0b00001100: 16bits at byte offset 2 from addr_i.
		0b00110000: 16bits at byte offset 4 from addr_i.
		0b11000000: 16bits at byte offset 6 from addr_i.
		0b00000001: 8bits at byte offset 0 from addr_i.
		0b00000010: 8bits at byte offset 1 from addr_i.
		0b00000100: 8bits at byte offset 2 from addr_i.
		0b00001000: 8bits at byte offset 3 from addr_i.
		0b00010000: 8bits at byte offset 4 from addr_i.
		0b00100000: 8bits at byte offset 5 from addr_i.
		0b01000000: 8bits at byte offset 6 from addr_i.
		0b10000000: 8bits at byte offset 7 from addr_i.
		This signal is a don't-care when the output rdy_o is 0.

	input [ARCHBITSZ] data_i
		Data which is to be used by PIWROP and PIRWOP.
		This signal is a don't-care when the output rdy_o is 0.

	output [ARCHBITSZ] data_o
		Data which is to be used by PIRDOP and PIRWOP.
		The value of this signal is assumed valid only
		for one clock cycle after the output rdy_o has become 1.
		Note that when valid, the value of this signal must
		be the result of the previous memory operation, not
		the memory operation currently set on the input op_i.

	output rdy_o
		When 0, the previously started memory operation
		has not yet completed.
		When 1, the previously started memory operation
		has completed and the memory operation set on
		the input op_i is to start executing on the next
		clock edge.
