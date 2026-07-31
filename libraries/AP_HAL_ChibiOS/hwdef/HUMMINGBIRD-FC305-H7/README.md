# NewBeeDrone HUMMINGBIRD FC305 H7 Flight Controller

The NewBeeDrone HUMMINGBIRD FC305 H7 is a flight controller based on the
STM32H743 microcontroller.

## Features

- STM32H743 microcontroller running at 480 MHz
- ICM42688P IMU
- DPS310 barometer
- MAX7456 analog OSD
- microSD card for logging
- 8 UARTs
- 8 motor outputs with bidirectional DShot support
- 2 servo outputs
- 1 NeoPixel LED output
- Battery voltage and current sensing
- Analog RSSI input
- USB
- Active-low buzzer output
- 2 status LEDs
- 3 GPIO outputs

The board does not have onboard dataflash or a compass.

## UART Mapping

| ArduPilot port | Hardware port | TX pin | RX pin | Default function |
| --- | --- | --- | --- | --- |
| SERIAL0 | OTG1 | - | - | MAVLink2 over USB |
| SERIAL1 | USART1 | PB6 | PB7 | ESC Telemetry |
| SERIAL2 | USART2 | PD5 | PD6 | None |
| SERIAL3 | USART3 | PD8 | PD9 | None |
| SERIAL4 | UART4 | PD1 | PD0 | None |
| SERIAL5 | UART5 | PB13 | PB12 | RC Input |
| SERIAL6 | USART6 | PC6 | PC7 | None |
| SERIAL7 | UART7 | PE8 | PE7 | None |
| SERIAL8 | UART8 | PE1 | PE0 | None |

USART1 and USART2 are connected to ESC telemetry signals. ArduPilot uses one
serial ESC telemetry instance, so USART1 is enabled for ESC telemetry by
default and USART2 is left unassigned.

## RC Input

The default RC input is configured on SERIAL5 using UART5. It supports serial
receiver protocols such as CRSF and ELRS. UART5 RX has a dedicated DMA stream,
and UART5 TX is DMA enabled for receiver telemetry.

## PWM Output

The HUMMINGBIRD FC305 H7 supports up to 11 PWM outputs.

| Output | MCU pin | Timer | Bidirectional DShot | Typical use |
| ---: | --- | --- | --- | --- |
| 1 | PA3 | TIM5_CH4 | Yes | Motor 1 |
| 2 | PA2 | TIM5_CH3 | Yes | Motor 2 |
| 3 | PA1 | TIM5_CH2 | Yes | Motor 3 |
| 4 | PA0 | TIM5_CH1 | Yes | Motor 4 |
| 5 | PD12 | TIM4_CH1 | Yes | Motor 5 |
| 6 | PD13 | TIM4_CH2 | Yes | Motor 6 |
| 7 | PD14 | TIM4_CH3 | Yes | Motor 7 |
| 8 | PD15 | TIM4_CH4 | Yes | Motor 8 |
| 9 | PB14 | TIM12_CH1 | No | Servo output |
| 10 | PB15 | TIM12_CH2 | No | Servo output |
| 11 | PE5 | TIM15_CH1 | No | NeoPixel LED |

Outputs are organized into timer groups. All outputs in the same group must
use the same output protocol and rate:

- Group 1: PWM1-PWM4 (TIM5), supporting PWM, DShot, and bidirectional DShot
- Group 2: PWM5-PWM8 (TIM4), supporting PWM, DShot, and bidirectional DShot
- Group 3: PWM9-PWM10 (TIM12), supporting standard PWM without DMA
- Group 4: PWM11 (TIM15), configured as a NeoPixel LED output by default

The motor ordering follows the Betaflight/X layout (`FRAME_TYPE = 12`).

## Buzzer

The buzzer output is connected to PA15 and is active low.

## OSD Support

The board has an onboard MAX7456 analog OSD connected to SPI4. Analog OSD is
enabled by default (`OSD_TYPE = 1`).

## microSD Card

The microSD card uses the four-bit SDMMC1 interface and is enabled for
filesystem logging. The SDMMC data and command pins are configured with
internal pull-ups.

## Battery Monitoring

The board has onboard battery voltage and current sensing. The default
parameters are:

| Parameter | Value |
| --- | ---: |
| `BATT_MONITOR` | 4 |
| `BATT_VOLT_PIN` | 10 |
| `BATT_CURR_PIN` | 11 |
| `BATT_VOLT_MULT` | 11.0 |
| `BATT_AMP_PERVLT` | 12.5 |

The voltage sensing circuit uses a 100k/10k resistor divider. Battery voltage
and current scaling should be calibrated against measured values before use.

## RSSI

Analog RSSI is available on ADC pin 8.

## Compass

The board does not have a built-in compass. An external compass can be
connected to I2C1.

## Barometer

The onboard DPS310 barometer is connected to I2C2. Both supported DPS310 I2C
addresses are probed.

## GPIOs

| Function | MCU pin | GPIO | Default state |
| --- | --- | ---: | --- |
| Buzzer | PA15 | 80 | High (off) |
| PINIO1 | PC15 | 81 | High (inactive) |
| PINIO2 | PC14 | 82 | High (inactive) |
| PINIO3 | PC13 | 83 | High (inactive) |
| LED0 | PE3 | 90 | Low |
| LED1 | PE4 | 91 | Low |

PINIO1, PINIO2, and PINIO3 are inverted and start in their inactive state.

## SPI3

The SPI3 SCK, MISO, and MOSI pins are defined, but no onboard SPI3 device or
dataflash is configured.

## Loading Firmware

Initial firmware load can be done with DFU by connecting USB while holding the
bootloader button. Load the `with_bl.hex` firmware using a DFU loading tool.

Once the initial firmware is loaded, firmware can be updated using any
ArduPilot ground station software. Updates should use the `*.apj` firmware
file.
