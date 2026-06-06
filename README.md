# ⚡ LV-LINE-FAULT-DETECTION-SYSTEM

> A real-time embedded fault detection system for low-voltage AC overhead conductors using STM32, multi-sensor fusion, GSM SIM800L, and RS485 — with instant SMS and call alerts to maintenance authorities.

---

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [System Architecture](#system-architecture)
- [Hardware Components](#hardware-components)
- [Software Tools](#software-tools)
- [Circuit & Pin Connections](#circuit--pin-connections)
- [How It Works](#how-it-works)
- [Detection Logic](#detection-logic)
- [GSM Alert via AT Commands](#gsm-alert-via-at-commands)
- [Results](#results)
- [Future Scope](#future-scope)

---

## Overview

Low-voltage AC distribution lines in rural and urban areas are vulnerable to conductor breakage caused by wind, lightning, falling tree branches, and severe weather. Traditional fault detection relies on manual patrolling — a process that is slow, unsafe, and impractical at scale. Existing protective devices like fuses and relays only trigger under high fault currents, making them ineffective for conductor breakage where fault current is near zero.

This project solves that with an **automated, real-time conductor breakage detection system** built on embedded multi-sensor fusion. The STM32F103C8T6 microcontroller continuously monitors current, voltage, and vibration sensors across the distribution line. When abnormal conditions are detected — such as current drop near zero, voltage imbalance, or mechanical disturbance — the system immediately isolates the faulty section via relay, triggers LED indicators, and dispatches structured SMS and voice call alerts to the electricity board and maintenance linemen via the GSM SIM800L module — **all without cloud dependency or human intervention**.

---

## Features

- ✅ Real-time conductor breakage detection using sensor fusion (current + voltage + vibration)
- ✅ STM32F103C8T6 (ARM Cortex-M3) as the central processing unit
- ✅ Immediate fault isolation via 5V relay module
- ✅ SMS and voice call alerts via GSM SIM800L with fault type, pole ID, and GPS location link
- ✅ Local fault indication using LED indicators and 16×2 I2C LCD display
- ✅ RS485 communication for multi-node scalability across large distribution networks
- ✅ Self-powered from the distribution line itself via step-down transformer + LM2596 buck converter
- ✅ Detects faults even at near-zero fault currents — where conventional fuses and relays fail
- ✅ Suitable for rural and urban low-voltage distribution networks
- ✅ Published research paper (IJSDR, ISSN: 2455-2631)

---

## System Architecture

The system is divided into three functional layers:

```
┌─────────────────────────────────────────────────────────────────┐
│  SENSING LAYER       ACS712 Current Sensor (PA0)                │
│                      Voltage Divider Sensor (PA1)               │
│                      SW-420 Vibration Sensor (GPIO)             │
├─────────────────────────────────────────────────────────────────┤
│  PROCESSING LAYER    STM32F103C8T6 (ARM Cortex-M3 @ 72 MHz)     │
│                      Sensor Fusion Logic (HAL Library)          │
│                      ADC, GPIO, UART peripheral control         │
├─────────────────────────────────────────────────────────────────┤
│  COMMUNICATION &     GSM SIM800L (AT Commands via UART)         │
│  OUTPUT LAYER        SMS + Voice Call → EB Office & Linemen     │
│                      5V Relay → Fault Isolation                 │
│                      16×2 I2C LCD → Real-time Status Display    │
│                      RS485 Transceiver → Multi-node expansion   │
└─────────────────────────────────────────────────────────────────┘
```

### Block Diagram

```
Low Voltage AC Line
        │
        ▼
Step-Down Transformer ──► LM2596 Buck Converter ──► DC Power Supply
                                                           │
                                                           ▼
Current Sensor (ACS712) ──┐                        STM32F103C8T6
Voltage Sensor ───────────┼──────────────────────► Main MCU
Vibration Sensor (SW-420)─┘                               │
                                               ┌──────────┼──────────┐
                                               ▼          ▼          ▼
                                          5V Relay    LCD Display   GSM SIM800L
                                          (Fault      (Real-time    (SMS + Call)
                                          Isolation)   Status)           │
                                               │                         ▼
                                         Current          Electricity Board Office
                                         Cut Off          Maintenance Linemen

                          RS485 Transceiver ◄──── Multi-node Expansion / Smart Grid
```

---

## Hardware Components

| Component | Model | Interface | Purpose |
|---|---|---|---|
| Microcontroller | STM32F103C8T6 | — | Central processing, sensor reading, fault logic |
| Current Sensor | ACS712 | ADC (PA0) | Measures line current using Hall Effect |
| Voltage Sensor | Voltage Divider Module | ADC (PA1) | Monitors line voltage level |
| Vibration Sensor | SW-420 | GPIO (PA3) | Detects mechanical disturbances / wire snapping |
| GSM Module | SIM800L | UART (PA9/PA10) | SMS and voice call alerts to authorities |
| Relay Module | 5V Relay | GPIO (PA2) | Fault isolation — cuts power to faulty section |
| Display | 16×2 I2C LCD | I2C (PB6/PB7) | Real-time status and fault display |
| Communication | RS485 MAX485 | UART | Multi-node long-distance communication |
| LED Indicator | Red/Green LED | GPIO | Local visual fault indication |
| Power Regulation | LM2596 Buck Converter | — | Regulated DC from AC line via transformer |
| Transformer | Step-Down Transformer | — | Safely steps down AC line voltage |

---

## Software Tools

| Tool | Purpose |
|---|---|
| **STM32CubeMX** | Microcontroller pin configuration and peripheral initialization |
| **STM32CubeIDE** | Firmware development, compilation, and debugging |
| **STM32CubeProgrammer** | Flashing compiled firmware to STM32 via ST-Link (SWD) |
| **STM32 HAL Library** | Hardware abstraction for ADC, GPIO, UART, I2C peripherals |

### STM32 HAL Functions Used

| Function | Purpose |
|---|---|
| `HAL_ADC_Start()` / `HAL_ADC_GetValue()` | Read analog sensor data |
| `HAL_GPIO_ReadPin()` | Read vibration sensor digital input |
| `HAL_GPIO_WritePin()` | Control relay, LED outputs |
| `HAL_UART_Transmit()` | Send AT commands to GSM SIM800L |
| `HAL_I2C_Master_Transmit()` | Drive 16×2 LCD display |

---

## Circuit & Pin Connections

### Current Sensor (ACS712) → STM32

| ACS712 Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 5V | Power supply |
| GND | GND | Ground |
| OUT | PA0 (ADC1_IN0) | Analog current output |
| IP+ | Supply line | Current input from source |
| IP− | Load side | Current output to load |

### Voltage Sensor → STM32

| Voltage Sensor Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 5V / 3.3V | Power supply |
| GND | GND | Ground |
| OUT | PA1 (ADC1_IN1) | Scaled analog voltage output |

### Vibration Sensor (SW-420) → STM32

| SW-420 Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 3.3V / 5V | Power supply |
| GND | GND | Ground |
| DO | PA3 (GPIO_Input) | Digital HIGH/LOW vibration output |

### GSM SIM800L → STM32 (UART)

| SIM800L Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 3.7–4.2V (external) | Power (requires dedicated supply) |
| GND | GND | Ground |
| TXD | PA10 (USART1_RX) | GSM data to STM32 |
| RXD | PA9 (USART1_TX) | Commands from STM32 |

### 5V Relay Module → STM32

| Relay Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 5V | Power supply |
| GND | GND | Ground |
| IN | PA2 (GPIO_Output) | Control signal from STM32 |
| COM | Power line input | Common terminal |
| NO | Load / line output | Normally Open — connects on fault |

### 16×2 I2C LCD → STM32

| LCD Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 5V | Power supply |
| GND | GND | Ground |
| SDA | PB7 (I2C1_SDA) | I2C data line |
| SCL | PB6 (I2C1_SCL) | I2C clock line |

### RS485 (MAX485) → STM32

| RS485 Pin | STM32 Pin | Description |
|---|---|---|
| VCC | 5V / 3.3V | Power supply |
| GND | GND | Ground |
| RO | STM32 RX | Receiver output to MCU |
| DI | STM32 TX | Driver input from MCU |
| DE | GPIO pin | Driver Enable (HIGH = Transmit) |
| RE | GPIO pin (tied with DE) | Receiver Enable (LOW = Receive) |

### Power Supply Chain

```
Low Voltage AC Line → Step-Down Transformer → LM2596 Buck Converter → STM32 + Sensors + Modules
```

---

## How It Works

1. **Boot** — STM32 initializes ADC, GPIO, UART, I2C, LCD, and GSM module.
2. **Continuous Monitoring** — Current (ACS712) and voltage sensors feed analog signals into the STM32 ADC. Vibration sensor feeds a digital GPIO input.
3. **Sensor Fusion** — STM32 processes multi-parameter data: current drop, voltage imbalance, and vibration state are analyzed together to reduce false alarms.
4. **Fault Detection** — A fault is declared when:
   - Current ≈ 0 (conductor breakage)
   - Voltage is abnormal (imbalance / undervoltage)
   - Vibration is detected (mechanical disturbance)
5. **Fault Isolation** — STM32 triggers the 5V relay via GPIO to immediately disconnect the faulty section.
6. **Local Alert** — Red LED activates at fault location; LCD displays fault status and sensor readings.
7. **GSM Alert** — STM32 sends AT commands to SIM800L dispatching:
   - Structured SMS with fault type, pole ID, GPS location link, and status
   - Voice call to EB office and maintenance linemen
8. **Continuous Loop** — System resumes monitoring for new fault conditions.

---

## Detection Logic

```c
// Read ADC values from current and voltage sensors
uint32_t send_vol = Read_ADC(ADC_CHANNEL_6);
uint32_t recv_vol = Read_ADC(ADC_CHANNEL_8);

int send_ok = (send_vol > THRESHOLD);
int recv_ok  = (recv_vol  > THRESHOLD);

// Read vibration sensor digital input
uint8_t tilt       = HAL_GPIO_ReadPin(GPIOB, GPIO_PIN_14);
int     tilt_fault = (tilt == 0);

// Fault condition: both ends fail + vibration detected → Line Breakage
if (!send_ok && !recv_ok && tilt_fault) {
    HAL_GPIO_WritePin(GPIOB, GPIO_PIN_4, GPIO_PIN_SET); // Relay ON
    HAL_GPIO_WritePin(GPIOA, GPIO_PIN_11, GPIO_PIN_SET); // LED ON
    GSM_SendSMS(num1, fault_msg);
    GSM_Call(num1);
}

// Fault condition: TX OK, RX fail + vibration → Mid-span breakage
else if (send_ok && !recv_ok && tilt_fault) { ... }

// Fault condition: TX OK, RX fail, no vibration → Voltage fluctuation
else if (send_ok && !recv_ok && !tilt_fault) { ... }
```

### ADC to Physical Value Conversion

```c
// Current calculation (ACS712)
float voltage_out = (adc_val * VREF) / ADC_RES;
float current     = (voltage_out - CURRENT_OFFSET) / SENSITIVITY * CAL_FACTOR;

// Voltage calculation (Voltage Divider)
float actual_voltage = voltage_out * VOLTAGE_SCALE;
```

---

## GSM Alert via AT Commands

```
AT                                        // Check communication
AT+CMGF=1                                 // Set SMS text mode
AT+CMGS="+91XXXXXXXXXX"                   // Set recipient number
> FAULT DETECTED!                         // Message body
  TYPE: LINE BREAKAGE
  LED: ON
  POLE ID: POLE 01
  LOCATION: https://maps.app.goo.gl/...
  STATUS: LINE FAILURE
[CTRL+Z]                                  // Send

ATD+91XXXXXXXXXX;                         // Voice call to authority
[10 second call duration]
ATH                                       // Hang up
```

### Fault Conditions and Alerts

| Condition | Fault Type | GSM Message |
|---|---|---|
| TX fail + RX fail + Vibration | LINE BREAKAGE | Pole 01 — both ends down with mechanical disturbance |
| TX OK + RX fail + Vibration | LINE BREAKAGE | Pole 02 — mid-span breakage detected |
| TX OK + RX fail + No Vibration | VOLTAGE FLUCTUATION | Pole 02 — electrical fault without mechanical event |

---

## Results

The system was successfully designed, implemented, and tested under simulated fault conditions. Key outcomes:

- The STM32-based multi-sensor system correctly detected **complete breakage, partial disconnection, and mechanical disturbances** in all test scenarios.
- On conductor breakage simulation, current dropped toward zero and voltage readings became abnormal — both correctly classified as faults via sensor fusion logic.
- The 5V relay activated with **extremely fast response time**, ensuring instant disconnection from the distribution line.
- The GSM SIM800L successfully dispatched **SMS alerts and voice calls** including fault type, pole ID, GPS location link, and status to configured numbers.
- The 16×2 LCD correctly displayed real-time sensor status (NORMAL / FAULT DETECTED / TX LINE BREAK / RX FAULT).
- The RS485 module was tested for basic data transmission, confirming multi-node scalability.
- Minor sensor noise was observed under electrical interference but did not trigger false alarms due to multi-condition fusion logic.

---

## Future Scope

- **IoT Integration** — Connect to ThingSpeak or AWS IoT for real-time cloud monitoring dashboard replacing basic SMS alerts
- **GPS Module** — Pinpoint exact fault pole coordinates for GIS-based fault mapping
- **Predictive Maintenance** — Implement ML algorithms to analyze voltage, current, and vibration patterns and anticipate faults before they occur
- **Solar Power** — Enable continuous outdoor deployment with solar-powered operation for off-grid rural poles
- **Additional Sensors** — Add temperature, humidity, and arc detection sensors to enrich monitoring accuracy
- **Advanced MCU** — Upgrade to STM32F4 or STM32H7 for complex real-time computations and multi-protocol support
- **Mobile App** — Real-time fault notifications and network health dashboard for field engineers
- **Extended Communication** — LoRa or NB-IoT for areas with poor GSM coverage

---

> *Built to automate fault detection in rural power distribution using embedded multi-sensor intelligence — making India's low-voltage grid safer, one pole at a time.*
