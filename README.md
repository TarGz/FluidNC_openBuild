# FluidNC Configuration - OpenBuild Acro 1510

FluidNC 4X 2209 V3 board configuration for OpenBuild Acro 1510 CNC machine.

## Current Version: 0.0.1

### Machine Specifications
- **Model**: OpenBuild Acro 1510
- **Controller**: FluidNC 4X 2209 V3 (TMC2209 drivers)
- **Working Area**: 1270mm (X) x 810mm (Y) x 200mm (Z)
- **Drive System**: GT2/GT3 Belt Drive

### Motor Configuration
- **X-axis**: NEMA 17, 1.0A, 57.300 steps/mm
- **Y-axis**: NEMA 17 (dual motors), 1.0A, 57.143 steps/mm
- **Z-axis**: NEMA 11, 0.4A, 500.000 steps/mm

### Motion Parameters
- **X/Y Max Speed**: 5000 mm/min
- **X/Y Acceleration**: 100 mm/s²
- **Z Max Speed**: 500 mm/min
- **Z Acceleration**: 10 mm/s²

## Deployment

### Upload to FluidNC Board

1. Connect to FluidNC web interface: `http://192.168.1.100`
2. Go to **Files** tab → **Flash** section
3. Upload `config.yaml` (will overwrite existing)
4. Restart the board

Or use the deployment script:
```bash
./deploy_config.sh 192.168.1.100
```

## Version History

### v0.0.1 (2025-11-09)
- Initial configuration based on GRBL settings from working Acro 1510
- Configured TMC2209 drivers for all axes
- Set NEMA 11 Z-axis to conservative 0.4A current
- Enabled homing on all axes (positive direction)
- Steps/mm from original GRBL: X=57.3, Y=57.143, Z=500
