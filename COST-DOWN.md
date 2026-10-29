# Budget Duck — getting Open Duck Mini v2 near $100

Source analysed: apirrone/Open_Duck_Mini `v2` (commit b23317a, 2026-01-31, Apache-2.0),
and apirrone/Open_Duck_Mini_Runtime, BOM = the project's
[Google Sheet](https://docs.google.com/spreadsheets/d/1gq4iWWHEJVgAA_eemkTEsshXqrYlFxXAPwO515KpCJc/edit?usp=sharing).
Prices below are USD estimates from AliExpress-style listings, Oct 2026 — verify before ordering.

## 1. Where the money goes today

Upstream BOM total: **€398** (≈ $430), €432 with the "expression package".

| Item | Qty | € | Share |
|---|---|---|---|
| Feetech STS3215 7.4 V smart servos | 14 | 196 | **49 %** |
| BNO055 IMU | 1 | 40 | 10 % |
| Raspberry Pi Zero 2W + SD card | 1 | 36 | 9 % |
| Power (2×18650, holder, BMS, UBEC, switch, USB-C charger, jacks, XT30) | — | 52 | 13 % |
| Cables, sheath, bearings, inserts, PLA | — | 58 | 15 % |
| Rest (servo adapter, 9 g antenna servos, foot switches) | — | 16 | 4 % |

**The servos are the whole problem.** 14 smart servos at any price above ~$5 each blow the
budget alone. Nothing else matters until that is solved.

The joint layout is 5 per leg (hip yaw / roll / pitch, knee, ankle) = 10, plus a 4-DOF neck/head,
plus 2 antenna 9 g servos.

## 2. What the code actually needs (this is where the easy savings are)

From the runtime's `scripts/v2_rl_walk_mujoco.py` `get_obs()`, the walking policy only consumes:

- **gyro + accelerometer** — not the BNO055's fused orientation. Any $2 6-axis IMU
  (BMI160 / ICM-42688 / MPU-6500) gives the same inputs. **−€38 with zero downside.**
- joint positions and velocities (velocity can be differentiated from position)
- 2 foot contact switches, the user command, and its own previous actions
- control loop at **50 Hz**; the policy is a ~880 KB fp32 ONNX MLP (~220 k params), small
  enough for an ESP32-S3 with PSRAM — no Linux SBC or SD card required.

## 3. Four ways to cut it

| | A. "Duckling" (target) | B. Smart-servo Duckling | C. Full-size, trimmed | D. Duck-shaped Otto |
|---|---|---|---|---|
| Height | ~22 cm (≈ 0.55 scale) | ~22 cm | 42 cm (unchanged) | ~12 cm |
| Leg actuators | 10× hobby servo, pot-wiper feedback mod | 10× Feetech SC09 / SCS0009 bus servo | 10× STS3215 | 4× SG90 (hip + ankle, no knees) |
| Head | 2 DOF (yaw + pitch), 9 g servos | 2 DOF, 9 g | 2 DOF, 9 g | fixed |
| Brain | ESP32-S3 N16R8 | ESP32-S3 N16R8 | Pi Zero 2W (keeps upstream runtime) | Arduino Nano clone |
| **Est. total incl. charging nest** | **≈ $93** | **≈ $165** | **≈ $230** | **≈ $38** |
| Walks well? | Hardest sim2real (backlash, deadband, no torque control) | Good — same kind of actuator as upstream | Yes, closest to upstream | Scripted shuffle; works out of the box, no balance |

**Recommendation: build A as the ~$93 duck, and design its servo mounts so B is a drop-in
upgrade** (SC09 and MG90S share roughly the same 23 × 12 mm body), so anyone who can't get A
walking can swap to bus servos without reprinting the body.

Option C exists to show the floor of the full-size design: even trimmed to 12 servos it can't get
much under ~$230, because 10 leg STS3215s are ~$130–150 on their own.

Option D (§7) is a different robot — Otto DIY's 4-servo biped wearing a duck shell. It is the
cheapest and the only one that walks the day it is assembled, but it shuffles rather than walks.

## 4. Option A bill of materials (~$93)

| Part | Qty | Est. $ | Notes |
|---|---|---|---|
| MG90S metal-gear micro servo | 6 | 10 | hip yaw, hip roll, ankles |
| Stronger micro servo (MG92B-class, ~3.5 kg·cm) | 4 | 16 | hip pitch + knees carry the load |
| SG90 9 g servo | 2 | 3 | head yaw + pitch |
| ESP32-S3 N16R8 dev board | 1 | 7 | 8 MB PSRAM holds the policy |
| PCA9685 16-ch PWM board | 1 | 2.5 | servo outputs |
| CD74HC4067 16-ch analog mux | 1 | 1 | reads 10 pot wipers on one ADC1 pin |
| 6-axis IMU (BMI160 / ICM-42688) | 1 | 2.5 | replaces €40 BNO055 |
| Foot contact microswitches | 2 | 1 | |
| 2S 7.4 V 800 mAh LiPo | 1 | 8 | ~45 g vs ~95 g for 2×18650 |
| 2S BMS with balance | 1 | 1.5 | |
| 6 V 5 A UBEC / buck | 1 | 3 | servo rail; ESP32 from its own LDO |
| Slide switch, JST/Dupont, wire, M2 screws | — | 6 | |
| Small bearings (hip yaw) | 2 | 2 | |
| PLA ~250 g + TPU feet | — | 4 | |
| 2 LEDs + resistors (eyes) | — | 0.5 | |
| **Duck subtotal** | | **≈ $68** | |
| Inductive pair (XKT-510-style Tx + Rx coils/boards) | 1 | 5 | see §5 |
| 5 V → 2S boost charger board | 1 | 1.5 | charges the pack from the Rx 5 V |
| Nest: USB-C 5 V input board + PLA + magnets | — | 3 | powered by any phone charger |
| **Nest subtotal** | | **≈ $9.50** | |
| Contingency (shipping, a spare servo, breakage) | | 15 | servos *will* die during tuning |
| **Total** | | **≈ $93** | |

### The pot-wiper feedback mod (what makes cheap servos usable)

Each MG90S already has a position potentiometer inside. Open the case, solder a thin wire to the
pot's middle (wiper) pin, route it out with the servo cable. That gives an analog voltage
proportional to the actual shaft angle — real joint position feedback for $0. The mux feeds all 10
into one ADC pin; at 50 Hz there is plenty of time to scan them. Velocity = filtered difference
of positions, which is what the policy normalizes anyway (`dof_vel * 0.05`).

### Torque sanity check

At 0.55 scale with an ~450 g robot, single-leg stance with the knee bent puts roughly
0.45 kg × 5 cm ≈ **2.2 kg·cm** on the knee — right at an MG90S's stall rating. That is why the
4 hip-pitch/knee servos are upgraded to ~3.5 kg·cm parts and why the battery is a LiPo instead of
18650s. If it still struggles, scale to ~20 cm before spending more on servos.

## 5. Inductive charging nest

**Choice: a non-Qi inductive module pair (XKT-510 class, 5 V / ~1–2 A) rather than Qi.**
Qi receivers need ~5 mm gap and tight alignment and expect a phone-style handshake; the generic
XKT pairs tolerate 10–20 mm and a few mm of misalignment, which matters when a walking robot sits
down imprecisely. They also cost less.

Power path:

```
USB-C 5 V phone charger ─► nest Tx board + coil (under a 1.2 mm PLA skin)
                              ⋮  ~5–10 mm air/plastic gap
duck belly Rx coil ─► Rx board 5 V ─► 5V→2S boost charger ─► 2S BMS ─► 2S LiPo
```

- Charge rate ~0.4–0.5 A into 800 mAh → **~1.5–2 h** full charge. Run time estimate 15–25 min of
  walking.
- Rx coil sits flat in the bottom of the body shell, centered between the hips. In the default
  crouched "sit" pose the belly is the lowest point, so the duck sits *on* the coil.
- **Nest shape:** a shallow printed bowl with two foot-shaped recesses and a raised center pad
  holding the Tx coil. The cone of the bowl self-centers the duck as it sits; a pair of 6×2 mm
  magnets (nest + belly) snap the last few mm of alignment.
- Charge detect: wire the boost charger's CHRG LED line to an ESP32 GPIO so firmware knows it is
  docked (and can go to a low-power idle with servos limp).
- Stretch goal, $1 more: an IR LED in the nest + IR receiver in the head so the duck can find and
  back into the nest on its own.

## 6. What it costs in work (the hidden price of $100)

The parts list is the easy part. Option A means:

1. **Re-CAD the body.** STLs can't simply be scaled — every servo pocket is sized for STS3215.
   Remix the Onshape model (Apache-2.0, fine to remix) at ~0.55 scale around MG90S-size pockets.
   Upstream head is 200 mm and needs a ≥ 200 mm bed; at 0.55 scale every part fits a 180 mm bed.
2. **New robot model.** Export a new MJCF for MuJoCo with the scaled masses.
3. **Identify the actuators.** Upstream used Rhoban BAM on the STS3215; do the same on the
   modded MG90S (backlash + deadband will be the big terms).
4. **Retrain the walk policy** in `Open_Duck_Playground` (MuJoCo Playground / JAX). Training wants
   a GPU; CPU-only training is possible but slow. Sim2real with hobby servos is the riskiest step in this plan.
5. **Port the runtime** from Python on the Pi to ESP32-S3 firmware (C++ with an ONNX→TFLite-Micro
   or hand-written MLP; the network is a plain MLP so the hand-written route is realistic).

Option B removes the risk in step 3/4 (bus servos with real position control) for ~$70 more;
Option C removes steps 1–5 entirely but stays well over $200.

**Shortcut borrowed from Otto:** Option A does not have to use a learned policy. Hand-tuned,
scripted gaits (the way OttoDIYLib walks) drop steps 2–4 and most of 5 — the ESP32 just plays
oscillator patterns per joint, with the IMU used only to stop on a fall. The walk is stiffer than
the RL gait, but the $93 duck becomes a weekend firmware job instead of a research project, and
the RL route stays open later on the same hardware.

## 7. Option D — duck-shaped Otto (~$38)

[Otto DIY](https://www.ottodiy.com/) is an open-source 4-servo biped (2 hips, 2 ankles) on an
Arduino Nano with an HC-SR04 ultrasonic sensor and a buzzer. Its gaits are pre-scripted in
[OttoDIYLib](https://github.com/OttoDIY/OttoDIYLib), so there is no simulation, training, or
tuning. Print files are on Printables, and every part fits a 180 mm bed.

**Licence:** the Otto design files are CC-BY-SA — a duck remix must credit ottodiy.com and stay
CC-BY-SA (commercial use is allowed, share-alike). The library README also mentions GPL, so check
each repo's licence file before reusing code.

**The duck part:** keep Otto's internal body box, servo pockets, and leg/foot geometry unchanged
(so stock gaits and calibration still work), and replace only the outer head/body shell with a
duck: bill over the ultrasonic "eyes" (the two transducers stay exposed as the eyes), a tail
on the back, and LED eye-highlights.

### Bill of materials

| Part | Qty | Est. $ | Notes |
|---|---|---|---|
| Arduino Nano clone (USB-C) | 1 | 3.5 | |
| Nano I/O shield | 1 | 2 | plug-in servo headers, no soldering |
| SG90 9 g servo | 4 | 6 | MG90S metal-gear is ~$2 more if gears strip |
| HC-SR04 ultrasonic | 1 | 1.5 | the eyes, and the docking range-finder |
| Passive buzzer | 1 | 0.5 | |
| 1S 3.7 V 1000 mAh LiPo | 1 | 4.5 | replaces Otto's 4×AA |
| TP4056 charger board (with protection) | 1 | 0.5 | swap R_PROG to 2.4 kΩ → 500 mA charge |
| 5 V 2 A boost board | 1 | 1.5 | servos + Nano run from 5 V |
| 1000 µF capacitor on the servo rail | 1 | 0.3 | stops servo current spikes resetting the Nano |
| Slide switch, wire, screws | — | 3 | |
| PLA ~80 g (Otto parts + duck shell) | — | 1.5 | |
| 2 LEDs + resistors | — | 0.5 | |
| **Duck subtotal** | | **≈ $25** | |
| Inductive pair (XKT-510-style Tx + Rx) | 1 | 5 | same module as §5 |
| Nest: USB-C 5 V input board + PLA + magnets | — | 3 | |
| **Nest subtotal** | | **≈ $8** | |
| Contingency | | 5 | |
| **Total** | | **≈ $38** | |

Run time estimate ~45–60 min walking; full charge ~2.5 h at 500 mA.

### Charging nest for Otto

The 1S pack makes the nest simpler than §5: the Rx board's 5 V goes straight into the TP4056 —
no boost charger, no BMS.

```
USB-C 5 V ─► nest Tx coil (under the right-foot pocket, 1.2 mm PLA skin)
                  ⋮  ~2–4 mm
right-foot Rx coil ─► Rx board 5 V ─► TP4056 ─► 1S LiPo ─► switch ─► 5 V boost ─► Nano + servos
```

- **Coil goes in the right foot sole.** Otto has no knees and can't sit, so the feet are the only
  part that reliably meets the floor. A 20–30 mm Rx coil fits the sole; the wire runs up past the
  ankle servo with a service loop. A 6×2 mm magnet in each sole + the nest pulls the last mm.
- **Nest shape:** a low pad (≤ 3 mm with chamfered entry so Otto can shuffle onto it) with two
  shallow foot pockets and a back wall.
- **Self-docking for $0:** the back wall is a target for the ultrasonic. Otto walks toward the nest
  and stops at a fixed range (calibrated once, ~6 cm), which puts its feet over the pockets.
- Charge detect: TP4056 CHRG pin → Nano GPIO, so the firmware knows it docked and can play a
  happy sound and rest the servos.

## 8. Do not use

Microduck's 3D files (Pollen Robotics) are reported as CC BY-SA-NC, and the community
"microduck-replica" is a reverse-engineered copy with no commercial licence. Remix only the
Apache-2.0 Open Duck Mini.
