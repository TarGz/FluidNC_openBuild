Brief: FluidNC 4X TMC2209 Motor Configuration - Lessons Learned
Context
User configured a NEMA 17 stepper motor (SIMAX3D, resistance 3.8Ω, ~1.2-1.5A rated) on a FluidNC 4X board with integrated TMC2209 drivers.
Critical Discovery: Pin Mapping Must Match Physical Connector
THE MAIN ISSUE: The board has 4 motor connectors labeled X, Y, Z, A (left to right). Each connector has a specific UART address AND specific I2SO pins that MUST match.
FluidNC 4X Pin Mapping Table:
Physical PositionLabelUART addrstep_pindirection_pindisable_pin1st (left)X0i2so.2i2so.1i2so.02ndY1i2so.5i2so.4i2so.73rdZ2i2so.10i2so.9i2so.84th (right)A3i2so.13i2so.12i2so.15
User's motor was physically connected to the Z connector (3rd position) but config initially had addr: 3 with Motor 4 pins (i2so.13, i2so.12, i2so.15). Motor wouldn't move despite commands being accepted.
Key Configuration Parameters
1. TMC2209 Current Settings
For NEMA 17 with 3.8Ω resistance (1.2-1.5A nominal):
yamlrun_amps: 1.2        # 80-100% of motor rated current
hold_amps: 0.5       # 40-50% of run_amps (reduces heat at rest)
Too low = insufficient torque. Too high = overheating.
2. Disable Pin Logic
TMC2209 often requires inverted disable signal:
yamldisable_pin: i2so.8:low    # :low inverts the signal
Test:

$MD should make motor FREE to turn manually
$ME should LOCK motor (hard to turn)

If backwards, try :high instead of :low, or toggle use_enable: true/false
3. Other Critical Settings
yamluart_num: 1              # TMC2209 UART bus
r_sense_ohms: 0.11       # Standard for TMC2209
microsteps: 16           # Balance of precision/speed
run_mode: CoolStep       # Energy efficient
```

## Configuration Commands

### View Current Config:
```
$CD          # Shows full config (NOT $$)
$$           # Only shows GRBL-style settings, NOT TMC2209 params
```

### TMC2209 Parameters Cannot Be Changed via Terminal
- Settings like `run_amps`, `hold_amps`, `addr`, pins are **ONLY in config.yaml**
- Must edit file via WebUI Config tab, save, then `$Restart`
- No `$` command can modify these values

### Motor Control Commands:
```
$ME          # Enable motors (lock)
$MD          # Disable motors (free)
$X           # Unlock machine
G91          # Relative mode
G0 Z10 F100  # Move Z axis 10mm at 100mm/min
$J=Z10F100   # Jog mode (alternative)
```

## Diagnostic Symptoms Guide

| Symptom | Likely Cause | Solution |
|---------|-------------|----------|
| Motor locked after $MD | Disable pin not inverted | Add `:low` or `:high` to disable_pin |
| Commands return `ok` but no movement | Wrong addr or wrong pins for physical connector | Match addr AND all 3 pins to connector position |
| Motor vibrates/buzzes | One wire loose or wrong coil pairing | Check all 4 wires tight in terminal block |
| "Active limit switch" warning | Limit pin logic inverted or floating | Change `:low`/`:high` or set to `NO_PIN` |
| Motor weak/no holding force | run_amps or hold_amps too low | Increase based on motor rating |

## Wire Connection (4-wire Bipolar Motor)
Motor has 2 coils (A and B). Use multimeter to identify pairs (few ohms resistance between pair, infinite between different coils).

Connect to terminal blocks:
```
[Coil B-] [Coil B+] [Coil A+] [Coil A-]
  Blue     Red       Black    Green   (typical colors, verify with meter)
Exact order within coil doesn't matter initially - can reverse one coil pair if motor spins wrong direction.
Full Working Config Example (Z Motor on 3rd Connector)
yamlz:
  steps_per_mm: 395.8
  max_rate_mm_per_min: 1000
  acceleration_mm_per_sec2: 40
  max_travel_mm: 200
  motor0:
    limit_neg_pin: NO_PIN
    limit_pos_pin: NO_PIN
    tmc_2209:
      uart_num: 1
      addr: 2                    # 3rd connector = addr 2
      r_sense_ohms: 0.11
      run_amps: 1.2
      hold_amps: 0.5
      microsteps: 16
      run_mode: CoolStep
      use_enable: true
      direction_pin: i2so.9      # Must match addr 2
      step_pin: i2so.10          # Must match addr 2
      disable_pin: i2so.8:low    # Must match addr 2
Common Mistakes to Avoid

Changing addr without changing pins - Both must match physical connector
Using $$ to check TMC2209 settings - Use $CD instead
Trying to set TMC2209 params via $ commands - Only via config file
Forgetting to $Restart after config changes - Required for TMC2209 changes
Not testing MD/MD/
MD/ME before movement
 - Enable/disable must work first

Task for Claude Code
Build a complete FluidNC 4X config file with proper TMC2209 motor configurations for X, Y, Z, and optionally A axes, ensuring addr and all pins (step, direction, disable) match the physical connector positions according to the table above.