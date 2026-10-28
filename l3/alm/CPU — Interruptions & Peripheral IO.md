A CPU can be divided into two parts:

|Component|Role|
|---|---|
|**CU — Control Unit**|Decides which micro-operations to execute and in which order|
|**DP — Datapath**|Performs calculations and transfers between registers and memory|

### Internal registers

|Register|Role|
|---|---|
|`PC`|Program Counter — address of the next instruction|
|`IR` / `Rinst`|Instruction Register — current instruction|
|`MA`|Memory Address Register — address placed on the memory address bus|
|`MD`|Memory Data Register — data read from or written to memory|
|`CPSR`|Current Processor Status Register — processor flags and state|
|`LR`|Link Register — function return address|
|`SPSR`|Saved Processor Status Register — saved processor state|
|`SLR`|Saved Link Register — saved interrupt return address in the simplified CPU|

### Memory access

Memory accesses pass through `MA` and `MD` in the simplified CPU.

**Read:**

```
MA ← address
MD ← Mem[MA]
```

**Write:**

```
MA ← address
MD ← value
Mem[MA] ← MD
```

Example: `ldr r0, [r1, #4]`

```
MA ← R1 + 4
MD ← Mem[MA]
R0 ← MD
```

---

## Interrupt Handling

### IRQ — Interrupt Request

- The current instruction finishes execution.
- The CPU checks whether an IRQ is pending and enabled.
- If accepted, the CPU executes the interrupt-entry sequence.
- Otherwise, it continues with the next Fetch.

### FIQ — Fast Interrupt Request

On classic ARM (AArch32):

- The current instruction finishes execution.
- The CPU checks whether a FIQ is pending and enabled.
- If accepted, the CPU saves the required state and switches to FIQ mode.
- The CPU branches to the FIQ handler.

FIQ has additional banked registers (`R8–R14`), reducing the need to save registers to memory.

### CPSR — Current Processor Status Register

|Flag|Meaning|
|---|---|
|`N`|Negative|
|`Z`|Zero|
|`C`|Carry|
|`V`|Overflow|
|`I`|IRQ mask|
|`F`|FIQ mask|

**Interrupt masks:**

- `0` → Interrupt enabled
    
- `1` → Interrupt masked

**Conditions:**

```
IRQ && !CPSR.I
FIQ && !CPSR.F
```

### Interrupt entry

Simplified CPU:

```
SLR       ← PC
SPSR      ← CPSR
CPSR.I    ← 1
PC        ← IRQ_VECTOR
```

- `SLR` saves the return address.
- `SPSR` saves the processor flags and interrupt mask.
- `I = 1` masks further IRQs.
- `PC` is redirected to the interrupt handler.

### Interrupt return

```
CPSR ← SPSR
PC   ← SLR
```

The handler must also preserve any general-purpose registers it modifies.

### Instructions

|Instruction|Definition|
|---|---|
|`accept`|Enable IRQ (`I = 0`)|
|`ignore`|Disable IRQ (`I = 1`)|
|`rti`|Return from Interrupt — restore `CPSR` and `PC`|

These are instructions of the simplified CPU used in the course.

---

## Interrupt Vector Table

The vector table contains the addresses of exception handlers.

### Cortex-M4

|Offset|Handler|
|---|---|
|`0x00`|Initial SP|
|`0x04`|Reset|
|`0x08`|NMI|
|`0x0C`|HardFault|
|`0x3C`|SysTick|
|`0x40`|First external IRQ|

```
Vector entry = Vector table base + Offset
```

On Cortex-M, bit 0 of a handler address stored in the vector table indicates Thumb state and must be `1`.

---

## Cortex-M4 Exception Handling

Unlike the simplified CPU, Cortex-M4 automatically saves part of the context on the stack.

### Exception entry

```
Push R0–R3, R12, LR, PC, xPSR
LR ← EXC_RETURN
PC ← Handler address
```

`EXC_RETURN` is a special value used to trigger an exception return.

### Exception return

```
bx lr
```

When `LR` contains `EXC_RETURN`, the CPU restores the stacked context and resumes the interrupted execution.

`R4–R11` must be preserved by software if the handler modifies them.

### NVIC — Nested Vectored Interrupt Controller

- Enables or disables external IRQ lines.
- Manages interrupt priorities.
- Tracks pending interrupts.
- Allows higher-priority exceptions to preempt lower-priority handlers.