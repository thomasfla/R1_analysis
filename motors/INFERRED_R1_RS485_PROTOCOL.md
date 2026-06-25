# Unitree R1 Motor Bus Protocol

## Link

- 6,000,000 baud rs485
- Transaction: 20-byte command, then 26-byte response
- One full transaction is about 85us per motor
- control frequency is measured at 1.25 kHz
<img width="1261" height="230" alt="image" src="https://github.com/user-attachments/assets/55b98d1a-5c2c-4b5c-a861-7600b401eb0a" />

## Command

20 bytes, controller to motor.

| Offset | Size | Type | Field |
|---:|---:|---|---|
| 0 | 2 | bytes | Header: `FE EE` |
| 2 | 1 | bitfield | Low 4 bits: local motor ID; upper bits: mode/flags |
| 3 | 1 | uint8 | Mode/flags |
| 4 | 2 | int16 LE | Torque/current command |
| 6 | 2 | int16 LE | Velocity command |
| 8 | 4 | int32 LE | Position command |
| 12 | 2 | uint16 LE | Kp |
| 14 | 2 | uint16 LE | Kd |
| 16 | 4 | uint32 LE | CRC-32 |

## Response

26 bytes, motor to controller.

| Offset | Size | Type | Field |
|---:|---:|---|---|
| 0 | 2 | bytes | Header: `FC EE` |
| 2 | 1 | bitfield | Low 4 bits: local motor ID; upper bits: mode/status |
| 3 | 1 | uint8 | Temperature 1 |
| 4 | 1 | uint8 | Temperature 2 |
| 5 | 1 | uint8 | Type/status |
| 6 | 2 | int16 LE | Measured torque/current |
| 8 | 2 | int16 LE | Measured velocity |
| 10 | 4 | int32 LE | Measured position |
| 14 | 6 | bytes | Reserved; observed zero |
| 20 | 2 | uint16 LE | Status |
| 22 | 4 | uint32 LE | CRC-32 |

## Connector Mapping

| Position | Assignment |
|---|---|
| Upper left | `L_ARM` |
| Upper right | `R_ARM` |
| Lower left | `WAIST_YAW` |
| Lower right | `HEAD_PITCH` |

## Local Motor Mapping

| Connector | Local ID | IDL index | Joint |
|---|---:|---:|---|
| `L_ARM` | 0 | 15 | `L_SHOULDER_PITCH` |
| `L_ARM` | 1 | 16 | `L_SHOULDER_ROLL` |
| `L_ARM` | 2 | 17 | `L_SHOULDER_YAW` |
| `L_ARM` | 3 | 18 | `L_ELBOW` |
| `L_ARM` | 4 | 19 | `L_WRIST_ROLL` |
| `R_ARM` | 0 | 22 | `R_SHOULDER_PITCH` |
| `R_ARM` | 1 | 23 | `R_SHOULDER_ROLL` |
| `R_ARM` | 2 | 24 | `R_SHOULDER_YAW` |
| `R_ARM` | 3 | 25 | `R_ELBOW` |
| `R_ARM` | 4 | 26 | `R_WRIST_ROLL` |
| `WAIST_YAW` | 0 | 13 | `WAIST_YAW` |
| `HEAD_PITCH` | 0 | 29 | `HEAD_PITCH` |

IDs are local to each connector.

## CRC-32

- Polynomial: `0x04C11DB7`
- Initial value: `0xFFFFFFFF`
- Final XOR: none
- Input is read as little-endian `uint32` words
- Each word is processed from bit 31 to bit 0
- CRC is stored little-endian

Command CRC:

```text
crc32_core(command[0:16])
```

Response CRC:

```text
crc32_core(response[2:22])
```
The response CRC excludes the `FC EE` header.

