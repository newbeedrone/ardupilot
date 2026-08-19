# NewBeeDrone INFINITY FC305 H7 Flight Controller

The NewBeeDrone INFINITY FC305 H7 is a flight controller based on the
STM32H743 microcontroller.

## Features

- STM32H743 microcontroller running at 480 MHz
- ICM42688P IMU
- BMP390 barometer
- AT7456E analog OSD
- microSD card for logging
- 8 UARTs
- CAN bus
- 8 motor outputs with bidirectional DShot support
- 2 servo outputs
- 1 NeoPixel LED output
- Dual battery voltage and current sensing
- External analog and analog RSSI inputs
- USB
- Passive beeper output
- 2 status LEDs
- 3 board control GPIOs
- External SPI expansion bus

The board does not have onboard dataflash or a compass.

## IMU Orientation

The ICM42688P is mounted in the same orientation as on the NewBeeDrone
HUMMINGBIRD FC305 H7. The hardware definition applies a 180-degree roll
rotation (`ROTATION_ROLL_180`).

The IMU CLKIN connection is available on PE9. ArduPilot leaves PE9 as an input
and uses the ICM42688P internal clock.

## UART Mapping

| ArduPilot port | Hardware port | TX pin | RX pin | Default function |
| --- | --- | --- | --- | --- |
| SERIAL0 | OTG1 | - | - | MAVLink2 over USB |
| SERIAL1 | USART1 | PB6 | PB7 | RC Input |
| SERIAL2 | USART2 | PD5 | PD6 | None |
| SERIAL3 | USART3 | PD8 | PD9 | None |
| SERIAL4 | UART4 | PD1 | PD0 | None |
| SERIAL5 | UART5 | PB13 | PB12 | ESC Telemetry |
| SERIAL6 | USART6 | PC6 | PC7 | None |
| SERIAL7 | UART7 | PE8 | PE7 | MSP DisplayPort |
| SERIAL8 | UART8 | PE1 | PE0 | GPS |

USART1 is the bidirectional receiver/CRSF port. UART4 is connected to the
SBUS/expansion port and can be configured as an alternative RC input. USART3
and UART5 are connected to the second and first ESC telemetry returns,
respectively; UART5 is enabled for ESC telemetry by default.

## PWM Output

The INFINITY FC305 H7 supports up to 11 PWM outputs.

| Output | MCU pin | Timer | Bidirectional DShot | Typical use |
| ---: | --- | --- | --- | --- |
| 1 | PA0 | TIM5_CH1 | Yes | Motor 1 |
| 2 | PA1 | TIM5_CH2 | Yes | Motor 2 |
| 3 | PA2 | TIM5_CH3 | Yes | Motor 3 |
| 4 | PA3 | TIM5_CH4 | Yes | Motor 4 |
| 5 | PD12 | TIM4_CH1 | Yes | Motor 5 |
| 6 | PD13 | TIM4_CH2 | Yes | Motor 6 |
| 7 | PD14 | TIM4_CH3 | Yes | Motor 7 |
| 8 | PD15 | TIM4_CH4 | Yes | Motor 8 |
| 9 | PB14 | TIM12_CH1 | No | Servo output |
| 10 | PB15 | TIM12_CH2 | No | Servo output |
| 11 | PE5 | TIM15_CH1 | No | NeoPixel LED |

Outputs in the same timer group must use the same output protocol and rate:

- Group 1: PWM1-PWM4 (TIM5), supporting PWM, DShot, and bidirectional DShot
- Group 2: PWM5-PWM8 (TIM4), supporting PWM, DShot, and bidirectional DShot
- Group 3: PWM9-PWM10 (TIM12), supporting standard PWM without DMA
- Group 4: PWM11 (TIM15), configured as a NeoPixel LED output by default

The motor ordering follows the Betaflight/X layout (`FRAME_TYPE = 12`).

## OSD Support

The onboard AT7456E analog OSD is connected to SPI3. Analog OSD is enabled by
default (`OSD_TYPE = 1`). UART7 is configured for MSP DisplayPort for a digital
video system.

## microSD Card

The microSD card uses the four-bit SDMMC1 interface and is enabled for
filesystem logging. The board does not have a card-detect input.

## Battery Monitoring

The board provides two analog battery voltage and current sensing pairs.

| Parameter | Battery 1 | Battery 2 |
| --- | ---: | ---: |
| Monitor type | Analog voltage and current | Analog voltage and current |
| Voltage pin | 10 | 5 |
| Current pin | 11 | 9 |
| Voltage multiplier | 11.0 | 11.0 |
| Amps per volt | 12.5 | 12.5 |

Both voltage inputs use a 100k/10k divider. The current scaling depends on the
connected ESC current sensor and must be calibrated before use.

## I2C and Compass

The onboard BMP390 barometer and the external compass connector share I2C2.
The board does not have a built-in compass.

## CAN

CAN1 is available on PB8 (RX) and PB9 (TX).

## GPIOs

| Function | MCU pin | GPIO | Startup state |
| --- | --- | ---: | --- |
| Buzzer | PA15 | 80 | High (off) |
| 12V rail enable | PC15 | 81 | High (enabled) |
| Camera switch | PC14 | 82 | High (camera 1) |
| PINIO3 | PC13 | 83 | High |
| LED0 | PE3 | 90 | Low |
| LED1 | PE4 | 91 | Low |

## Loading Firmware

Initial firmware load can be done with DFU by connecting USB while holding the
bootloader button. Load the `with_bl.hex` firmware using a DFU loading tool.

Once the initial firmware is loaded, firmware can be updated using any
ArduPilot ground station software. Updates should use the `*.apj` firmware
file.
