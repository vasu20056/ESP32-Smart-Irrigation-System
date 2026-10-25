# ESP32 Smart Irrigation - Power System Guide

## Status: ✅ COMPLETE POWER DESIGN

**Date:** 2026-01-01
**Component:** Power Supply System with D Cells + Laptop Connection

---

## Your Requirements

- **Available:** 1.1V D cells (NiMH rechargeable cells)
- **Need:** 3.3V for sensors, multiplexer, ESP32
- **Need:** 5V for relay module
- **Special:** ESP32 connected to laptop via USB for Serial Monitor

---

## CRITICAL UNDERSTANDING: Two Operating Modes

Your system will operate in **TWO DIFFERENT MODES**:

### Mode 1: Development/Debugging (USB Connected)
- ESP32 powered by laptop USB (5V)
- Serial Monitor active
- Use this for testing, calibration, monitoring

### Mode 2: Field Operation (Battery Powered)
- ESP32 powered by D cells
- Standalone operation
- Use this for actual irrigation in the field

---

## Solution Overview

### D Cell Configuration for 3.3V

**WRONG APPROACH:** You CANNOT get stable 3.3V from 1.1V cells by series connection:
- 2 cells = 2.2V (too low, ESP32 won't boot)
- 3 cells = 3.3V (perfect voltage BUT will drop to 2.7V as cells discharge)
- 4 cells = 4.4V (too high, will damage ESP32 3.3V logic)

**CORRECT APPROACH:** Use 4 cells + voltage regulator

```
┌──────────────────────────────────────────────┐
│  4× D Cells (1.1V NiMH) = 4.4V nominal       │
│         ↓                                     │
│  LM1117-3.3 Voltage Regulator                │
│         ↓                                     │
│      3.3V stable output                       │
└──────────────────────────────────────────────┘
```

**Why 4 cells?**
- Fresh: 4 × 1.1V = 4.4V
- Half-discharged: 4 × 1.0V = 4.0V (still enough for regulator)
- Minimum: 4 × 0.9V = 3.6V (regulator needs 3.6V minimum for 3.3V output)

---

## Complete Power System Design

### Components You Need

#### For Battery-Powered Operation:

1. **4× D Cell NiMH Batteries (1.1V each)**
   - Total voltage: 4.4V nominal
   - Capacity: ~8000-10000 mAh typical for D cells
   - Runtime: 10-15 hours continuous (depends on WiFi usage)

2. **D Cell Battery Holder (4-cell series)**
   - Amazon/AliExpress: "4 D Cell Battery Holder"
   - Output: Two wires (Red = +4.4V, Black = GND)
   - Price: $3-5

3. **LM1117-3.3 Voltage Regulator** (or AMS1117-3.3)
   - Input: 4.4V-15V
   - Output: 3.3V stable
   - Max current: 800mA (enough for ESP32 + sensors + MUX)
   - Price: $0.50-1 each

4. **Capacitors for Regulator Stability**
   - 10µF electrolytic (input side)
   - 10µF electrolytic (output side)
   - Price: $0.20 each

5. **5V Step-Up Converter (for relay)**
   - MT3608 DC-DC Boost Converter
   - Input: 3.3V
   - Output: 5V (adjustable)
   - Price: $1-2

6. **SPDT Power Switch**
   - To switch between USB and battery power
   - Or use a relay/diode OR circuit
   - Price: $1-2

---

## Power Layout Diagram

### MODE 1: Development (USB Connected)

```
┌─────────────────────────────────────────────────────────────────┐
│                    LAPTOP USB PORT (5V)                          │
└────────────────────────┬────────────────────────────────────────┘
                         │ USB Cable
                         ↓
                  ┌──────────────┐
                  │   ESP32      │
                  │   USB Port   │
                  └──────┬───────┘
                         │
              ┌──────────┴──────────┐
              │                     │
         5V (VIN pin)          3.3V (3V3 pin)
              │                     │
              │                     ├─── CD4051 Multiplexer VCC
              │                     │
              │                     ├─── Sensors 0-8 VCC
              │                     │    (9× capacitive sensors)
              │                     │
              ↓                     ↓
         ┌─────────────┐       ┌────────────┐
         │  5V Relay   │       │  Sensors   │
         │  Module     │       │  + MUX     │
         │             │       │            │
         │  VCC  GND   │       │  VCC  GND  │
         └──┬──────┬───┘       └──┬─────┬───┘
            │      │              │     │
            └──────┴──────────────┴─────┴─────── ESP32 GND
                                                  (Common ground)

🔌 BATTERY HOLDER: DISCONNECTED (or switched off)
```

**In this mode:**
- Laptop USB provides 5V to ESP32 VIN pin
- ESP32's onboard 3.3V regulator powers:
  - ESP32 logic
  - CD4051 multiplexer
  - All 9 sensors
- ESP32's 5V pin (VIN passthrough) powers relay module
- Serial Monitor works via USB
- **No batteries needed**

---

### MODE 2: Field Operation (Battery Powered)

```
┌──────────────────────────────────────────────────────────────────┐
│                   4× D CELLS (1.1V each)                          │
│                      Series Connection                            │
│                                                                    │
│    [+]──[D]──[D]──[D]──[D]──[-]                                  │
│     +4.4V              GND                                        │
└─────┬────────────────────┬───────────────────────────────────────┘
      │                    │
      │ +4.4V              │ GND
      ↓                    ↓
┌─────────────────────────────────┐
│   LM1117-3.3 Regulator          │
│                                  │
│   IN ─┤10µF├─[LM1117]─┤10µF├─ OUT
│  4.4V        GND  3.3V           │
└──────┬──────┬────┬───────────────┘
       │      │    │
     GND    GND  3.3V
       │      │    │
       │      │    ├──────────────────┬── ESP32 3.3V pin (direct power)
       │      │    │                  │
       │      │    ├─── CD4051 VCC    │
       │      │    │                  │
       │      │    ├─── Sensors VCC   │
       │      │    │    (0-8)         │
       │      │    │                  │
       │      │    └─── MT3608 Input  │
       │      │              ↓         │
       │      │         ┌──────────┐  │
       │      │         │ MT3608   │  │
       │      │         │ Boost    │  │
       │      │         │ 3.3V→5V  │  │
       │      │         └────┬─────┘  │
       │      │              │ 5V     │
       │      │              ↓        │
       │      │         ┌──────────┐  │
       │      │         │  Relay   │  │
       │      │         │  Module  │  │
       │      │         │  VCC     │  │
       │      │         └────┬─────┘  │
       │      │              │        │
       └──────┴──────────────┴────────┴─── Common GND

🔌 USB CABLE: DISCONNECTED
⚠️  IMPORTANT: Power ESP32 via 3.3V pin, NOT VIN pin
```

**In this mode:**
- 4× D cells provide 4.4V
- LM1117 regulator converts 4.4V → 3.3V stable
- 3.3V powers:
  - ESP32 directly via 3.3V pin
  - CD4051 multiplexer
  - All 9 sensors
- MT3608 boost converter: 3.3V → 5V for relay
- **No USB connection** (standalone field operation)

---

## Detailed Wiring Instructions

### Step 1: Build the 3.3V Regulator Circuit

```
Components:
- LM1117-3.3 regulator (TO-220 package)
- 10µF electrolytic capacitor (×2)
- Breadboard or PCB

Pinout (LM1117-3.3):
   ┌─────┐
   │ LM  │ (metal tab facing you)
   │1117 │
   └─┬─┬─┘
     1 2 3
     │ │ │
   GND OUT IN

Wiring:
  Battery +4.4V ──┤+10µF-├── Pin 3 (IN)

  Pin 1 (GND) ──── Battery GND

  Pin 2 (OUT) ──┤+10µF-├── 3.3V output
                   GND
```

**Physical Assembly:**
1. Insert LM1117 into breadboard
2. Connect 10µF capacitor: + leg to battery +4.4V, - leg to GND
3. Connect LM1117 pin 3 to battery +4.4V
4. Connect LM1117 pin 1 to GND
5. Connect 10µF capacitor: + leg to LM1117 pin 2, - leg to GND
6. Output is LM1117 pin 2 (3.3V stable)

---

### Step 2: Build the 5V Boost Converter Circuit

```
MT3608 Module (Typical Pinout):
┌─────────────┐
│   MT3608    │
│   □□        │ (potentiometer for voltage adjust)
│  IN+ IN-    │
│  OUT+ OUT-  │
└─────────────┘

Wiring:
  3.3V rail ──── IN+
  GND rail  ──── IN-

  Relay VCC ──── OUT+
  Relay GND ──── OUT-

Adjustment:
1. Connect multimeter to OUT+ and OUT-
2. Turn potentiometer clockwise to increase voltage
3. Adjust until multimeter reads 5.0V
4. Lock the potentiometer (use glue or nail polish)
```

---

### Step 3: Connect Power to ESP32

#### MODE 1 (USB Development):
```
Laptop USB ──── ESP32 USB port
                   │
                   ├─── ESP32 automatically powers itself
                   ├─── 3.3V pin becomes output (use for sensors)
                   └─── VIN pin becomes 5V output (use for relay)

Battery holder: DISCONNECTED or switched OFF
```

#### MODE 2 (Battery Field):
```
⚠️  CRITICAL: Power ESP32 via 3.3V pin, NOT VIN pin

Regulator 3.3V ──── ESP32 3.3V pin (input power)
Regulator GND  ──── ESP32 GND

DO NOT connect anything to VIN pin in battery mode!
USB cable: DISCONNECTED
```

**Why 3.3V pin in battery mode?**
- ESP32's VIN pin expects 5V minimum
- If you feed 3.3V to VIN, the onboard regulator will drop it to ~2.9V
- ESP32 needs 2.3V-3.6V on 3.3V pin to operate properly
- Bypassing the onboard regulator is more efficient

---

### Step 4: Connect Sensors and Multiplexer

```
All modes (USB or Battery):

3.3V rail ──┬─── CD4051 Pin 16 (VCC)
            │
            ├─── Sensor 0 VCC
            ├─── Sensor 1 VCC
            ├─── Sensor 2 VCC
            ├─── Sensor 3 VCC
            ├─── Sensor 4 VCC
            ├─── Sensor 5 VCC
            ├─── Sensor 6 VCC
            ├─── Sensor 7 VCC
            └─── Sensor 8 VCC

GND rail ───┬─── CD4051 Pin 8 (GND)
            ├─── CD4051 Pin 7 (VEE)
            ├─── CD4051 Pin 6 (INH connected to GND)
            ├─── Sensor 0 GND
            ├─── Sensor 1 GND
            ├─── Sensor 2 GND
            ├─── Sensor 3 GND
            ├─── Sensor 4 GND
            ├─── Sensor 5 GND
            ├─── Sensor 6 GND
            ├─── Sensor 7 GND
            └─── Sensor 8 GND

Sensor signals:
  Sensor 0 OUT ──── CD4051 Pin 13 (CH0)
  Sensor 1 OUT ──── CD4051 Pin 14 (CH1)
  Sensor 2 OUT ──── CD4051 Pin 15 (CH2)
  Sensor 3 OUT ──── CD4051 Pin 12 (CH3)
  Sensor 4 OUT ──── CD4051 Pin 1 (CH4)
  Sensor 5 OUT ──── CD4051 Pin 5 (CH5)
  Sensor 6 OUT ──── CD4051 Pin 2 (CH6)
  Sensor 7 OUT ──── CD4051 Pin 4 (CH7)
  Sensor 8 OUT ──── ESP32 GPIO 35 (direct)

  CD4051 Pin 3 (Z) ──── ESP32 GPIO 34
```

---

### Step 5: Connect Relay Module

#### MODE 1 (USB):
```
ESP32 VIN (5V) ──── Relay VCC
ESP32 GND      ──── Relay GND
ESP32 GPIO 23  ──── Relay IN (signal)
```

#### MODE 2 (Battery):
```
MT3608 OUT+ (5V) ──── Relay VCC
MT3608 OUT- (GND) ─── Relay GND
ESP32 GPIO 23     ─── Relay IN (signal)
```

---

### Step 6: Mode Switching Circuit (Optional but Recommended)

To safely switch between USB and battery modes without rewiring:

```
┌────────────────────────────────────────────────────────────┐
│  USB Power OR Battery Power (Auto-switching with diodes)   │
└────────────────────────────────────────────────────────────┘

                 LAPTOP USB 5V
                       │
                       ├─── 1N5817 Schottky Diode ───┐
                       │                              │
                                                      ↓
                 Battery 4.4V                    ┌────────┐
                       │                         │ Common │
                       ├─── 1N5817 Schottky ─────│  Rail  │
                       │    Diode                └────┬───┘
                                                      │
                                                   LM1117
                                                      ↓
                                                    3.3V

How it works:
- When USB connected: USB 5V > Battery 4.4V, so USB powers the system
- When USB disconnected: Battery takes over automatically
- Diodes prevent backflow (USB won't charge batteries, batteries won't damage USB)
```

**Alternative: Manual SPDT Switch**
```
        SPDT Switch
         ┌──┐
USB 5V ──│  │
         │  ├── Common ── Regulator Input
Batt ────│  │
         └──┘

Position 1: USB selected
Position 2: Battery selected
```

---

## Complete Parts List

### Essential Components:

| Component | Quantity | Purpose | Price (Est.) |
|-----------|----------|---------|--------------|
| D Cell NiMH 1.1V | 4 | Main power | $10-15 |
| 4-cell D battery holder | 1 | Battery housing | $3-5 |
| LM1117-3.3 regulator | 1 | 4.4V → 3.3V | $0.50 |
| 10µF electrolytic cap | 2 | Regulator stability | $0.40 |
| MT3608 boost module | 1 | 3.3V → 5V for relay | $1-2 |
| 1N5817 Schottky diode | 2 | Auto power switching | $0.40 |
| SPDT switch (optional) | 1 | Manual mode switch | $1 |
| Breadboard or PCB | 1 | Assembly | $2-5 |
| Jumper wires | 20+ | Connections | $2 |

**Total Cost:** ~$20-30

---

## Power Consumption Analysis

### Current Draw Breakdown:

| Component | Current (mA) | Voltage |
|-----------|--------------|---------|
| ESP32 (WiFi active) | 80-170 | 3.3V |
| ESP32 (WiFi sleep) | 20-40 | 3.3V |
| CD4051 Multiplexer | <1 | 3.3V |
| Capacitive Sensor (×9) | 5-10 each | 3.3V |
| **3.3V Rail Total** | **150-250 mA** | 3.3V |
| Relay Module (active) | 50-70 | 5V |
| Relay Module (idle) | 10-20 | 5V |

### Battery Life Estimation:

**D Cell Capacity:** 8000-10000 mAh typical

**Scenario 1: Continuous WiFi + Monitoring**
- Average draw: 200 mA @ 3.3V
- Runtime: 8000 mAh ÷ 200 mA = **40 hours** (1.6 days)

**Scenario 2: WiFi Sleep Mode (recommended)**
- Average draw: 50 mA @ 3.3V
- Runtime: 8000 mAh ÷ 50 mA = **160 hours** (6.6 days)

**Scenario 3: Deep Sleep (future enhancement)**
- Average draw: 5-10 mA @ 3.3V
- Runtime: 8000 mAh ÷ 10 mA = **800 hours** (33 days)

**Relay Impact:**
- Irrigation cycles: 2-4 per day × 15 minutes each
- Total relay time: 30-60 min/day
- Additional consumption: ~20 mAh/day (minimal)

---

## Power System Assembly Steps

### Final Assembly Checklist:

1. ✅ **Prepare battery pack:**
   - Insert 4× D cells into holder
   - Verify voltage: should read 4.2-4.6V with fresh cells
   - Mark polarity clearly (+/-)

2. ✅ **Build regulator circuit:**
   - Assemble LM1117 with capacitors on breadboard
   - Test output: should be 3.30V ±0.05V
   - Secure all connections

3. ✅ **Setup boost converter:**
   - Connect MT3608 to 3.3V rail
   - Adjust output to exactly 5.0V
   - Lock potentiometer

4. ✅ **Install power switching:**
   - Option A: Use diode OR circuit for auto-switching
   - Option B: Use SPDT switch for manual switching
   - Label positions clearly

5. ✅ **Connect ESP32:**
   - USB mode: Just plug USB cable
   - Battery mode: 3.3V → ESP32 3.3V pin, GND → ESP32 GND

6. ✅ **Connect peripherals:**
   - Sensors + MUX to 3.3V rail
   - Relay to 5V rail (USB VIN or MT3608 output)
   - All GNDs to common ground

7. ✅ **Test both modes:**
   - USB mode: Connect laptop, verify Serial Monitor works
   - Battery mode: Disconnect USB, verify system boots
   - Check voltage rails with multimeter

---

## Safety Warnings

### ⚠️ CRITICAL WARNINGS:

1. **NEVER connect USB and battery simultaneously without diodes**
   - Use diode OR circuit or SPDT switch
   - Can damage USB port or batteries

2. **NEVER exceed 3.6V on ESP32 3.3V pin**
   - Will permanently damage the chip
   - Always use LM1117 regulator, never direct battery connection

3. **NEVER reverse polarity**
   - ESP32, sensors, and MUX are NOT reverse-polarity protected
   - Double-check before powering on

4. **Relay voltage must be 5V**
   - 3.3V is insufficient to energize relay coil
   - Always use boost converter in battery mode

5. **Common ground is mandatory**
   - All GNDs must connect together
   - Including USB GND, battery GND, sensor GND, relay GND

---

## Troubleshooting

### Problem: ESP32 doesn't boot on battery power

**Possible Causes:**
- Battery voltage too low (<3.6V)
  - **Fix:** Charge/replace batteries
- Regulator output wrong
  - **Fix:** Measure LM1117 output, should be 3.25-3.35V
- Poor connections
  - **Fix:** Check all solder joints/breadboard connections

---

### Problem: Relay doesn't click in battery mode

**Possible Causes:**
- MT3608 output voltage wrong
  - **Fix:** Adjust potentiometer to 5.0V
- Insufficient current from boost converter
  - **Fix:** Use larger input capacitor (100µF) on MT3608

---

### Problem: System resets randomly on battery

**Possible Causes:**
- Voltage drops during WiFi transmission (high current spikes)
  - **Fix:** Add 100µF-220µF capacitor across ESP32 3.3V and GND
- Weak batteries
  - **Fix:** Use fresh/fully charged cells

---

### Problem: Serial Monitor doesn't work in USB mode

**Possible Causes:**
- Wrong COM port selected
  - **Fix:** Tools → Port → Select correct port
- USB driver not installed
  - **Fix:** Install CP210x or CH340 driver
- Battery also connected (conflict)
  - **Fix:** Disconnect battery when using USB

---

## Best Practices

### For Development (USB Mode):
1. Always disconnect battery when using USB
2. Use Serial Monitor for debugging
3. Test all functions before switching to battery mode
4. Keep USB cable connected during firmware uploads

### For Field Operation (Battery Mode):
1. Fully charge all 4 D cells before deployment
2. Disconnect USB cable completely
3. Use WiFi sleep mode to extend battery life
4. Monitor battery voltage via web dashboard
5. Replace batteries when voltage drops below 3.6V

### Power Management Code (Future Enhancement):

```cpp
// Add to your code for battery monitoring
float readBatteryVoltage() {
  // Use voltage divider on GPIO 33 to monitor battery
  int raw = analogRead(33);
  float voltage = (raw / 4095.0) * 3.3 * 2;  // Assuming 1:1 divider
  return voltage;
}

void checkBatteryLevel() {
  float battV = readBatteryVoltage();
  if (battV < 3.6) {
    Serial.println("⚠️ LOW BATTERY! Replace soon.");
    // Send notification via web dashboard
  }
}
```

---

## Summary: Quick Reference

### USB Mode (Development):
```
Laptop USB → ESP32 USB port
           → Auto-powers everything
           → Serial Monitor works
           → Battery: DISCONNECTED
```

### Battery Mode (Field):
```
4× D cells (4.4V) → LM1117 → 3.3V → ESP32 3.3V pin
                                   → Sensors + MUX
                                   → MT3608 → 5V → Relay
                     USB: DISCONNECTED
```

### Power Rails:
- **3.3V Rail:** ESP32, sensors, multiplexer
- **5V Rail:** Relay module only
- **GND:** Common ground for everything

### Key Voltages:
- Battery fresh: 4.4V
- Battery minimum: 3.6V (replace below this)
- Regulated output: 3.3V ±0.05V
- Boost output: 5.0V ±0.1V

---

## Files Modified

- **POWER_SYSTEM_GUIDE.md** - This file (complete power system documentation)

---

**Guide Complete:** ✅
**Power System Designed:** ✅
**Both Modes Documented:** ✅
**Status:** READY FOR ASSEMBLY

---

## Next Steps

1. **Purchase components** from the parts list above
2. **Assemble regulator circuit** on breadboard
3. **Test voltage outputs** with multimeter
4. **Connect ESP32** and verify both USB and battery modes
5. **Add battery monitoring** code (optional but recommended)
6. **Deploy to field** after thorough testing

If you need help with any specific assembly step or have questions about the wiring, let me know!
