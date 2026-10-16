# eCTF Selection Test — Study Guide

---

## PART 1: C FUNDAMENTALS

### Bit Manipulation (K&R §2.9)

Operators:
- `&` — AND: both bits must be 1. Use: masking/clearing bits
- `|` — OR: either bit is 1. Use: setting bits
- `^` — XOR: bits differ. Use: toggling bits, basic encryption
- `~` — NOT: flip all bits
- `<<` — left shift (multiply by 2)
- `>>` — right shift (divide by 2)

Examples:
```c
x = x & 0xFF;        // keep only lowest 8 bits (mask)
x = x | (1 << 3);    // set bit 3
x = x & ~(1 << 3);   // clear bit 3
x = x ^ (1 << 3);    // toggle bit 3
x = x << 2;          // multiply x by 4
x = x >> 1;          // divide x by 2
```

XOR properties — know these cold:
- `a ^ a = 0`
- `a ^ 0 = a`
- XOR is reversible: if `c = a ^ b`, then `a = c ^ b`
- Basic encryption: `ciphertext = plaintext ^ key`, `plaintext = ciphertext ^ key`
- XOR swap (no temp variable):
```c
a = a ^ b;
b = a ^ b;
a = a ^ b;
```

---

### Pointers (K&R §5.1, 5.2)

```c
int x = 5;
int *p = &x;   // p holds the address of x
*p = 10;       // dereference: changes x to 10
```

- `&` — address-of operator: gives you the memory address of a variable
- `*` — dereference operator: gives you the value at that address
- Pointer type matters: `int *p` means p points to an int (4 bytes)

---

### Pointer Arithmetic (K&R §5.3, 5.4)

```c
int arr[] = {10, 20, 30};
int *p = arr;   // p points to arr[0]
p++;            // p now points to arr[1] — moves by sizeof(int) bytes, not by 1 byte
```

Key rule: pointer arithmetic scales by the size of the type.
- `p + 1` on an `int *` moves 4 bytes forward
- `p + 1` on a `char *` moves 1 byte forward

sizeof:
```c
sizeof(int)     // typically 4 bytes
sizeof(char)    // always 1 byte
sizeof(arr)     // total bytes in array
sizeof(arr) / sizeof(arr[0])  // number of elements
```

---

### Structs (K&R §6.1, 6.2, 6.4)

```c
struct Point {
    int x;
    int y;
};

struct Point p1;
p1.x = 3;
p1.y = 4;

// Pointer to struct — use arrow operator
struct Point *ptr = &p1;
ptr->x = 10;   // same as (*ptr).x = 10
```

---

### typedef (K&R §6.7)

Creates an alias for a type — cleaner, more portable code.

```c
typedef unsigned char uint8_t;
typedef unsigned int  uint32_t;

// Without typedef:
struct GPIO_Config { uint8_t pin; uint8_t mode; };
struct GPIO_Config led;

// With typedef:
typedef struct {
    uint8_t pin;
    uint8_t mode;
} GPIO_Config;

GPIO_Config led;   // cleaner
```

---

### uint8_t / uint32_t (C99 stdint.h — not in K&R)

Fixed-width integer types. Critical in embedded — you need to know exact sizes.

```c
#include <stdint.h>

uint8_t  a;   // exactly 8 bits  (0–255)
uint16_t b;   // exactly 16 bits
uint32_t c;   // exactly 32 bits
int8_t   d;   // signed 8 bits   (-128 to 127)
```

Why: `int` size varies by platform (2 or 4 bytes). On an MCU you can't afford surprises.

---

### volatile

```c
volatile uint32_t *reg = (volatile uint32_t *)0x40021000;
```

Tells the compiler: **do not optimize this variable away**. Read/write it every single time.

Why it matters in embedded: hardware registers change without the CPU doing anything (external events, peripherals). Without `volatile`, the compiler might cache the value in a register and never re-read it — your code breaks silently.

Rule: always use `volatile` for memory-mapped hardware registers.

---

## PART 2: MCU BASICS

### Microcontroller vs Microprocessor

**Microprocessor (CPU only):**
- Just the compute core
- Needs external chips: RAM, Flash, peripherals
- Example: Intel x86, ARM Cortex-A (your laptop/phone)

**Microcontroller (CPU + everything on one chip):**
- CPU + RAM + Flash + GPIO + UART + timers + ADC — all integrated
- Self-contained, cheap, low power
- Example: STM32, Arduino ATmega, ESP32
- Designed for embedded control, not general computing

---

### GPIO (General Purpose Input/Output)

Physical pins on an MCU you configure in software.

**Output mode:**
- MCU drives the pin — actively sets it HIGH (3.3V) or LOW (0V)
- Two types:
  - Push-pull: actively drives both HIGH and LOW (most common)
  - Open-drain: can only pull LOW or float; needs external pull-up for HIGH (used in I2C)
- Use: LED control, sending signals to other chips

**Input mode:**
- MCU reads the pin — external circuit controls the voltage
- Three types:
  - Floating: nothing connected — reads garbage. Never use.
  - Pull-up: internal resistor to VCC. Default HIGH, goes LOW when button pressed to GND.
  - Pull-down: internal resistor to GND. Default LOW, goes HIGH when button pressed to VCC.
- Use: reading buttons, sensors

Key rule: never leave input pins floating. Always configure pull-up or pull-down.

**"Driving" a pin** means actively supplying current to set a definite voltage. Opposite of floating (uncontrolled).

---

### Hardware Registers (Memory-mapped)

A register is a special memory address that controls hardware directly.

```c
// Write to a hardware register at a fixed address
volatile uint32_t *GPIOA_ODR = (volatile uint32_t *)0x40020014;
*GPIOA_ODR = 0x01;   // set pin 0 HIGH
```

- Each peripheral (GPIO, UART, timers) has registers at fixed addresses
- Write to the address → hardware responds immediately
- Must use `volatile` — hardware changes register values without CPU involvement

---

### Interrupts

**Concept:** Hardware signals the CPU to stop what it's doing, save its state, run a special function (ISR — Interrupt Service Routine), then return.

Examples:
- Button pressed → interrupt fires → CPU handles it immediately
- Timer expires → interrupt fires
- UART receives data → interrupt fires

Without interrupts: CPU would have to constantly poll ("is button pressed? is button pressed?") — wastes cycles.

With interrupts: CPU does other work, hardware taps it on the shoulder only when needed.

---

### UART (Universal Asynchronous Receiver Transmitter)

Serial communication protocol. Two wires:
- TX (transmit)
- RX (receive)

One device's TX connects to the other's RX, and vice versa.

Key parameters: baud rate (bits per second) — both sides must agree (e.g. 115200 baud).

Main use in embedded: **debugging**. Connect MCU's UART to a PC's USB-serial adapter, print debug messages. The `printf` of the embedded world.

---

### Flash vs RAM on an MCU

**Flash (non-volatile):**
- Stores your program (firmware) permanently
- Survives power-off
- Slow to write, fast to read
- Typical size: 256KB – 2MB on hobbyist MCUs

**RAM (volatile):**
- Stores variables, stack, heap at runtime
- Lost on power-off
- Fast read and write
- Typical size: 64KB – 512KB

At startup: bootloader copies data from Flash to RAM. Code executes from Flash (or RAM on some MCUs).

---

### Bootloader

Program that runs first when the MCU powers on, before your main firmware.

Responsibilities:
1. Basic hardware init
2. Verify firmware integrity (in secure systems — checks signature/hash)
3. Load and jump to main application
4. Sometimes: allow firmware updates over UART/USB

In eCTF context: **bootloader security is critical**. A compromised bootloader = game over. This is where a lot of eCTF attacks target.

---

## PART 3: HARDWARE BASICS

### Pull-up / Pull-down Resistors

**Pull-up:** Resistor between pin and VCC. Default state = HIGH. Button press pulls it to GND → LOW.

**Pull-down:** Resistor between pin and GND. Default state = LOW. Button press connects to VCC → HIGH.

Why: floating pins pick up electrical noise and read random values. Resistors guarantee a defined default state.

Most modern MCUs have internal pull-up/down resistors you enable in software — no external component needed.

---

### Clock Speed

MCU runs on a clock — a signal oscillating at fixed frequency (e.g. 72 MHz).

- Every instruction takes 1+ clock cycles
- Higher clock = faster execution, more power consumption
- Peripherals (UART, SPI, I2C) derive their speed from the system clock
- Clock misconfiguration = wrong baud rate = broken communication

---

### SPI (Serial Peripheral Interface)

4-wire synchronous protocol:
- MOSI (Master Out Slave In)
- MISO (Master In Slave Out)
- SCK (clock)
- CS/SS (chip select — which slave to talk to)

Fast, full-duplex. Used for: flash memory, displays, sensors.

---

### I2C (Inter-Integrated Circuit)

2-wire protocol:
- SDA (data)
- SCL (clock)

Multiple devices on same 2 wires, each with a unique address. Slower than SPI. Uses open-drain + pull-up resistors. Used for: sensors, EEPROMs, small peripherals.

---

## QUICK REFERENCE — Things to Know Cold

| Concept | One-liner |
|---|---|
| XOR encryption | `ciphertext = plaintext ^ key` |
| XOR self-inverse | `a ^ a = 0`, `a ^ 0 = a` |
| volatile | Don't optimize; re-read hardware register every time |
| uint8_t | Fixed 8-bit type — use for registers, buffers |
| Floating input | Never. Always pull-up or pull-down. |
| UART | Serial debug interface. TX↔RX. |
| Bootloader | First code to run. Loads firmware. Security-critical. |
| Flash | Stores program. Non-volatile. |
| RAM | Runtime variables. Lost on power-off. |
| GPIO output | MCU drives voltage HIGH/LOW |
| GPIO input | MCU reads external voltage |
| Interrupt | Hardware asynchronously signals CPU |
| Pull-up | Default HIGH, button pulls LOW |
| Pull-down | Default LOW, button pulls HIGH |

---

*Good luck. Be there before 3:30pm.*