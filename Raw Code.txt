; Author : Matthew Romleski
; Tech ID: 12676184
; Program that tests a 16-bit integer by pushing it into a subroutine that:
; 1.) splits the decimal into two equal halves (in this case, 12 and 33).
; 2.) Calculate the squares of these two halves.
; 3.) Add these squares together.
; 4.) If the result is the same as the original number (1233) then return 1. Else return 0.

		.include <atxmega128a1udef.inc>

		.dseg
		
		.def	rtrnVal = r25

		.def	resLow	= r6
		.def	resHi	= r7
		.def	numLow	= r16
		.def	numHi	= r17
		.def	hundLow	= r18
		.def	hundHi	= r19
		.def	temp	= r20

		.def	tempLow	= r0
		.def	tempHi	= r1
		.def	remLow	= r21
		.def	remHi	= r22
		.def	qLow	= r23
		.def	qHi		= r24

		.cseg
		.org	0x00
		rjmp	start
		.org	0xF6


start:	ldi		numLow, 0xD1 ; Initializes the number we'll be testing (0x04D1, or 1233D).
		ldi		numHi, 0x04 ; ^^
		ldi		hundLow, 100 ; Loads 100, which we'll be using as the divisor.
		ldi		hundHi, 0 ; ^^
		ldi		temp, low(RAMEND) ; Initializes the stack pointer.
		out		CPU_SPL, temp ; ^^
		ldi		temp, high(RAMEND) ; ^^
		out		CPU_SPH, temp ; ^^

		call	tester ; Returns its result in r25 (rtrnVal).

done:	rjmp	done



tester:	push	numLow ; Stores variables in the stack.
		push	numHi ; ^^
		push	hundLow ; ^^
		push	hundHi ; ^^
		
		call	div16U ; Result stored in r23 (qLow), remainder in r21 (remLow). Q=12, R=33
		
		pop		hundHi ; Pops variables from the stack.
		pop		hundLow ; ^^
		pop		numHi ; ^^
		pop		numLow ; ^^
		
		mul		qLow, qLow ; Calculates 12^2.
		mov		resLow, tempLow ; Moves the result of the multiplication.
		mov		resHi, tempHi ; ^^
		mul		remLow, remLow ; Calculates 33^2.
		add		resLow, tempLow ; Adds 12^2 & 33^2.
		adc		resHi, tempHi ; ^^
		
		cp		resLow, numLow ; Compares the low bits of the result with the low bits of the original number.
		brne	rtrn0
		cp		resHi, numHi ; ^^ high bits ^^ high bits of the original number.
		brne	rtrn0

rtrn1:	ldi		rtrnVal, 1 ; Only does this is resLow=numLow and resHi=numHi.
		jmp		return

rtrn0:	ldi		rtrnVal, 0 ; Does this is resLow=/=numLow or resHi=/=numHi.

return:	ret



div16U:	ldi		temp, 0 ; Resets the loop counter.
		mov		qLow, numLow ; Sets numLow:numHi as the dividend.
		mov		qHi, numHi ; ^^
		clr		remLow ; Sets remainder registers to zero.
		clr		remHi ; ^^
divlp:	lsl		qLow ; Shifts left, stores MSB in carry.
		rol		qHi ; Rotates carry into lowest, MSB out to carry.
		rol		remLow ; ^^
		rol		remHi ; ^^
		mov		tempLow, remLow ; Saves the remainder to a temp registry.
		mov		tempHi, remHi ; ^^
		sub		tempLow, hundLow ; Subtracts our divisor from the remainder.
		sbc		tempHi, hundHi ; ^^
		brmi	negtve ; Branches if negative.
		ori		qLow, 0x01 ; Sets 0 bit in the Quotient to 1 (if result of SBC was positive).
		mov		remLow, tempLow ; Updates the remainder.
		mov		remHi, tempHi ; ^^
		jmp		incrmt ; 
negtve: andi	qLow, 0xFE ; Sets 0 bit in Quotient to 0 (if result of SBC was negative).
incrmt:	inc		temp ; Increments the loop counter.
		cpi		temp, 16 ; Sees if the subroutine has looped 16 times.
		brne	divlp ; If it hasn't, loops again.
		ret		; Otherwise it, exits from the subroutine.