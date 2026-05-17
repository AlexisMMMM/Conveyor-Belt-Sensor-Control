# 🏭 Conveyor Belt Sensor Control — Project 5

Control de banda transportadora con detección de objetos mediante lógica Ladder en CODESYS, comunicación Modbus TCP y simulación 3D en Factory I/O.

---

## 📋 Project Overview

This project implements an industrial conveyor belt control system where a motor starts automatically and stops when a sensor detects an object at a defined position. The system replicates a real industrial environment using only software tools.

**Key achievement:** Full industrial control loop running entirely in software — no physical hardware required.

---

## 🏗️ System Architecture

```
┌─────────────────┐     Modbus TCP      ┌──────────────────┐     3D Simulation
│   CODESYS V3.5  │ ◄─────────────────► │  Factory I/O     │ ──────────────────►
│   SoftPLC       │    127.0.0.1:502    │  Ultimate v2.5   │   Visual Feedback
│   Ladder Logic  │                     │  Scene: A to B   │
└─────────────────┘                     └──────────────────┘
```

### Signal Flow

| Signal | Direction | Modbus Register | Description |
|---|---|---|---|
| Sensor_Object | Factory I/O → CODESYS | Coil 0 | Object detected by physical sensor |
| Motor_Running | CODESYS → Factory I/O | Discrete Input 0 | Conveyor belt activation |

---

## ⚙️ Ladder Logic

### Variables

| Variable | Type | Description |
|---|---|---|
| Motor_Start | BOOL | Motor start signal |
| Motor_Stop | BOOL | Motor stop signal (NC contact) |
| Motor_Running | BOOL | Motor output / self-latch |
| Sensor_Object | BOOL | Object detection from Factory I/O |

### Logic Description

**Network 1 — Motor Start/Stop with Sensor Interlock:**

```
Motor_Start   Motor_Stop   Sensor_Object        Motor_Running
   ──┤ ├──────────┤/├──────────┤/├─────────────────( )───
         │
Motor_Running
   ──┤ ├──
```

- Motor starts when `Motor_Start = TRUE`
- Self-latch via `Motor_Running` contact (autoretencion)
- Motor stops when `Motor_Stop = TRUE` (NC contact opens)
- Motor stops when `Sensor_Object = TRUE` — object detected (NC contact opens)

---

## 🔌 Communication Setup

### CODESYS — Modbus TCP Server Device

| Parameter | Value |
|---|---|
| Port | 502 |
| Coils (Bobinas) | 2 — mapped to Sensor inputs |
| Discrete Inputs | 1 — mapped to Motor output |

**I/O Mapping:**
- `Bobinas Bit0` → `PLC_PRG.Sensor_Object`
- `Entradas discretas Bit0` → `PLC_PRG.Motor_Running`

### Factory I/O — Modbus TCP Client

| Parameter | Value |
|---|---|
| Host | 127.0.0.1 |
| Port | 502 |
| Slave ID | 1 |
| Read Digital | **Coils** ⚠️ |

**Signal Mapping:**
- `Sensor` → `Coil 0`
- `Input 0` → `Conveyor`

> ⚠️ **Critical:** Read Digital must be set to **Coils** (not Inputs). Setting it to Inputs inverts the communication direction and the sensor signal will not reach CODESYS.

---

## 🛠️ Tools & Software

| Tool | Version | Purpose |
|---|---|---|
| CODESYS Development System | V3.5 SP22 | PLC programming (Ladder) |
| CODESYS Control Win V3 x64 | 3.5.22.10 | SoftPLC runtime simulator |
| CODESYS Modbus | 4.6.0.0 | Modbus TCP Server device |
| Factory I/O | Ultimate v2.5.10 | 3D industrial simulation |

---

## 🐛 Troubleshooting Log

### Issue: Sensor signal not reaching CODESYS

**Symptom:** Sensor_Object value in CODESYS did not change when Factory I/O sensor detected the box. Manual forcing worked correctly.

**Root cause:** Factory I/O's `Read Digital` was set to `Inputs`, causing it to READ discrete inputs from CODESYS instead of WRITING sensor values to the PLC coils.

**Fix:** Changed `Read Digital` from `Inputs` to `Coils` in Factory I/O driver configuration.

**Lesson learned:** In Modbus TCP Client/Server setups, signal direction (read vs write) must match the data flow intent. Sensors from Factory I/O must WRITE to the PLC — not read from it.

---

## 📸 Screenshots

| Description | File |
|---|---|
| Ladder Logic running | `screenshots/ladder_logic.png` |
| Factory I/O conveyor with sensor stop | `screenshots/factory_io_running.png` |
| Modbus driver mapping | `screenshots/modbus_driver.png` |

---

## 🗺️ Roadmap Context

This project is **Part 5** of an Industry 4.0 Engineer learning path:

- ✅ Project 1 — Traffic Light (Semaforo Vehicular)
- ✅ Project 2 — Conveyor Belt with Product Sorter
- ✅ Project 3 — Tank Level Control HMI
- ✅ Project 4 — Star-Delta Motor Starter
- ✅ **Project 5 — Conveyor Belt Sensor Control (CODESYS + Modbus TCP + Factory I/O)**
- 🔜 Project 6 — OPC UA Data Bridge to Python
- 🔜 Project 7 — Industrial Dashboard (InfluxDB + Grafana)

---

## 👩‍💻 Author

**Kaori** — Control & Automation Engineer  
[github.com/AlexisMMMM](https://github.com/AlexisMMMM)
