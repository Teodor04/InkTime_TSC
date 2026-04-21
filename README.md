# InkTime Watch

A low-power e-paper smartwatch built around the **nRF52840**, designed for long battery life, sunlight readability, BLE connectivity, and essential wearable functionality such as notifications, timekeeping, basic motion tracking, and haptic feedback.

The platform integrates a **1.54" e-paper display**, **BLE 5.0**, **USB-C charging**, **battery fuel gauging**, **accelerometer-based activity tracking**, and a **haptic feedback driver**, all inside a compact custom enclosure.

![InkTime Watch Render](Images/Diagrama.png)

---

## Overview

InkTime Watch is a custom smartwatch hardware platform optimized for:
- **Ultra-low power consumption**
- **Always-readable e-paper display**
- **Bluetooth Low Energy communication**
- **Battery-powered operation**
- **Compact wearable form factor**

The system is centered on the **Nordic nRF52840**, which handles BLE communication, USB connectivity, GPIO control, and communication with all peripherals over SPI and I2C.

---

## Block Diagram

```text
                              ┌─────────────────────────────────────────────┐
                              │                 nRF52840                   │
                              │          MCU + BLE + USB Interface         │
                              │                                             │
                              │ SPI ───────────────► E-Paper Display        │
                              │ I2C ───────────────► BMA421                 │
                              │ I2C ───────────────► MAX17048               │
                              │ I2C ───────────────► BQ25180                │
                              │ I2C ───────────────► DRV2605L               │
                              │ GPIO ──────────────► Buttons                │
                              │ GPIO ──────────────► PFET Display Switch    │
                              │ SWDIO / SWDCLK ────► Tag-Connect SWD        │
                              │ D+ / D- ───────────► USB Type-C             │
                              │ ANT ───────────────► 2.4 GHz Antenna        │
                              │ XC1 / XC2 ─────────► 32 MHz Crystal         │
                              │ XL1 / XL2 ─────────► 32.768 kHz Crystal     │
                              └─────────────────────────────────────────────┘
                                              │
                                              ▼
                              ┌─────────────────────────────────────────────┐
                              │                 Power Tree                 │
                              │ USB 5V → BQ25180 → SYS → RT6160 → 3.3V    │
                              │ LiPo → BQ25180 BAT                         │
                              │ LiPo → MAX17048                            │
                              │ 3.3V → PFET → Switched EPD Rail           │
                              └─────────────────────────────────────────────┘


Bill of Materials (BOM)
Active & Critical Components
+----------------------+-------------------------------------------+-----------+------------+----------------------+
| Component            | Description                               | Package   | Part       | Datasheet            |
+----------------------+-------------------------------------------+-----------+------------+----------------------+
| nRF52840             | BLE 5.0 Cortex-M4F MCU                   | AQFN-73   | C190794    | Nordic nRF52840      |
| BQ25180YBGR          | 1A LiPo Charger (I2C)                    | DSBGA-8   | BQ25180YBGR| TI BQ25180           |
| MAX17048G+T10        | Fuel Gauge                              | TDFN-8    | C2682616   | Maxim MAX17048       |
| RT6160AWSC           | 3.3V Buck-Boost                         | WLCSP-15  | C7065276   | Richtek RT6160A      |
| BMA421               | IMU / Step Counter                      | LGA-12    | C5242966   | Bosch BMA421         |
| DRV2605YZFR          | Haptic Driver                           | DSBGA-9   | C527464    | TI DRV2605           |
| 2450AT18B100E        | 2.4GHz Antenna                          | 1206      | C2917717   | Johanson             |
| USBLC6-2SC6Y         | USB ESD Protection                      | SOT-23-6  | C2969755   | ST USBLC6-2          |
| KH-TYPE-C-16P        | USB-C Connector                         | SMD       | C168704    | Generic              |
| 503480-2400          | 24-pin FPC (EPD)                        | SMD       | —          | Molex                |
| DMG2305UX            | P-MOSFET (EPD power)                    | SOT-23    | C2940629   | Diodes Inc           |
| MBR0530              | Schottky Diode                          | SOD-123   | C77336     | Onsemi               |
| TC2030-IDC           | SWD Tag-Connect                         | PCB       | —          | Tag-Connect          |
+----------------------+-------------------------------------------+-----------+------------+----------------------+

Passive Components
+-----------------------------+-----------+--------+-----+-----------------------------+
| Component                   | Value     | Package| Qty | Function                    |
+-----------------------------+-----------+--------+-----+-----------------------------+
| Decoupling capacitors       | 100nF     | 0201   | 5   | MCU + peripherals           |
| Crystal load capacitors     | 12pF      | 0201   | 4   | Crystals                    |
| Bulk capacitors             | 4.7uF     | 0402   | 4   | Power buffering             |
| USB capacitor               | 4.7uF     | 0402   | 1   | USB stability               |
| Power capacitors            | 22uF      | 0402   | 2   | RT6160 input               |
| Charger capacitors          | 1uF       | 0402   | 3   | BQ25180 support            |
| E-paper capacitors          | 1uF / 50V | 0402   | 9   | Display driver             |
| MCU DC/DC inductor          | 10uH      | 0402   | 1   | nRF52840 regulator         |
| Buck-boost inductor         | 0.47uH    | 2012   | 1   | RT6160                     |
| I2C pull-up resistors       | 10k       | 0201   | 2   | SDA / SCL                  |
| USB-C CC resistors          | 5.1k      | 0201   | 2   | USB configuration          |
| Crystal                     | 32 MHz    | 2016   | 1   | MCU / RF clock             |
| Crystal                     | 32.768 kHz| 3215   | 1   | RTC                        |
+-----------------------------+-----------+--------+-----+-----------------------------+

## Hardware Description

### MCU - nRF52840

The **nRF52840** is the central processing unit of the InkTime watch. It integrates:

- **ARM Cortex-M4F** running at **64 MHz**, with floating-point unit
- **Bluetooth Low Energy 5.0**, used for time synchronization, notifications, and wireless communication
- **USB 2.0 Full Speed**, used for charging detection and firmware-related USB functionality
- **1 MB Flash** and **256 KB RAM**

The MCU coordinates all peripherals on the board. Communication with external devices is handled through:

- **SPI** for the e-paper display
- **I2C** for the motion sensor, fuel gauge, charger, regulator, and haptic driver

In addition to the communication buses, the MCU also manages:
- GPIO-based control lines for the display
- interrupt lines from peripherals
- button inputs
- haptic enable control
- SWD programming and debugging

---

### Power Architecture

The power system is designed around a rechargeable **single-cell Li-Po battery** and a **USB-C charging interface**.

```text
USB-C --> BQ25180 (charger / power path) --> Battery charging
                                     --> System rail

Battery --> MAX17048 --> battery monitoring
System rail / regulated rail --> MCU + peripherals
Display high-voltage rails --> generated by dedicated E-paper drive circuitry

BQ25180YBGR

The BQ25180 is the main charger and power-path management IC. It:

charges the Li-Po battery
monitors VBUS
provides system power management
generates interrupt/status events through PMIC_INT
communicates with the MCU over I2C
RT6160AWSC

The RT6160AWSC is used in the board power architecture as a switching regulator that helps generate the required regulated rail for the system.

MAX17048G+T10

The MAX17048 is a battery fuel gauge that monitors the battery voltage and estimates the state of charge (SOC) over I2C, allowing the firmware to display battery level and low-battery warnings.

E-Paper Display

The e-paper display is connected through an FPC connector and communicates with the MCU over SPI, together with several control GPIOs.

Signal	Direction	Description
SCK	MCU → EPD	SPI clock
MOSI	MCU → EPD	SPI data
EPD_CS	MCU → EPD	Chip select
EPD_DC	MCU → EPD	Data / Command select
EPD_RST	MCU → EPD	Hardware reset
EPD_BUSY	EPD → MCU	Busy flag

The display power rail is switched through a dedicated P-channel MOSFET, controlled by the MCU, so the panel can be powered only when needed. This reduces idle leakage and improves battery life.

The display driver circuitry generates the dedicated voltages required by the e-paper panel, such as:

PREVGH
PREVGL
VCOM
other gate/source driving rails required by the display
Motion Sensor - BMA421

The BMA421 is a low-power 3-axis accelerometer connected over I2C. It provides:

motion detection
step counting
orientation and activity-related sensing
interrupt-based wake-up functionality

Two interrupt lines are connected to the MCU:

IMU_INT1 for primary motion/step events
IMU_INT2 for additional event signaling or future extensions

This allows the MCU to remain in a low-power state and wake only when meaningful motion events occur.

Haptic Feedback - DRV2605 + ERM Motor

The DRV2605YZFR haptic driver controls the vibration motor used for user feedback. It connects to the MCU over I2C and also uses a dedicated digital enable signal:

HAPTIC_EN enables or disables the driver
OUT+ / OUT- drive the ERM motor
the chip supports waveform-based playback using its internal effects library

The haptic subsystem is used for:

notifications
alerts
button feedback
warning indications
USB Interface - KH-TYPE-C-16P + USBLC6-2SC6Y

The board uses a USB Type-C connector for charging and USB connectivity.

Main USB-related functions:

battery charging through VBUS
USB 2.0 D+ / D- lines connected to the nRF52840
ESD protection on USB data lines using USBLC6-2SC6Y
USB attach / power detection using the VBUS signal

The USB D+ / D- lines connect directly to the nRF52840 USB transceiver, while the USB-C configuration network ensures proper device operation.

RF Section - 2450AT18B100E Antenna

The board uses a 2.4 GHz chip antenna for BLE communication.

Key layout requirements for the RF section:

the antenna is placed at the PCB edge
copper is removed from the antenna keepout area
no signal traces are routed underneath the antenna
an impedance matching network is placed between the MCU RF pin and the antenna

This section is critical for BLE performance and must follow the recommended RF layout constraints.

SWD Debug - TC2030-IDC

Programming and debugging are performed through a Tag-Connect TC2030-IDC footprint.

The debug interface includes:

SWDIO
SWDCLK
nRESET
3.3V reference
GND

Additional exposed test pads are available for easier bring-up and debugging:

TP_SWDIO
TP_SWDCLK
TP_RESET
TP_3.3V
TP_GND

This allows programming and debugging without a permanently mounted header.

nRF52840 Pinout Specification
Shared I2C Bus

All I2C peripherals share the same bus, with pull-up resistors to 3.3V.

Pin	Function	Connected Devices
P0.06	SDA	BMA421, MAX17048, BQ25180, RT6160, DRV2605
P0.07	SCL	BMA421, MAX17048, BQ25180, RT6160, DRV2605
SPI Bus - E-Paper Display
Pin	Function	Description
P0.02	SCK	SPI clock for e-paper display
P0.03	MOSI	SPI data to e-paper display
P0.05	CS	E-paper chip select
P0.15	DC	Data / Command select
P0.16	RST	Display hardware reset
P0.17	BUSY	Display busy/status input
Display Power Control
Pin	Function	Description
P1.01	PFET Gate Control	Enables/disables the display power rail
Interrupts and Peripheral Control
Pin	Function	Description
P0.08	IMU_INT1	Primary motion / step wake interrupt
P1.08	IMU_INT2	Secondary motion interrupt
P0.10	ALRT	Fuel gauge alert / low battery interrupt
P0.11	PMIC_INT	Charger interrupt / status signal
P0.12	HAPTIC_EN	Enables the haptic driver
Physical Buttons

All buttons are connected as active-low inputs to GND.

Pin	Function
P0.13	Up Button
P0.14	Down Button
P1.00	Enter / Esc Button
Clocks and Debug
Pin / Signal	Function
XC1 / XC2	32 MHz crystal
XL1 / XL2	32.768 kHz crystal
SWDIO	SWD debug data
SWDCLK	SWD debug clock
nRESET	MCU reset
VBUS	USB power / attach detection
D+ / D-	Native USB 2.0
ANT	RF output to matching network and antenna
Pin Assignment Rationale

The pin assignment was chosen to keep routing compact and logically grouped:

I2C on P0.06 / P0.07 allows all low-speed peripherals to share a common bus.
SPI on P0.02 / P0.03 / P0.05 keeps display routing compact toward the FPC connector.
Display control pins are grouped close to the SPI bus to simplify routing.
Interrupt lines are placed on dedicated GPIOs so the MCU can identify wake sources immediately.
Buttons are assigned to GPIOs that can be routed toward the lower PCB edge, where the user interface is located.
HAPTIC_EN is placed near the I2C-related section for compact routing.
PFET gate control is placed on a dedicated GPIO to isolate display power switching from the communication buses.
SWDIO / SWDCLK / RESET are broken out to the Tag-Connect footprint for reliable programming and debug access.
Design Notes
The RF section must be kept clear of copper and routed carefully to preserve BLE performance.
The e-paper display requires dedicated high-voltage driving rails generated by the display power circuitry.
The charger, regulator, and battery monitoring ICs form the core of the power subsystem.
The design uses both communication buses and dedicated GPIO control to balance flexibility, low power, and routing simplicity.

License

This project is intended for educational use unless otherwise specified.
