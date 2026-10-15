# ISO 2575 Automotive Warning Telltale Architecture

This document defines the automotive warning telltale symbol encoding, bitmask mappings, and optical priority hierarchy implemented in the **`automotive-tft-cluster`** firmware in accordance with **ISO 2575: Road vehicles — Symbols for controls, indicators and tell-tales**.

---

## 1. Color Classification & Safety Tiers

ISO 2575 mandates specific optical coloration based on driver urgency:

| Color | Severity Tier | Operational Meaning | Action Required |
|:---:|:---:|:---|:---|
| 🔴 **RED** | **Danger / Critical** | Critical failure or safety hazard endangering vehicle occupants | Immediate vehicle stop and engine shutdown |
| 🟡 **AMBER / YELLOW** | **Caution / Advisory** | Non-critical component malfunction or system degraded mode | Service required soon; drive with caution |
| 🟢 **GREEN** | **Active Normal** | Normal system operation or driver assistance active | Informational only |
| 🔵 **BLUE** | **Status Indicator** | High-beam headlamps active | Driver awareness |

---

## 2. CAN Telemetry Telltale Bitmask Encoding (CAN ID: `0x280`)

The instrument cluster receives real-time telltale states via the powertrain / body CAN bus packed into a 16-bit bitfield:

```c
typedef enum {
    TELLTALE_CHECK_ENGINE      = (1U << 0),  /**< Yellow: MIL / OBD-II Emission Fault */
    TELLTALE_OIL_PRESSURE_LOW  = (1U << 1),  /**< Red: Critical Engine Lubrication Loss */
    TELLTALE_COOLANT_TEMP_HIGH = (1U << 2),  /**< Red: Thermal Overheat Threshold Exceeded */
    TELLTALE_BRAKE_SYSTEM_WARN = (1U << 3),  /**< Red: Parking Brake Engaged / Low Fluid */
    TELLTALE_BATTERY_FAULT     = (1U << 4),  /**< Red: Alternator / 12V Charging Failure */
    TELLTALE_ABS_FAULT         = (1U << 5),  /**< Yellow: Anti-Lock Braking Subsystem Deactivated */
    TELLTALE_ESC_ACTIVE        = (1U << 6),  /**< Yellow: Electronic Stability Control Intervention */
    TELLTALE_AIRBAG_FAULT      = (1U << 7),  /**< Red: SRS Airbag Safety System Disabled */
    TELLTALE_SEATBELT_UNBUCKLE = (1U << 8),  /**< Red: Occupant Restraint Unlatched */
    TELLTALE_DOOR_AJAR         = (1U << 9),  /**< Red: Door / Tailgate Latch Open */
    TELLTALE_TPMS_WARNING      = (1U << 10), /**< Yellow: Tire Under-Inflation Detected */
    TELLTALE_HIGH_BEAM         = (1U << 11), /**< Blue: High Beam Headlamps Active */
    TELLTALE_TURN_LEFT         = (1U << 12), /**< Green: Left Turn Directional Flasher */
    TELLTALE_TURN_RIGHT        = (1U << 13), /**< Green: Right Turn Directional Flasher */
    TELLTALE_FOG_LIGHTS        = (1U << 14), /**< Green: Front/Rear Fog Lamps Engaged */
} telltale_mask_t;
```

---

## 3. Optical Arbitration & Warning Overlay Hierarchy

When multiple warnings are active simultaneously, the cluster graphics compositor renders the highest-priority symbol in the primary center alert zone:

1. **Priority 1 (Critical Red):** Low Oil Pressure (`0x0002`), High Coolant Temp (`0x0004`), Brake Failure (`0x0008`).
2. **Priority 2 (Safety Red):** SRS Airbag (`0x0080`), Seatbelt (`0x0100`), Door Open (`0x0200`).
3. **Priority 3 (Advisory Yellow):** Check Engine (`0x0001`), ABS Fault (`0x0020`), TPMS (`0x0400`).
4. **Priority 4 (Normal Drivers):** High Beam (`0x0800`), Turn Signals (`0x1000` / `0x2000`).
