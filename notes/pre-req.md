For the test:

# K&R 2nd Ed — exact sections:

1. Bit manipulation (AND/OR/XOR/shifts): §2.9
2. Pointers basics: §5.1, 5.2
3. Pointer arithmetic: §5.3, 5.4
4. Structs: §6.1, 6.2, 6.4
4. sizeof: §5.3 (comes up naturally)
5. Typedef: §6.7

# K&R won’t cover:

uint8_t / uint32_t — C99 stdint.h, not in K&R. Just know they’re fixed-width integer types
volatile — barely touched in K&R. Just know: tells compiler don’t optimize this variable away, used for hardware registers

### Priority order: §2.9 → §5.1-5.4 → §6.1-6.4

# MCU basics (30 min max)

What a microcontroller is vs a microprocessor (MCU has CPU + memory + peripherals on one chip)

GPIO — what it is, input/output pins

What a register is in hardware context (memory-mapped, you write to an address to control hardware)

Interrupts — what they are conceptually (hardware signals CPU to stop and handle something)

UART — serial communication, TX/RX, used for debugging embedded systems

Flash vs RAM on an MCU — program stored in flash, runs from RAM

What a bootloader does

# Hardware rudimentary

Pull-up / pull-down resistors — concept only

Clock speed and why it matters

SPI, I2C — just know they’re communication protocols between chips

That’s genuinely it. Don’t go deeper than one-liner understanding on each. He said rudimentary — he means it.

K&R §2.9, §5.1-5.4 + this list = you’re prepared. Go.
