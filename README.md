# MRE-320-PID-Rover-Report
# Wall-Following Robot with PID Control

## Project Overview

This project challenges students to design and build a mobile robot capable of autonomously following a wall at a fixed distance using feedback control. The robot utilizes a **PID (Proportional–Integral–Derivative) controller** to adjust motor speeds and steering based on distance measurements from ultrasonic sensors. The project emphasizes practical application of control theory, sensor integration, and embedded system design.

**Target distance:** 30 cm from the left edge of the robot to the wall  
**Key behaviors:** Maintain stable distance, minimal oscillation, detect front obstacles, stop or reroute as needed

---

## Table of Contents

- [Technical Requirements](#technical-requirements)
- [Hardware Components](#hardware-components)
- [Engineering Problem-Solving Process](#engineering-problem-solving-process)
  - [Step 1: Understand the Problem Context](#step-1-understand-the-problem-context)
  - [Step 2: Apply Engineering Principles](#step-2-apply-engineering-principles)
  - [Step 3: Formulate the Problem Mathematically](#step-3-formulate-the-problem-mathematically)
  - [Step 4: Design and Implement the Solution](#step-4-design-and-implement-the-solution)
  - [Step 5: Evaluate and Iterate](#step-5-evaluate-and-iterate)
- [Critical Troubleshooting: Motor Gearbox Issue](#critical-troubleshooting-motor-gearbox-issue)
- [PID Tuning Results](#pid-tuning-results)
- [Lessons Learned](#lessons-learned)

---

## Technical Requirements

| Component | Specification | Status |
|-----------|---------------|--------|
| Microcontroller | Arduino Uno | ✅ |
| Motor Driver | L293D | ✅ |
| Actuators | 2x DC motors with gearboxes | ✅ |
| Sensors | 2x Ultrasonic sensors (HC-SR04) | ✅ |
| Target Distance | 30 cm from robot edge | ✅ |
| Power Source | 7.4V LiPo battery | ✅ |
| Chassis | 3D-printed custom design | ✅ |
| Drive Wheels | 3D-printed (2x) | ✅ |
| Caster Wheel | 3D-printed (1x) | ✅ |
| Wheel Configuration | 3-wheel (2 driven + 1 caster) | ✅ |

> ⚠️ **Note:** Original DC motors without gearboxes FAILED to meet requirements. See [Critical Troubleshooting](#critical-troubleshooting-motor-gearbox-issue) for details.

---

## Hardware Components

### Final Configuration (Successful)

| Component | Specification | Purpose |
|-----------|---------------|---------|
| Arduino Uno | Arduinomega | Main controller |
| L293D Motor Driver | H-bridge IC | Direction and speed control |
| DC Motors (x2) | With 50:1 gearboxes | Drive and steering (differential drive) |
| Ultrasonic Sensor (Front) | HC-SR04 | Obstacle detection |
| Ultrasonic Sensor (Left) | HC-SR04 (slanted) | Wall distance measurement |
| Chassis | 3D-printed PLA | Mechanical platform |
| Drive Wheels (x2) | 3D-printed PLA | Traction and movement |
| Caster Wheel (x1) | 3D-printed with ball bearing or low-friction tip | Balance and stability (3rd wheel) |
| Battery | 7.4V LiPo battery| Power source |

### Custom 3D-Printed Components

| Component | Material | Quantity | Design Notes |
|-----------|----------|----------|---------------|
| Chassis | PLA | 1 | Two-layer design with motor mounts and sensor brackets |
| Drive Wheels | PLA | 2 | Diameter: 65 mm, width: 20 mm, rubber band traction surface |
| Caster Wheel | PLA | 1 | Omni-directional or fixed swivel design for stable 3-point contact |
| Sensor Mounts | PLA | 2 | Adjustable brackets for front and left ultrasonic sensors |

### Wheel Configuration (3-Wheel Design)

The robot uses a **3-wheel configuration** for stability and simplicity:

| Wheel | Type | Location | Function |
|-------|------|----------|----------|
| Left Drive Wheel | 3D-printed, powered | Left side, center | Propulsion and steering (differential) |
| Right Drive Wheel | 3D-printed, powered | Right side, center | Propulsion and steering (differential) |
| Caster Wheel | 3D-printed, free-rotating | Front or rear center | Balance, prevents tipping, allows free rotation |

**Why 3 wheels instead of 4?**
- **Static stability:** 3 points always contact the ground (no rocking)
- **Simplified kinematics:** No wheel slipping issues common with 4-wheel skid-steer
- **Weight distribution:** Lighter than 4-wheel designs
- **Easy 3D printing:** Single caster wheel is simpler than matching four wheels

**Caster Wheel Design:**
- Printed in two parts: fork + wheel
- Optional: small ball bearing inserted for smooth rotation
- Mounted at the front of the chassis for stability during forward motion

### Sensor Placement

| Sensor | Location | Orientation | Purpose |
|--------|----------|-------------|---------|
| Front ultrasonic | Front center (3D-printed bracket) | Straight ahead | Obstacle detection (stop within 15 cm) |
| Left ultrasonic | Left side, 5 cm from front | Slightly slanted (approx. 15°) | Wall distance measurement for PID |

**Note on slanted left sensor:** The slant angle provides a "look-ahead" effect, helping the robot anticipate wall changes before they arrive. The measured distance is calibrated using `actual = measured × cos(θ)`.

---

## Engineering Problem-Solving Process

### Step 1: Understand the Problem Context

#### Core Objective
Maintain a consistent lateral distance of 30 cm from a wall while moving forward autonomously.

#### Sub-Problems Identified

| Sub-Problem | Description | Challenge |
|-------------|-------------|-----------|
| **Sensing** | Measure distance to wall and front obstacles | Ultrasonic noise, reflective surfaces, slant angle calibration |
| **Control** | Convert distance error into motor commands | Nonlinear response, PID tuning, stability |
| **Actuation** | Implement differential drive with available motors | Torque requirements, motor asymmetry, PWM response |
| **Mechanical Stability** | Maintain balance and ground contact | 3-wheel caster design prevents rocking |
| **Obstacle Handling** | Prioritize safety over wall-following | State management, override logic |

#### Interdisciplinary Aspects
- **Control Engineering** – PID feedback loops, stability analysis
- **Embedded Systems** – Real-time constraints, Arduino timing
- **Mechanical Design** – 3D-printed chassis, 3-wheel kinematics, gearbox selection, caster wheel integration
- **Electrical Engineering** – Power distribution (9V to 5V regulation), noise filtering

---

### Step 2: Apply Engineering Principles

#### PID Control Theory

| Term | Role | Effect |
|------|------|--------|
| **Proportional (Kp)** | Responds to current error | Fast response, can cause oscillation |
| **Integral (Ki)** | Accumulates past error | Eliminates steady-state error, risk of windup |
| **Derivative (Kd)** | Predicts future error | Dampens oscillation, sensitive to noise |

#### Ultrasonic Sensing Principle

A 3-reading moving average filter reduced noise by approximately 90% without adding significant lag.

#### Differential Drive Logic (3-Wheel Configuration)

With a caster wheel providing stability, the two driven wheels control all motion:

| Command | Left Motor | Right Motor | Effect |
|---------|------------|-------------|--------|
| Forward | Base + correction | Base - correction | Straight or turn |
| Reverse | - (Base + correction) | - (Base - correction) | Backward motion |
| Stop | 0 | 0 | Halt |
| Correction sign | + when too far | - when too far | Turn toward wall |

The caster wheel rotates freely and follows the direction set by the driven wheels.

#### Power Management

The robot is powered by a single 7.4V LiPo rechargeable battery that supplies both the Arduino and the L293D motor driver.

---

### Step 3: Formulate the Problem Mathematically

#### Variable Definitions

| Variable | Symbol | Value/Unit | Description |
|----------|--------|------------|-------------|
| Target distance | `r` | 30.0 cm | Desired wall distance |
| Measured distance | `y(t)` | cm | Actual wall distance at time t |
| Error | `e(t)` | cm | `r - y(t)` (positive = too far) |
| Control output | `u(t)` | dimensionless | PID result (-255 to +255 scale) |
| Sampling time | `dt` | 0.05 sec | Control loop period |
| Base speed | `v_base` | 150 PWM | Nominal forward speed |

#### PID Equation (Discrete Form)

The discrete-time PID control equation is implemented in the Arduino loop with a sampling time `dt = 0.05` seconds:

$$ u(t) = K_p \cdot e(t) + K_i \cdot \sum_{k=0}^{t} e(k) \cdot \Delta t + K_d \cdot \frac{e(t) - e(t-1)}{\Delta t} $$

Where:

| Term | Symbol | Description |
|------|--------|-------------|
| Control output | $u(t)$ | Motor correction value applied to left/right speeds |
| Proportional gain | $K_p = 18$ | Responds to current error |
| Integral gain | $K_i = 3$ | Accumulates past error (eliminates steady-state error) |
| Derivative gain | $K_d = 40$ | Predicts future error (dampens oscillation) |
| Error at time t | $e(t)$ | Target distance (30 cm) minus measured wall distance |
| Previous error | $e(t-1)$ | Error from previous loop iteration |
| Sampling time | $\Delta t = 0.05$ | Time between control loop updates (20 Hz) |

**In code implementation:**

- The integral term is limited using anti-windup to prevent excessive accumulation when the robot is stuck or turning sharply
- The derivative term uses the difference between current and previous error divided by `dt`
- The final correction is constrained to the range [-255, 255] before being added/subtracted from motor speeds
**Final tuned gains:** Kp = 18, Ki = 3, Kd = 40

#### Motor Speed Mapping

$$ \text{leftSpeed} = \text{constrain}(v_{base} + u(t), 0, 255) $$

$$ \text{rightSpeed} = \text{constrain}(v_{base} - u(t), 0, 255) $$

The correction is **added** to the left motor and **subtracted** from the right motor so that:
- Positive error (too far from wall) → positive correction → left motor faster, right motor slower → robot turns RIGHT toward the wall
- Negative error (too close to wall) → negative correction → left motor slower, right motor faster → robot turns LEFT away from the wall

#### Obstacle State Machine Logic

$$ \text{baseSpeed} = \begin{cases} 
150 & \text{if frontDistance} \geq 20 \text{ cm} \\
80 & \text{if } 10 \leq \text{frontDistance} < 20 \text{ cm} \\
0 & \text{if frontDistance} < 10 \text{ cm}
\end{cases} $$

#### 3-Wheel Kinematics

For a differential drive robot with a caster wheel:
- Instantaneous center of rotation (ICR) lies on the axis between the two driven wheels
- Caster wheel automatically aligns with the direction of motion
- No additional kinematic equations needed for control — the caster is passive


---

### Step 4: Design and Implement the Solution

#### Hardware Implementation

**3D-Printed Chassis Design:**
- Custom-designed in CAD software
- Two-layer design: lower layer for motors/battery, upper layer for Arduino/sensors
- Integrated motor mounts for geared DC motors
- Sensor brackets 
- **Caster wheel mount** at the front of the chassis

**3D-Printed Drive Wheels (2x):**
- Diameter: 65 mm for good ground clearance
- Width: 20 mm for stability
- Hexagonal hub for secure motor shaft attachment

**3D-Printed Caster Wheel (1x):**
- Diameter: 25-30 mm (smaller than drive wheels)
- Swivel design or fixed low-friction tip
- Mounted at the front center of chassis
- Provides 3-point contact with ground (never rocks)

**Why front caster vs. rear caster:**
- Front caster placement pulls the robot during forward motion
- Provides stability when stopping (prevents nose-diving)
- Simplifies weight distribution (battery can be placed in rear)

**Sensor Mounting:**
- Left ultrasonic: 5 cm from front, slanted forward at approximately 15° using 3D-printed bracket
- Front ultrasonic: Centered on front bumper bracket above caster wheel

### Power Distribution (7.4V LiPo Battery)

The robot is powered by a single 7.4V LiPo rechargeable battery that supplies both the Arduino and the L293D motor driver.

**Power Connections:**

| Component | Connected To | Voltage |
|-----------|--------------|---------|
| Arduino Vin | 7.4V LiPo battery (+) | 7.4V |
| L293D VCC2 (motor power) | 7.4V LiPo battery (+) | 7.4V |
| L293D VCC1 (logic power) | Arduino 5V pin | 5V |
| Ultrasonic sensors (VCC) | Arduino 5V pin | 5V |
| All GND pins | 7.4V LiPo battery (-) | 0V (common ground) |

**Key Design Choices:**

- **Common ground** – All components share the same ground connection to ensure stable sensor readings and motor control
- **1000µF capacitor** – Connected across motor power supply to absorb current spikes and prevent Arduino resets
- **Separate power paths** – L293D uses dedicated 7.4V for motors and clean 5V from Arduino for logic, isolating electrical noise

**Expected runtime:** Approximately 45 minutes on a fully charged 7.4V LiPo battery (typical 1000-1500 mAh capacity).

### Step 5: Evaluate and Iterate

**Final Performance Metrics:**
- Steady-state error: ±1.5 cm (target < 2 cm) ✅
- Oscillation amplitude: ±3 cm (target < 5 cm) ✅
- Response time: 0.7 seconds (target < 1 sec) ✅
- Start from rest: Automatic ✅
- ±30° initial angle recovery: Within 3 seconds ✅
- Corner handling: Smooth, no wall contact ✅
- Obstacle stop: 14.2 cm (target < 15 cm) ✅
- Battery life: ~45 minutes (target > 30 min) ✅

**Final PID Gains:** Kp = 18, Ki = 3, Kd = 40

---

## Critical Troubleshooting: Motor Gearbox Issue

### The Problem (Initial Failure)

With standard DC motors (no gearboxes), the robot exhibited:

- No movement when placed on floor - required manual push to start
- After push, could not maintain straight line
- Could not hold 30 cm wall distance - spun in circles
- Starting at ±30° angle caused complete loss of orientation

### Root Cause

**Insufficient stall torque:** The original motors could not overcome static friction between wheels and floor. Static friction exceeds dynamic friction, requiring more torque to start than to maintain motion.

**Motor asymmetry:** Left and right motors had slightly different torque outputs even with identical PWM signals, causing continuous turning.
### Solution

Replaced standard DC motors with **DC motors equipped with 50:1 gearboxes**.

**Why gearboxes solved the problem:**
- Torque multiplication: output torque = motor torque × gear ratio (50× increase)
- Mechanical damping reduces relative differences between motors
- Higher torque allows PID to overcome initial error
- Full control authority enables recovery from ±30° misalignment

**Quantitative improvement:** Stall torque increased from approximately 0.5 kg·cm to 25 kg·cm (50× increase).

### Results After Motor Change
| Test Condition | Before | After |
|----------------|--------|-------|
| Start from rest | ❌ No motion | ✅ Immediate |
| ±30° initial angle | ❌ Loses wall | ✅ Recovers |
| Straight line | ❌ Impossible | ✅ Stable |
| Wall distance | ❌ Cannot hold | ✅ ±1.5 cm |

**Key Lesson:** A control system is only as good as its actuators. No amount of PID tuning can compensate for motors that cannot physically produce required forces. The exact same code worked perfectly after upgrading to gearbox motors.

## PID Tuning Results

### Tuning History

| Trial | Kp | Ki | Kd | Behavior | Outcome |
|-------|----|----|----|----------|---------|
| 1 | 10 | 0 | 0 | Slow response, error >8 cm | ❌ Too slow |
| 2 | 30 | 0 | 0 | Wild oscillation, crashed | ❌ Unstable |
| 3 | 20 | 5 | 0 | Improved but overshoots | ❌ Steady error |
| 4 | 18 | 3 | 30 | Smooth, minor oscillation | ✅ Acceptable |
| 5 | 18 | 3 | 40 | Optimal, minimal oscillation | ✅ FINAL |

### Error Over Time (Final Tuning)

After a step change in wall position (e.g., entering a curve), the error converges to zero within 0.7 seconds. Maximum overshoot is 2.5 cm. Steady-state error remains within ±1.5 cm.

### Final Gains Justification

- **Kp = 18:** Provides responsive correction without causing oscillation. Higher values (30+) caused unstable behavior.
- **Ki = 3:** Small integral term eliminates steady-state error on long straight walls. Higher values caused windup during curves.
- **Kd = 40:** Strong derivative term dampens oscillations from the P term. The high value compensates for mechanical inertia and sensor lag.
## Lessons Learned

### 1. Mechanical Foundation First
Hours spent tuning PID gains were wasted because the real problem was mechanical (insufficient motor torque). Always verify actuators before tuning controllers.

### 2. Static Friction Is a Hidden Variable
Floor surface, wheel material, and robot weight determine starting torque requirements. A robot that works on a smooth table may fail on carpet.

### 3. Motor Symmetry Is Not Guaranteed
Two "identical" DC motors can have different torque outputs. Gearboxes reduce this problem through mechanical damping and torque multiplication.

### 4. The "Needs a Push" Symptom Has One Primary Cause
If a wheeled robot needs a push to start, suspect insufficient motor torque before suspecting code or sensors.

### 5. 3D Printing Enables Rapid Iteration
Custom 3D-printed chassis and brackets allowed quick redesign of sensor mounts and wheel dimensions without waiting for shipped parts.

### 6. Three Wheels Are Better Than Four
The 3-wheel configuration (2 driven + 1 caster) eliminated rocking and provided perfect ground contact at all times. No complex suspension needed.

### 7. 7.4V LiPo Battery Considerations
The 7.4V LiPo battery provides sufficient voltage and current for both motors and Arduino. A 1000µF capacitor across the motor power supply smoothed current spikes and prevented Arduino resets.

