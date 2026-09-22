# Automatic Centre Stand for Motorcycles

An Arduino-controlled electromechanical center stand mechanism designed to eliminate manual lifting effort when parking motorcycles. Developed for the **Mechatronics and Instrumentation Lab** at **IIT Indore**.

---

## Key Features

* **One-Touch Automated Actuation:** Converts rotational motion into linear thrust to deploy and retract the center stand smoothly via simple button inputs.
* **Relay H-Bridge Direction Control:** Uses two 5V electromechanical relays to reverse polarity for bidirectional TT gear motor operation.
* **Automatic Motor Cutoff (Overload Prevention):** Features continuous digital monitoring using end-stop limit switches to immediately cut off motor power upon full deployment or retraction, preventing motor stall, over-travel, and battery drain.
* **Compact Telescoping Actuator:** Implements a lead screw ("rod-in-a-rod") design with an M6 threaded rod and sliding T-nut housed inside protective guide tubes.

---

## Hardware Architecture & Components

| Component | Function / Specification |
| :--- | :--- |
| **Microcontroller** | Arduino Nano R3 (ATmega328P) |
| **Actuator Drive** | TT DC Gear Motor (High-torque, low RPM) |
| **Transmission** | M6 Steel Threaded Rod + M6 T-Nut captive assembly |
| **Motor Control** | 2-Channel 5V Relay Modules (H-Bridge configuration) |
| **Sensors / Limits** | Tactile Micro-Switches (Extend / Retract end-stops) |
| **Power Supply** | 9V DC / Regulated 5V Step-Down for Logic |

---

## System Pinout Mapping

| Arduino Pin | Connection / Component | Logic / Signal |
| :--- | :--- | :--- |
| **Digital Pin 2** | Forward Command Input (`leverForward`) | `INPUT_PULLUP` |
| **Digital Pin 3** | Reverse Command Input (`leverReverse`) | `INPUT_PULLUP` |
| **Digital Pin 4** | Extended Stop Switch (`stopButtonFwd`) | `INPUT_PULLUP` |
| **Digital Pin 5** | Retracted Stop Switch (`stopButtonRev`) | `INPUT_PULLUP` |
| **Digital Pin 6** | Relay Forward Control (`relayForward`) | `OUTPUT` |
| **Digital Pin 7** | Relay Reverse Control (`relayReverse`) | `OUTPUT` |

---

## Firmware Overview

The system firmware operates as a finite state machine with integrated safety interlocks:

```cpp
// Automatic Centre Stand Control Logic
const int leverForward  = 2;
const int leverReverse  = 3;
const int stopButtonFwd = 4;
const int stopButtonRev = 5;

const int relayForward  = 6;
const int relayReverse  = 7;

const int RELAY_ON  = LOW;
const int RELAY_OFF = HIGH;

enum State { STOPPED, FORWARD, REVERSE };
State motorState = STOPPED;

void setup() {
  pinMode(leverForward, INPUT_PULLUP);
  pinMode(leverReverse, INPUT_PULLUP);
  pinMode(stopButtonFwd, INPUT_PULLUP);
  pinMode(stopButtonRev, INPUT_PULLUP);
  pinMode(relayForward, OUTPUT);
  pinMode(relayReverse, OUTPUT);
  stopMotor();
}

void loop() {
  bool stopFwdPressed = (digitalRead(stopButtonFwd) == LOW);
  bool stopRevPressed = (digitalRead(stopButtonRev) == LOW);

  // Safety Cutoff: Stop buttons have absolute priority
  if (stopFwdPressed || stopRevPressed) {
    stopMotor();
    motorState = STOPPED;
  }
}

void startForward() {
  digitalWrite(relayReverse, RELAY_OFF);
  delay(20);
  digitalWrite(relayForward, RELAY_ON);
}

void startReverse() {
  digitalWrite(relayForward, RELAY_OFF);
  delay(20);
  digitalWrite(relayReverse, RELAY_ON);
}

void stopMotor() {
  digitalWrite(relayForward, RELAY_OFF);
  digitalWrite(relayReverse, RELAY_OFF);
}
