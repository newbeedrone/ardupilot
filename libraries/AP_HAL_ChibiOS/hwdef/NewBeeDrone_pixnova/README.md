# NewBeeDrone PixNova

## Introduction

The NewBeeDrone PixNova is an FMUv6X-derived flight controller using an
STM32H753IIK6 processor. This target supports the production PixNova Rev B
sensor configuration.

The firmware board ID is 5900.

## Features

- STM32H753IIK6 processor with 2 MB flash
- three ICM45686 IMUs on independent SPI buses
- internal IST8310 compass and BMP388 barometer on I2C4
- external IST8310 compass on GPS1/I2C1
- external ICP-20100 barometer on I2C2
- 32 KB SPI FRAM parameter storage
- temperature-controlled IMU heater
- STM32F100 PX4IO V2 connected over USART6 at 1.5 Mbit/s
- eight MAIN outputs controlled by the IOMCU
- eight AUX outputs controlled by the FMU
- three external I2C buses and one external SPI bus with two chip selects
- two CAN buses, Ethernet, microSD, and USB

## UART Mapping

The UART mapping follows the FMUv6X convention:

| ArduPilot port | PixNova connector | MCU peripheral |
| --- | --- | --- |
| SERIAL0 | USB | OTG1 |
| SERIAL1 | TELEM1 | UART7 with RTS/CTS |
| SERIAL2 | TELEM2 | UART5 with RTS/CTS |
| SERIAL3 | GPS1 | USART1 |
| SERIAL4 | GPS2 | UART8 |
| SERIAL5 | TELEM3 | USART2 with RTS/CTS |
| SERIAL6 | UART4/EXT2 | UART4 |
| SERIAL7 | FMU DEBUG | USART3 |

USART6 is reserved for communication with the IOMCU and is not exposed as a
general-purpose serial port.

The USART3 debug connector can also be configured as a general-purpose serial
port in ArduPilot.

## Connector and MCU Pin Mapping

The following table records the firmware signal mapping. Refer to the
NewBeeDrone PixNova hardware documentation for connector pin numbering.

| Connector | Signals | MCU pins |
| --- | --- | --- |
| USB | D-, D+, VBUS sense | PA11, PA12, PA9 |
| TELEM1 | TX, RX, RTS, CTS | PE8, PF6, PF8, PE10 |
| TELEM2 | TX, RX, RTS, CTS | PC12, PD2, PC8, PC9 |
| TELEM3 | TX, RX, RTS, CTS | PD5, PA3, PD4, PD3 |
| GPS1 | TX, RX, I2C1 SCL, I2C1 SDA | PB6, PB7, PB8, PB9 |
| GPS2 | TX, RX | PE1, PE0 |
| UART4/EXT2 | TX, RX | PH13, PH14 |
| FMU DEBUG | TX, RX | PD8, PD9 |
| RC input | timer capture input | PI5 |
| CAN1 | RX, TX | PD0, PD1 |
| CAN2 | RX, TX | PB12, PB13 |
| I2C2 | SCL, SDA, ICP-20100 DRDY | PF1, PF0, PG5 |
| I2C3 | SCL, SDA | PA8, PH8 |
| SPI6 | SCK, MISO, MOSI | PB3, PA6, PG14 |
| SPI6 device 1 | CS, DRDY | PI10, PD11 |
| SPI6 device 2 | CS, DRDY | PA15, PD12 |
| ADC | 6.6 V input, 3.3 V input | PC2, PC3 |
| Ethernet RMII | TX/RX, clock, management | PB11, PG13, PG12, PC4, PC5, PA7, PA1, PC1, PA2 |

## I2C Mapping

ArduPilot numbers the internal I2C bus first:

| ArduPilot bus | MCU peripheral | Type | Default devices |
| --- | --- | --- | --- |
| 0 | I2C4 | internal | IST8310 at `0x0c`, BMP388 at `0x76` |
| 1 | I2C1 | external | GPS1 compass at `0x0e`, power monitor 1 |
| 2 | I2C2 | external | ICP-20100 at `0x63`, optional power monitor 2 |
| 3 | I2C3 | external | general-purpose external I2C |

## SPI Mapping

| MCU peripheral | Type | Device |
| --- | --- | --- |
| SPI1 | internal | ICM45686 IMU 1 |
| SPI2 | internal | ICM45686 IMU 2 |
| SPI3 | internal | ICM45686 IMU 3 |
| SPI5 | internal | FRAM |
| SPI6 | external | two external chip selects and data-ready inputs |

## RC Input

The dedicated RC input on PI5 is configured as a timer capture input and can
be used for supported unidirectional receiver protocols. Half-duplex and
bidirectional protocols such as CRSF/ELRS, FPort, and SRXL2 should use a full
UART such as SERIAL6.

Any available UART can be configured for an RC protocol. See the
[ArduPilot RC systems documentation](https://ardupilot.org/copter/docs/common-rc-systems.html)
for the protocol-specific serial settings.

## PWM Output

The PixNova provides 16 outputs: MAIN1 to MAIN8 from the STM32F100 IOMCU and
AUX1 to AUX8 directly from the STM32H753 FMU.

The MAIN outputs use the standard PX4IO V2 pin mapping:

- MAIN1 and MAIN2 use TIM2
- MAIN3 and MAIN4 use TIM4
- MAIN5 through MAIN8 use TIM3

The normal and DShot-capable IOMCU firmware images are embedded in the
ArduPilot firmware. Set `BRD_IO_DSHOT` to `1` and reboot to load the DShot
firmware before configuring a MAIN output for DShot.

The AUX outputs are grouped as follows:

- AUX1 through AUX4 use TIM5
- AUX5 and AUX6 use TIM4
- AUX7 and AUX8 use TIM12

AUX1 through AUX6 support PWM and DShot. Bidirectional DShot input capture is
available on AUX1, AUX3, AUX5, and AUX6. AUX7 and AUX8 do not have update DMA
and support PWM only.

Outputs in the same timer group must use the same output rate and protocol. If
one output in a group uses DShot, all outputs in that group must use DShot.

## GPIOs

PWM outputs can be used as GPIOs after setting the corresponding
`SERVOx_FUNCTION` parameter to `-1`.

The GPIO numbers used by ArduPilot pin parameters are:

- AUX1 through AUX8: 50 through 57
- FMU_CAP1: 58
- NFC GPIO: 60
- SPI6 device 1 DRDY: 93
- SPI6 device 2 DRDY: 94
- MAIN1 through MAIN8: 101 through 108

## Battery and Power Monitoring

The default battery monitor configuration supports INA226, INA228, and INA238
digital power modules connected to power monitor 1:

```text
BATT_MONITOR 21
BATT_I2C_BUS 1
BATT_I2C_ADDR 0
```

For a second digital power module connected to power monitor 2, configure:

```text
BATT2_MONITOR 21
BATT2_I2C_BUS 2
BATT2_I2C_ADDR 0
```

The PixNova IOV2 baseboard uses a 100 kOhm/10 kOhm divider for servo rail
voltage sensing. ArduPilot applies the corresponding 11:1 scale to the IOMCU
servo rail voltage telemetry.

## CAN

The PixNova has two independent CAN interfaces. Both interfaces are enabled by
default on ArduPilot CAN driver 1. They can be assigned to separate logical
drivers with the `CAN_P1_DRIVER` and `CAN_P2_DRIVER` parameters when required.

## Ethernet

Ethernet is enabled by default with DHCP. Network port 1 is configured as a
MAVLink 2 UDP server on port 14550:

```text
NET_ENABLE 1
NET_DHCP 1
NET_P1_TYPE 2
NET_P1_PROTOCOL 2
NET_P1_PORT 14550
```

## Loading Firmware

PixNova firmware uses board ID 5900 and starts at flash address `0x08020000`,
matching the corresponding PX4 target. The alternate RAM map required for
compatibility with the PX4 bootloader is enabled.

The ArduPilot bootloader can load the `NewBeeDrone_pixnova` `.apj` firmware
using an ArduPilot-compatible ground station. It also supports DFU and loading
application firmware from microSD.

The common board ID and application start address provide the compatibility
needed to switch between PX4 and ArduPilot application firmware. Replacing the
bootloader itself is a separate operation and should only be performed with
the correct PixNova bootloader image.

## Further Information

- [PX4 NewBeeDrone PixNova board-support PR](https://github.com/PX4/PX4-Autopilot/pull/26966)
- [Pixhawk FMUv6X Standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-012%20Pixhawk%20Autopilot%20v6X%20Standard.pdf)
- [Pixhawk Autopilot Bus Standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-010%20Pixhawk%20Autopilot%20Bus%20Standard.pdf)
- [Pixhawk Connector Standard](https://github.com/pixhawk/Pixhawk-Standards/blob/master/DS-009%20Pixhawk%20Connector%20Standard.pdf)
