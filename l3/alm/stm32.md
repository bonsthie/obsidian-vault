[]()
Each GPIO port is separated by `0x400` (1024) bytes.

GPIOA = `0x40020000`
GPIOB = `0x40020400`
GPIOC = `0x40020800`


```c
register_address = GPIOx_BASE + register_offset;
```
# GPIO port

- `MODER` (32-bit) -> configures what each pin does
- `IDR` (32-bit, lower 16 useful) -> reads the physical value of each pin
- `ODR` (32-bit, lower 16 useful) -> stores/drives output values
-  (32-bit) -> atomically set/reset output pins

## MODER

2 bits per pin:

```
00 = input
01 = output
10 = alternate function
11 = analog
```

For pin `n`:

```
MODER[2n+1 : 2n]
```

## BSRR

Atomically set/reset selected output pins.

```
bits 0-15  -> set GPIO pin to 1
bits 16-31 -> reset GPIO pin to 0
```

Example:

```c
GPIOD_BSRR = (1 << 12);         // PD12 = 1
GPIOD_BSRR = (1 << (12 + 16));  // PD12 = 0
```

Both at once:

```c
GPIOD_BSRR =
    (1 << 12) |
    (1 << (14 + 16)); // PD12 = 1, PD14 = 0
```

This:

```c
GPIOD_BSRR = (1 << 12);
```

corresponds logically to:

```c
uint32_t value = GPIOD_ODR;
value |= (1 << 12);
GPIOD_ODR = value;

```

but `BSRR` does it with **one atomic write**. ALM_cours3_26

# Level vs edge

```c
int get_pin(void) {
    return (GPIOA_IDR >> 5) & 1u;
}
```

## Level = state now

```c
if (get_pin() == 1) {
    // pin is currently high
}
```

## Edge = state change

Rising edge `0 -> 1`:

```c
while (get_pin() != 0) { } // wait until low
while (get_pin() == 0) { } // wait until high
```

Falling edge `1 -> 0`:

```c
while (get_pin() == 0) { } // wait until high
while (get_pin() != 0) { } // wait until low
```


# TODO 

- Timers and SysTick
- DAC and hardware RNG
- SPI vs I2C