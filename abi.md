	SPDX-License-Identifier: GPL-2.0-only
	(c) William Fonkou Tambe

# PUxx ABI

## Registers convention

	| Register | ABI Name |    Description     | Saver  |
	| -------- | -------- | ------------------ | ------ |
	|    %0    |   %sp    | Stack Pointer      | Callee |
	|    %1    |   %1     | Func Arg / Ret Val | Caller |
	|    %2    |   %2     | Function Argument  | Caller |
	|    %3    |   %3     | Function Argument  | Caller |
	|    %4    |   %4     | Function Argument  | Caller |
	|    %5    |   %5     | Function Argument  | Caller |
	|    %6    |   %6     | Function Argument  | Caller |
	|    %7    |   %7     | Function Argument  | Caller |
	|    %8    |   %8     |                    | Caller |
	|    %9    |   %9     |                    | Caller |
	|    %10   |   %tp    | Task Pointer       | Caller |
	|    %11   |   %11    | Struct Value       | Caller |
	|    %12   |   %12    | Static Chain       | Caller |
	|    %13   |   %sr    | Scratch Register   |        |
	|    %14   |   %fp    | Frame Pointer      | Callee |
	|    %15   |   %rp    | Return Pointer     | Callee |
	|          |   %ap    | Arguments Pointer  |        |


## Calling convention

	The stack grows toward address 0 and is aligned to sizeof(void*).
	Scalar arguments are passed by value while aggregate arguments
	are passed by reference; for example, a 64bits scalar argument
	will use 2 registers, with the 32 lower bits stored in the lower
	indexed register.
	Arguments are passed in registers %1 thru %7, while arguments which
	cannot find enough registers and vargars are passed on the stack
	at the address pointed by %ap.
	Arguments passed on the stack are aligned to sizeof(void*).
	Related GCC macros: TARGET_PASS_BY_REFERENCE, TARGET_FUNCTION_ARG.

	Stack after function's prologue:

	%sp ->+-----------------------+           Low address
	      |                       | \
	      |  Function Arguments   |  | Outgoing Arguments
	      |                       | /
	      +-----------------------+
	      |                       |
	      |   Local Variables     |
	      |                       |
	%fp ->+-----------------------+
	      |  Previous Frame Ptr   |
	      +-----------------------+
	      |    Return Address     |                Callee
	%ap ->+-----------------------+----------------------
	      |                       |                Caller
	      |  Function Arguments   |
	      |                       |
	      +-----------------------+
	      |                       |
	      |   Local Variables     |
	      |                       |
	      +-----------------------+          High Address


## Function value

	Function value is returned in %1 if its size is less-than-or-equal
	to sizeof(void*), otherwise it is returned in memory, and the address
	of that memory location is passed in %11.
	Related GCC macros: TARGET_RETURN_IN_MEMORY, TARGET_STRUCT_VALUE_RTX,
	FUNCTION_VALUE_REGNO_P, TARGET_FUNCTION_VALUE, TARGET_LIBCALL_VALUE.
