# CR31 Power Distribution Module (PDM)

The Power Distribution Module is a new board for CR31 that is designed to split GLV into multiple outputs to provide power for various components on the car.

Each output is monitored to ensure voltage, current, and power are within expected ranges; if any output exceeds its configured threshold, the PDM will disable it and log an error message over CAN.

## Important Components

- **STM32F415RGT6** - Handles the main logic for PDM (runs the code from this repository)
    - Also provides auxiliary **GPIO** for additional input/output needs
- **[INA238-Q1](https://www.ti.com/lit/ds/symlink/ina238-q1.pdf) Voltage/Current Sensors (8 total)** - Monitor voltage, current, and power for each output; communicate over **I2C**
    - I2C Addresses: `0x80`, `0x81`, `0x83`, `0x84`, `0x85`, `0x87`, `0x8C`, `0x8D`
- **[BTS7004-1EPP](https://www.infineon.com/assets/row/public/documents/10/49/infineon-bts7004-1epp-datasheet-en.pdf) High-Side Drivers (8 total)** - Control power from GLV; each controlled with one **GPIO** pin
- **[TCAN1042-Q1](https://www.ti.com/lit/ds/symlink/tcan1042hv-q1.pdf) CAN Transceiver** - Allows communication over **CAN**, primarily for error logging

## CAN Frames (Subject to Change)

### Output Exceeded Threshold

This is the error message that is logged when an output has been disabled after exceeding its configured thresholds.  It should be sent only once, at the moment the output is disabled.

<details>
    <summary>Message Format (7 Bytes)</summary>

```
   0 1 2 3 4 5 6 7 8 9 A B C D E F
  +-+-+-+-+-+-+-+-+
0 | Output  |V|C|P|
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
1 | Voltage                       |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
3 | Current                       |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
5 | Power                         |
  +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

#### Output (5 bits)

The index of the output that was disabled.  This should currently be within the range `0`-`7`, though the message allows up to index `31` in case the PDM is ever expanded.

#### VCP Flags (3 bits)

Marks which values exceeded their threshold (**V**oltage, **C**urrent, or **P**ower).  Should be `1 (true)` if the value exceeded the threshold, or `0 (false)` if it was within the expected range.

#### Voltage (16 bits)

The voltage at the moment before the output was disabled.  It is stored as an unsigned fixed-point integer, where `1 bit` = `0.003125 V`.

#### Current (16 bits)

The current at the moment before the output was disabled.  It is stored as an unsigned fixed-point integer, where `1 bit` = `0.0002 A`.

#### Power (16 bits)

The Power at the moment before the output was disabled.  It is stored as an unsigned fixed-point integer, where `1 bit` = `0.001 W`.

</details>
