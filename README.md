# Industrial Robot Multi-Scenario Control Simulator

**C# / Blazor / Three.js**

工業多場景機器人控制器模擬系統，整合 **Robot Kinematics、Sensor Fusion、Motion Control、Industrial Communication、Fault Recovery** 與 3D Digital Twin。

本專案以軟體模擬方式建立工業自動化控制流程，在沒有實體機械手臂的情況下，驗證控制演算法、感測資料處理、設備狀態機與異常復原邏輯。

---

## Features

### 1. Multi-Scenario Robot Control

提供三種工業自動化控制情境：

* **SEMI E84 Wafer Handling**
* **7-DOF Linear Rail Transfer**
* **Dual Conveyor Dynamic Tracking**

共用機械手臂控制、感測器資料、狀態管理與 Recovery 機制。

---

### 2. Robot Motion Control

建立 6-DOF Robot Arm 與 1-DOF Linear Rail 模型：

* Inverse Kinematics（IK）
* Forward Kinematics（FK）
* Joint Position / Velocity Control
* S-Curve Trajectory
* Velocity / Acceleration Limitation
* Simplified Multibody Dynamics
* Gravity / Friction / Torque Modeling

控制流程：

```text
Target Pose
    ↓
Inverse Kinematics
    ↓
Joint Target
    ↓
Trajectory Planning
    ↓
Dynamic Motion Update
    ↓
Robot State
```

---

### 3. Multi-Sensor Fusion

模擬多種工業感測器：

* RGB-D
* Encoder
* ToF
* Photoelectric Sensor
* Pressure Sensor

以 **Extended Kalman Filter（EKF）**進行物體位置與姿態估測，並將 Encoder FK、RGB-D Estimate 與其他感測資料用於狀態診斷。

```text
RGB-D ─────┐
ToF ───────┤
            ├→ State Estimation → Controller
Encoder ────┤
Pressure ───┘
```

---

### 4. Conveyor Dynamic Tracking

建立雙輸送帶自動抓取流程。

輸送帶以固定速度輸送物料，控制器根據物料估測位置計算 **Time-to-Intercept（TTI）**，並以 **50 Hz 動態更新**目標位置，再透過 IK 與動態控制完成移動中的物料攔截。

```text
Conveyor
    ↓
Object Detection
    ↓
EKF State Estimation
    ↓
Time-to-Intercept
    ↓
50 Hz Dynamic Tracking
    ↓
Inverse Kinematics
    ↓
Grasp
    ↓
Transfer to Output Belt
```

---

### 5. Closed-Loop Grasp Control

抓取過程並非單純依照固定座標執行，而是整合多種回饋資訊：

* Admittance Control
* Adaptive TCP Offset
* Ground Offset Estimation
* Pressure Feedback
* Torque Monitoring
* Grasp Overlap Ratio

根據感測結果動態修正 TCP 與接觸位置，提高抓取成功率。

---

### 6. Fault Detection & Recovery

建立完整的抓取失敗復原流程：

```text
Grasp Failure
      ↓
Residual Analysis
      ↓
Position Compensation
      ↓
Retract
      ↓
Closed-Loop Retry
      ↓
 ┌───────────────┐
 │               │
Success       Failure
 │               │
 ↓               ↓
Continue    ALARM / PAUSED
                ↓
              HOME
```

透過位置殘差、Pressure、Torque 與 Overlap 等資訊判斷異常，失敗時進行補償與二次嘗試，連續失敗後進入 Alarm / Pause / Home Recovery。

---

### 7. SEMI E84 / SECS-GEM

模擬半導體自動化設備通訊與控制流程。

#### SEMI E84 PI/O

包含：

* VALID
* CS_0
* TR_REQ
* BUSY
* COMP
* L_REQ
* READY
* HO_AVBL

實作基本 E84 Transfer Handshake：

```text
E84_IDLE
   ↓
E84_WAIT_READY
   ↓
E84_TRANSFER_ACTIVE
   ↓
Transfer Complete
   ↓
E84_IDLE
```

#### SECS/GEM

模擬 Host Command 與 Equipment Control：

* S2F41 Host Command
* S2F42 Command Acknowledge
* Event / Alarm
* Equipment Control State
* Remote Command
* Task Cancellation / Recovery

---

## System Architecture

```text
┌────────────────────────────────────────────┐
│                 Blazor UI                  │
└─────────────────────┬──────────────────────┘
                      │
                      ↓
┌────────────────────────────────────────────┐
│          Robot Control / State Machine     │
│                                            │
│  IK / FK / Trajectory / Dynamics /        │
│  Grasp / Recovery / Scenario Control       │
└──────────────┬─────────────────────────────┘
               │
       ┌───────┴────────┐
       ↓                ↓
┌──────────────┐  ┌──────────────────┐
│ Sensor Fusion│  │ Industrial Comm. │
│ EKF / Sensors│  │ E84 / SECS-GEM   │
└──────┬───────┘  └──────────────────┘
       │
       ↓
┌────────────────────────────────────────────┐
│          Simulation / Digital Twin         │
│                                            │
│  Robot / Rail / Conveyor / Sensors / Box   │
└─────────────────────┬──────────────────────┘
                      │
                      ↓
              Three.js / JSInterop
```

Three.js 主要負責 **3D Simulation 與 Visualization**，控制演算法與狀態處理則由 C# / Blazor 執行。

---

## Technology Stack

| Category         | Technology                           |
| ---------------- | ------------------------------------ |
| Language         | C#                                   |
| Framework        | Blazor                               |
| Visualization    | Three.js                             |
| Interop          | JavaScript Interop                   |
| State Estimation | EKF                                  |
| Robot Control    | IK / FK / Trajectory Control         |
| Motion           | S-Curve / Dynamic Simulation         |
| Communication    | SECS/GEM / SEMI E84                  |
| Simulation       | Robot / Conveyor / Sensor Simulation |

---

## Control Flow

完整控制流程：

```text
Sensor Input
     ↓
Sensor Fusion / EKF
     ↓
State Estimation
     ↓
Target Prediction
     ↓
IK / Trajectory Planning
     ↓
Robot Motion Control
     ↓
Grasp / Transfer
     ↓
Sensor Feedback
     ↓
Success / Failure Detection
     ↓
Compensation / Recovery
```

---

## Project Purpose

本專案主要用於驗證**工業機器人控制軟體的演算法與系統流程**。

透過 Software Simulation / Digital Twin，在沒有實體機械手臂的環境下，先驗證：

* 控制流程
* 感測器融合
* 運動規劃
* 動態物料追蹤
* 工業設備通訊
* Fault Detection
* Recovery Logic

降低直接於實體設備進行演算法開發與除錯的成本。

> **Note:** 本專案為 Software Simulation / Prototype，並非實際工業機械手臂控制器；真實設備仍需進一步處理 Hardware I/O、Robot SDK、即時性、Calibration、Safety Interlock 與實機參數。
