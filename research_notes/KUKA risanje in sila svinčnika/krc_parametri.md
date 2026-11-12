# KUKA KR C (KRC1/KRC2/KRC3) motion parameters, defaults and kinematic limits for simulating a small KUKA (KR 3 / KR 6 class) pencil-drawing robot

Source-generation key: **[KSS5.2]** = KUKA "Expert Programming, KSS Release 5.2, KR C2/KR C3" (issued 26 Sep 2003); **[CFG5.2]** = KUKA "Configuration, KSS Release 5.2, KR C2/KR C3" (issued 05 Aug 2005); **[REPO]** = real controller files (`$config.dat`, `bas.src`, `$machine.dat`) from a KRC2-era KSS installation published on GitHub (itk-thrivaldi/krl, "floor" robot; the robot type is NOT a KR 3; its numbers show the structure of the files, not KR 3 values).

## 1. Default values after BAS(#INITMOV,0) / INI fold

### Takeaway
BAS(#INITMOV,0) copies the `DEF_*` values from `$CONFIG.DAT` into the motion variables. With a stock KRC2 `$config.dat` that gives: PTP `$VEL_AXIS[i]=100 %`, `$ACC_AXIS[i]=DEF_ACC_PTP` (50 % in the sample file; 100 % when acceleration adaptation is on), `$VEL.CP=2.0 m/s`, `$ACC.CP=2.3 m/s²`, `$VEL.ORI1=$VEL.ORI2=200 °/s`, `$ACC.ORI1=$ACC.ORI2=100 °/s²`, `$ORI_TYPE=#VAR`, `$CIRC_TYPE=#BASE`, `$ADVANCE=3`. `$OV_PRO` is not set by INITMOV. `DEF_OV_PRO=100` sits in `$config.dat`, and the operator can change override from the pendant.

### Cited Findings
- `$config.dat` (KRC2-era) contains: `INT DEF_OV_PRO=100`, `INT DEF_ADVANCE=3`, `INT DEF_VEL_PTP=100`, `INT DEF_ACC_PTP=50`, `DEF_CIRC_TYP=#BASE`, `REAL DEF_VEL_CP=2.0`, `DEF_VEL_ORI1=200.0`, `DEF_VEL_ORI2=200.0`, `DEF_VEL_ORIS=200.0`, `DEF_ACC_CP=2.29999995`, `DEF_ACC_ORI1=100.0`, `DEF_ACC_ORI2=100.0`, `DEF_ACC_ORIS=200.0`, `DEF_VEL_FACT=1.0`; approximation `DEF_APO_CPTP=50`, `DEF_APO_CVEL=100`, `DEF_APO_CDIS=3.0` (mm), `DEF_APO_CORI=5.0` (°) — [REPO $config.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/System/$config.dat)
- Inline-form (ILF) defaults in the same file: `PDEFAULT={VEL 100.0,ACC 100.0,APO_DIST 100.0}`, `LDEFAULT={VEL 2.0,ACC 100.0,APO_DIST 100.0,APO_FAC 50.0,ORI_TYP #VAR}` — [REPO $config.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/System/$config.dat)
- `DEF_ACC_CP=2.29999995` with `DEF_VEL_CP=1.0` also appears in a `$config.dat` quoted on the Siemens Process Simulate forum. DEF_VEL_CP is a site/robot-specific setting, so 2.0 is a typical value but not a universal one — [Siemens PLM forum (via search summary; page returns 403)](https://community.plm.automation.siemens.com/t5/Tecnomatix-Robotics-Forum/Process-Simulate-KUKA-machine-data/td-p/286792)
- The body of `INITMOV()` in `bas.src`: `ACC_PTP(100.0)` ("100% = DEF_ACC_PTP"), `VEL_PTP(100.0)` ("100% = DEF_VEL_PTP"), `ACC_CP(100.0)`, `$ACC.ORI1=DEF_ACC_ORI1`, `$ACC.ORI2=DEF_ACC_ORI2`, `VEL_CP(DEF_VEL_CP)`, `$VEL.ORI1=DEF_VEL_ORI1`, `$VEL.ORI2=DEF_VEL_ORI2`, `$APO.CPTP/CDIS/CVEL/CORI = DEF_APO_*`, `$FILTER=$DEF_FLT_PTP`, `$ADVANCE=DEF_ADVANCE`, `$ORI_TYPE=#VAR`, `$CIRC_TYPE=DEF_CIRC_TYP`, all correction frames reset to `$NULLFRAME`, `BASE(0)` (= $WORLD), `$TOOL=TOOL_DATA[1]` — [REPO bas.src](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/System/bas.src)
- Scaling logic in `bas.src`: `VEL_PTP(f)` sets `$VEL_AXIS[i]=DEF_VEL_PTP*f/100` (capped at DEF_VEL_PTP when f>100). `ACC_PTP(f)` sets `$ACC_AXIS[i]=DEF_ACC_PTP*f/100` only if `$ADAP_ACC==#NONE`; otherwise `$ACC_AXIS[i]=f` (so 100 %). `ACC_CP(f)` sets `$ACC.CP=DEF_ACC_CP*f/100` and scales ORI1/ORI2 the same way. `VEL_CP(v)` sets `$VEL.CP=v*DEF_VEL_FACT`, capped at DEF_VEL_CP — [REPO bas.src](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/System/bas.src)
- Units, per KUKA: `$VEL.CP` REAL m/s (path velocity); `$VEL.ORI1` REAL °/s (swivel velocity); `$VEL.ORI2` REAL °/s (rotational velocity); `$VEL_AXIS[4]–[6]` INT %; `$ACC.CP` REAL m/s²; `$ACC.ORI1/ORI2` REAL °/s²; `$ACC_AXIS` INT % — [KSS5.2 Expert Programming, Table 4, p. 68–69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `$VEL_AXIS[]` and `$ACC_AXIS[]` are "given as percentages of the maximum value defined in the machine data". If they are not programmed for all axes, the program stops with an error message — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KUKA says the initialization sequence presets CP velocities and accelerations "to the maximum values defined in the machine data or $CONFIG.DAT" — [KSS5.2 p. 69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `$ORI_TYPE=#VAR` and `$CIRC_TYPE=#BASE` are set by BAS(#INITMOV,0). The system's own default for `$CIRC_TYPE` is #PATH — [KSS5.2 p. 70–75, Table 5](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `DEF_ACC_CP` is the "current setting for the maximum acceleration", and it can be changed from the menu. The higher the value, the more likely the controller's monitoring functions trip and stop the robot with an error message — [CFG5.2 p. 26–27](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- Other machine-data limits: `$ACC_MA={CP 10.0, ORI1 1000.0, ORI2 1000.0}` (max path acceleration in m/s², swivel and rotation in °/s²), `$DEF_OV_JOG=10` (default jog override 10 %), `$RED_T1=7` (T1 reduction factor %, "MAX: 15%"), `$VEL_CP_T1=0.1` m/s ("MAX: 0.25") — [REPO $machine.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/Mada/$machine.dat)

### Inferences
- Simulator defaults: `$VEL.CP=2.0 m/s`, `$ACC.CP=2.3 m/s²`, `$VEL.ORI1/2=200 °/s`, `$ACC.ORI1/2=100 °/s²`, `$VEL_AXIS=100 %`, `$ACC_AXIS=100 %` (or 50 % if you model the stock `DEF_ACC_PTP=50` with `$ADAP_ACC=#NONE`), `$OV_PRO=100 %` in Auto. A real T1 test run is capped at 250 mm/s TCP speed (`$VEL_CP_T1` ≤ 0.25 m/s). The sample file uses 0.1 m/s.
- The effective path speed is roughly `$VEL.CP × $OV_PRO/100`, further capped by T1 limits. An ILF `LIN P1 Vel=0.2 m/s` calls `BAS(#VEL_CP,0.2)`.
- The approximation defaults matter for the drawing: with `C_DIS`, blending starts 3 mm before the point (`DEF_APO_CDIS=3.0`), so corners get rounded.

### Gaps
- No KUKA document states factory DEF_* values per robot type. They vary per installation (DEF_VEL_CP=1.0 vs 2.0 seen).
- KRC1 (KSS 2.x/3.x) defaults were not found. The BAS.SRC structure is believed to be the same, but this is unverified.
- The exact `$OV_PRO` value at program start is not documented here. On a real controller it keeps the last value the operator set on the KCP.

## 2. PTP motion: synchronous PTP, velocity profile, max axis speeds

### Takeaway
KRC PTP is synchronous: all axes start and stop together. The leading axis (the one needing the longest time) runs at its programmed `$VEL_AXIS`/`$ACC_AXIS` limits, and the others are slowed to match. With `$ADAP_ACC=#STEP1` and the higher motion profile, the axes are also phase-synchronous (accelerate, cruise and decelerate together). The higher motion profile is torque-based, so 50 % acceleration ≠ half the acceleration. Max axis speed in °/s = motor rated rpm / gear ratio × 6. For simulation, a trapezoidal (or S-curve) profile with phase sync and per-axis vmax from the spec sheet is a fair approximation.

### Cited Findings
- "all of the axes start and stop moving at the same time … only the axis with the longest trajectory, the so-called leading axis, is actually moved with the programmed acceleration and velocity limits" — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- With `$ADAP_ACC=#STEP1` and `$OPT_MOVE=#STEP1`, the axes "enter the acceleration, constant motion and deceleration phases together" (phase-synchronous), so the path always follows the same course whatever the velocity or acceleration — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "The higher motion profile is used as standard for PTP motions": a time-optimized motion where the permissible torque range is used optimally. The acceleration % "relates … to the acceleration torque of the axis and not directly to the acceleration itself" — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KUKA recommends identical velocity and acceleration values for all axes, because the leading axis is unknown for Cartesian PTP targets — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Machine-data structure (KRC2, a larger robot; illustrative only): `$VEL_AXIS_MA[i]` = motor rated speed in rpm (3250, 3250, 3250, 4200, 4200, 2850); `$RAT_MOT_AX[i]` gear ratios (−125, −125, 125, −690/9, 33500/441, −111/4); `$RAISE_TIME[i]` axis ramp-up time in ms (400, 800, 350, 250, 250, 260), used as the max acceleration reference when no dynamic model exists; `$IN_POS_MA` positioning window 0.1–0.2 mm/° — [REPO $machine.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/Mada/$machine.dat)
- `$ACC_AXIS` % refers to the max acceleration "defined by means of $RAISE_TIME in the machine data" when no dynamic model exists — [KSS 8.x System Variables (via search summary)](http://www.wtech.com.tw/public/download/manual/kuka/krc4/KUKA%20System%20Variables%208.1%208.2%208.3.pdf)
- **KR 3 (older model, 3 kg, 635 mm, sold with KR C3 ≈ 2005), third-party reseller data:** axis speeds A1 240, A2 210, A3 240, A4 375, A5 300, A6 375 °/s — [Eurobots KR 3](https://www.eurobots.net/kuka-robots-kr-3-p248-en.html). UNVERIFIED against a KUKA sheet.
- **KR 6/2 (6 kg, 1611 mm, KRC1/KRC2 era), dealer data:** A1–A3 156 °/s, A4 343, A5 362, A6 659 °/s (another dealer: A5 365, A6 662) — [robots.com KR 6-2 / Reprobots / Robots Done Right, via search summary](https://www.robots.com/industrial-robots/kuka-kr-6-2); [Reprobots](https://www.reprobots.com/product-page/kuka-kr-6-2). Note that 3250 rpm / 125 × 6 = 156 °/s matches the A1–A3 value of the KRC2 `$machine.dat` above. That file is likely a KR 6/2-KR 16 family robot, whose ranges (±185, +35/−155, +154/−130, ±350, ±130, ±350) also match the KR 6/2. This is an inference.
- **KR 6 R900 sixx (AGILUS, KR C4, comparison only):** A1 360, A2 300, A3 360, A4 381, A5 388, A6 615 °/s; reach 901 mm; ±0.03 mm; ≈52 kg — [KUKA AGILUS sales sheet via search summary](https://www.kuka.com/-/media/kuka-downloads/imported/6b77eecacfe542d3b736af377562ecaa/pb0001_kr_agilus_en.pdf); [Spez KR AGILUS sixx](https://ntnu.no/wiki/download/attachments/332336040/Spez_KR_AGILUS_sixx_en.pdf?api=v2)

### Inferences
- A simulator model: for each axis, Δq_i; t_i = time for a trapezoid with v_i = vmax_i·VEL%/100·OV/100 and a_i = vmax_i/t_raise_i·ACC%/100. T = max t_i. Scale the other axes so they finish at T (phase-sync: same accel/cruise/decel fractions). As a ramp-time guess for a small robot, take 0.25–0.4 s to vmax (from `$RAISE_TIME` of the wrist and main axes of the sample robot). This is an order-of-magnitude figure, flagged.
- T1 jogging/test: KUKA T1 limits TCP to 250 mm/s. `$RED_T1` (e.g. 7 %) reduces axis speed for PTP in T1.

### Gaps
- No original KUKA spec sheet for the old KR 3 (635 mm) was found. The speeds above come from one reseller.
- `$RAISE_TIME`, `$VEL_AXIS_MA` and gear ratios for KR 3 were not found.

## 3. LIN/CIRC path velocity profile; how $VEL.CP limits TCP speed

### Takeaway
For CP motions the TCP moves at `$VEL.CP` (m/s) with ramp `$ACC.CP` (m/s²). Orientation changes are limited by `$VEL.ORI1/ORI2` (°/s). At any instant at least one component (translation, swivel, rotation or wrist axes) runs at its programmed limit, and the rest are adapted phase-synchronously. So a LIN with a large orientation change can be slower than `$VEL.CP`. Axis velocity and acceleration are still monitored (`$VEL_ACT_MA`, `$ACC_ACT_MA`). Near the wrist singularity this trips a braking stop unless `$CP_VEL_TYPE` reduction is active.

### Cited Findings
- CP velocities and accelerations "relate … to the motion of the TCP. The TCP is thereby moved at a precisely defined velocity" — [KSS5.2 p. 68](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "At any given moment … at least one of the motion components -- translation, swiveling, rotation or Axis[4]...[6] -- is being carried out with programmed acceleration and velocity. The non-dominant components are adapted phase-synchronously." — [KSS5.2 p. 69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Axis velocity and acceleration are monitored in CP against `$VEL_ACT_MA` and `$ACC_ACT_MA`. If exceeded, "a braking reaction is triggered and an error message is generated", e.g. near the wrist singularity — [KSS5.2 p. 69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf). The sample machine data has `$VEL_ACT_MA=110` % and `$ACC_ACT_MA=250` % — [REPO $machine.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/Mada/$machine.dat)
- `$CP_VEL_TYPE` defaults to `#CONSTANT` (no reduction in program mode). `#VAR_T1` reduces in T1 and `#VAR_ALL` in all modes; "Reduction is always active in jog mode". `$CPVELREDMELD=1` gives a message when the path velocity is reduced — [KSS5.2 p. 69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `$ORI_TYPE=#CONSTANT` keeps the start orientation; `#VAR` (INITMOV default) interpolates orientation; `#JOINT` interpolates wrist axis angles linearly and avoids the wrist singularity problem — [KSS5.2 p. 70–71](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- For LIN, S and T of the end point are ignored (the status of the start point is kept). A complete PTP (with S/T, or axis-specific) must come before the first LIN — [KSS5.2 p. 75](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `BAS(#VEL_CP,v)` caps `$VEL.CP` at `DEF_VEL_CP` (e.g. 2.0 m/s) — [REPO bas.src](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/System/bas.src)
- A real KUKA pen/drawing project ("kukodraw") generates KRL with `BAS(#INITMOV,0)`, `BAS(#CP_PARAMS,2.0)` on LIN blocks, and pen down/up as Z −17.64 mm / Z −15 mm (a 2.64 mm lift) — [kukodraw kuka.py](https://github.com/daniel-kurushin/kukodraw/blob/master/kuka.py)

### Inferences
- Simulator: 1-D trapezoid along path length s with v = min($VEL.CP·OV, v_ori-limited, axis-limited) and a = $ACC.CP. For a short drawing segment (e.g. 10 mm at 0.2 m/s with 2.3 m/s²), the ramp distance is v²/2a ≈ 8.7 mm per side, so short strokes never reach the programmed speed. Without approximation (no C_DIS) the robot stops exactly at every point (and checks `$IN_POS_MA`).
- 2.3 m/s² is the 100 % default. ILF `ACC=100` gives 2.3 m/s².

### Gaps
- Jerk limits and the exact CP profile shape (KRC2 uses a smoothed/filtered ramp via `$FILTER`/`$DEF_FLT_CP`) are not documented numerically here.

## 4. Kinematic data of KR 3 (and old KR 6)

### Takeaway
The older KR 3 (KR C3 era, 635 mm reach, 3 kg, ≈53 kg) has no public DH table I could find. Its reseller-listed ranges are A1 ±180, A2 −45/+135, A3 −225/+45, A4 ±180, A5 ±135, A6 unlimited (unverified). The modern KR 3 R540 AGILUS does have published data: 345/20/260/20/260/75 mm, reach 541 mm. The KR 6/2 is a much larger arm (1611 mm, 235 kg), so it is not a "small" pencil robot.

### Cited Findings
- KR 3 (older): 3 kg payload, 635 mm reach, ±0.05 mm repeatability, ≈53 kg — [RoboDK KR 3](https://robodk.com/robot/KUKA/KR-3); [Robotverse](https://www.robotverse.in/directory/robot/kuka-kr-3)
- KR 3 (older) reseller data: controller KR C3 ("the compact version of the KR C2"), year 2005; ranges A1 ±180°, A2 −45°…+135°, A3 −225°…+45°, A4 ±180°, A5 ±135°, A6 unlimited — [Eurobots KR 3](https://www.eurobots.net/kuka-robots-kr-3-p248-en.html). The A2/A3 zero convention differs from the KUKA standard (A2 = −90° horizontal), so treat these with caution.
- KR 3 R540 AGILUS (KR C4 compact / KR C5 micro, 2013+, comparison): reach 541 mm, ≈26.5 kg; A1 ±170°, A2 −170°/+50°, A3 −110°/+155°, A4 ±175°, A5 ±120°, A6 ±350° — [KUKA datasheet 0000270971 (via search summary)](https://www.kuka.com/-/media/kuka-downloads/imported/6b77eecacfe542d3b736af377562ecaa/0000270971_en.pdf)
- KR 3 R540 modified DH (Khalil) from a university tutorial: base height r1 = 345 mm, d2 = 20 mm (A1–A2 offset), d3 = 260 mm (upper arm), d4 = 20 mm (A3–A4 offset), r4 = 260 mm (forearm), flange r7 = 75 mm; spherical wrist — [Univ. Angers tutorial](https://perso-laris.univ-angers.fr/~boimond/Tutorial_2_Inverse_Geometric_Model_Kuka_KR3_R540.pdf)
- KR 6/2: 6 kg, 1611 mm reach, ±0.05 mm, 235 kg; ranges A1 ±185°, A2 +35°/−155°, A3 +154°/−130°, A4 ±350°, A5 ±130° (one dealer ±135°), A6 ±350° — [RoboDK KR 6-2](https://robodk.com/robot/KUKA/KR-6-2); [Reprobots (via search summary)](https://www.reprobots.com/product-page/kuka-kr-6-2)
- Software limit switch values of a KRC2 robot with the same ranges: `$SOFTN_END` −185, −155, −130, −350, −130, −350 and `$SOFTP_END` 185, 35, 154, 350, 130, … — [REPO $machine.dat](https://github.com/itk-thrivaldi/krl/blob/master/floor/R1/Mada/$machine.dat)
- KUKA mechanical zero / typical HOME: `{A1 0, A2 -90, A3 90, A4 0, A5 0, A6 0}` — [KSS5.2 p. 58, 62](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- For a school "small orange KUKA on a round black base, KRC1/KRC2 era", the candidates are the older KR 3 (KR C3, ~2003–2008) or KR 5 sixx/KR 6 R… (later). A KR 6/2 is a floor-standing 1.6 m-reach robot. A simulator could use the R540 geometry scaled up to 635 mm reach as a placeholder, flagged as an assumption.
- KUKA sign conventions: A2 = −90° means the upper arm is horizontal, and the A3 = +90° HOME has the forearm horizontal.

### Gaps
- No KUKA dimension drawing or DH data for the older 635 mm KR 3 was found. The RoboDK `.robot` file probably contains it, but it is a binary download and was not inspected.
- No official axis speeds or ranges for the older KR 3.

## 5. Singularities, $SINGUL_POS, Status/Turn

### Takeaway
KUKA distinguishes 3 singularities: overhead (α1: wrist point on the A1 axis), extended (α2: A2–A3 extension through the wrist point, edge of workspace) and wrist (α5: A5 ≈ 0, A4 ∥ A6). For PTP targets in α1/α5, `$SINGUL_POS[1]`/`[3]` decides whether A1/A4 go to 0° or keep the start value. In CP motion near α5 the axis speed or acceleration monitoring trips a stop. Status S (bits 0–2) picks the arm/wrist configuration. Turn T (bits 0–5) picks the sign of each axis angle.

### Cited Findings
- Three singularities. Near one, "small Cartesian changes … give rise to major changes in the axis angles" — [KSS5.2 p. 66](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Overhead (α1): the wrist root point lies on axis 1, so A1 is undetermined. For a PTP end point there, A1 either goes to 0° or keeps its start value, selected by `$SINGUL_POS[1]` (machine data) — [KSS5.2 p. 66](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Extended (α2): the robot is at the limit of its work envelope. The inverse is unique, but low Cartesian speed gives high A2/A3 speed — [KSS5.2 p. 67](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Wrist (α5): A4 ∥ A6, infinite solutions. A PTP end point within −0.01812° ≤ A5 ≤ +0.01812° either sends A4 to 0° or keeps A4, selected by `$SINGUL_POS[3]` — [KSS5.2 p. 67–68](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Status bits: bit 0 = wrist root point in the basic area (0, x of wrist point in the A1 frame > 0) vs overhead area (1); bit 1 = arm configuration (for robots whose A3 and A4 intersect, 0 if A3 < 0°, else 1; with an A3–A4 offset the switch angle depends on the offset); bit 2 = wrist (0 if 0° ≤ A5 < 180° or A5 < −180°; 1 if −180° ≤ A5 < 0° or A5 ≥ 180°) — [KSS5.2 p. 64, Table 3](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Turn bits: bit x = 0 means axis (x+1) ≥ 0°, 1 means < 0°. E.g. T 'B10011' = A1, A2 and A5 negative — [KSS5.2 p. 63, Table 2](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- If S/T are omitted in later PTPs, "the robot retains the old value for S and selects the T value which gives the shortest possible axis trajectory". The first motion must be a complete PTP (with S and T, or axis-specific). S and T are ignored in CP motions — [KSS5.2 p. 63](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- `$ORI_TYPE=#JOINT` with a status change during the motion: "the robot is stopped at the start of the main run and a corresponding error message is displayed" — [KSS5.2 p. 71](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- A typical example uses `S 2, T 35`; S=2 (binary 010) is the common "normal" arm configuration in KUKA examples — [KSS5.2 p. 62](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- For a pencil drawing on a horizontal table with the tool pointing down, A5 is often near 0 or near ±90 depending on A4/A6. A vertical tool with A4 = 0 tends to put A5 near the wrist singularity when the wrist point is directly above the TCP, so a simulator should warn at |A5| < ~1–5°. That threshold is a heuristic; KUKA only gives ±0.01812° for the PTP special case.

### Gaps
- The exact KRC2 message text for "singularity" in CP was not found. In practice it shows up as an axis velocity/acceleration monitoring stop (see §6), not a dedicated "singularity" message. KSS 8.x has messages like "Singularity A5", which are not verified for KRC2.

## 6. Typical error/stop messages

### Takeaway
Messages a student can expect on KRC2: software limit switch (A1–A6, +/−), workspace monitoring, CP velocity/acceleration monitoring stops, "CP/PTP approximation not possible", undeclared or uninitialized motion variables (e.g. $VEL_AXIS not set), the need for a BCO run (block coincidence; "dead-man" + reduced speed), and "117 Torque exceeded axis Ax" (collision monitoring). Numeric KRC2 codes are mostly unverified.

### Cited Findings
- BCO (block coincidence): before program execution the robot must match the programmed position. The BCO run is done with Start held down at automatically reduced velocity. The robot stops when it reaches the path, then Start is pressed again. No BCO run in Automatic External — [KSS5.2 p. 62](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- If `$VEL_AXIS`/`$ACC_AXIS` are not programmed for all axes, the program generates an error message — [KSS5.2 p. 57](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Message "CP/PTP approximation not possible": the robot then positions exactly at the point — [KSS5.2 p. ~84 (line ref)](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Exceeding `$VEL_ACT_MA`/`$ACC_ACT_MA` in CP gives a braking reaction and an error message — [KSS5.2 p. 69](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- The message "117 Torque exceeded axis Ax" sets `$COLL_ALARM` — [CFG5.2 p. 69](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- Software limit switches are machine protection set so the robot cannot hit the mechanical stops. The "Set software limit switches" function adds a 5° buffer to measured values — [KSS 5.5 end-user manual (via search summary)](https://www.hgpauction.com/wp-content/uploads/2017/02/KSS-5.5-Operating-and-Programming-Instructions-for-End-Users.pdf); [KSS5.2 p. 174](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KRC4 message 01750 "Axis in invalid range" (software limit switch reached or point outside the reachable workspace) is a KRC4 code, not KRC2 — [dynamic-automation blog (via search summary)](https://www.dynamic-automation.de/en/blog/kuka-error-codes)
- A blog cites KRC2 messages 1211/3056 for a software limit stop on A3. These are unverified — [industrialmonitordirect (via search summary)](https://industrialmonitordirect.com/es/blogs/knowledgebase/kuka-krc2-kr180pa-crash-diagnostics-poslog-and-logbuch-analysis)

### Inferences
- Suggested simulator texts (paraphrased in KSS style): "Software limit switch -A3", "Workspace error" / "target not reachable", "BCO run required – hold Start", "Axis velocity exceeded A4" (near the wrist singularity), "117 Torque exceeded axis A3". Label them as approximations of the real wording.

### Gaps
- No authoritative KRC2 message catalogue (numbers + exact wording) was available online. The KSS 5.x message documentation is on KUKA's customer portal.

## 7. Collision / torque monitoring: does a pencil pressing a board trigger it?

### Takeaway
On KRC2/KSS 5.x, collision monitoring is OFF by default. When enabled (requires `$ADAP_ACC=#STEP1` and correct load data), the default tunnel is 200 % of max motor torque per axis, with `$TORQMON_TIME` response time (default 0 ms). Such a wide band will not detect the few newtons of a pencil pressing on paper. Even a tuned tunnel ("measured `$TORQ_DIFF` + 5–10 %") is far above typical pencil forces. A position-controlled robot just pushes the pencil, so the pencil breaks or the board flexes instead.

### Cited Findings
- "Monitoring is switched off by default, but the default value for the monitoring tunnel is 200%"; when the torque leaves the tunnel, "a path-maintaining stop reaction is triggered after the specified response time" — [CFG5.2 §2.8.1 p. 69](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- "The width of the tolerance band is equal to the maximum torque [Nm] multiplied by the percentage in $TORQMON_..."; defaults `$TORQMON_DEF[1..6]=200`, `$TORQMON_COM_DEF[1..6]=200`; `$TORQMON_TIME=0.0` ms; requires `$ADAP_ACC=#STEP1`; load data must be accurate; tune with `$TORQ_DIFF[]` + 5–10 % — [CFG5.2 p. 69–70](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- On a collision the "command torques of the axes involved are automatically increased" and the robot "may be able to overcome the resistance and continue its motion". Monitoring "does not provide any guarantee against damage" — [CFG5.2 p. 69](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- `$COLL_ENABLE` is set when `$torqmon` < 200 %; `$COLL_ALARM` is set on "117 Torque exceeded axis Ax". External axes are not monitored — [CFG5.2 p. 69](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)
- Torque mode (soft servo, `$TORQUE_AXIS`, `$CURR_RED`) can make an axis compliant. It disables the axis's monitoring and cannot be combined with other axes moving — [CFG5.2 §2.7 p. 64–68](https://robot.zaab.org/wp-content/uploads/2014/04/KR-C2-KR-C3-Configuration.pdf)

### Inferences
- Rough scale: a pencil force of ~2–10 N at ~0.5 m lever is ~1–5 Nm on A2/A3. A small robot's max axis torque is tens of Nm, so even a tight 10–20 % tunnel is a few Nm. Pencil contact would be at or below the noise of the dynamic model. A simulator should not stop on pencil contact. It could model a stop only for gross collisions (e.g. pushing the pencil 10+ mm into a rigid board), and that is a heuristic.
- Practical drawing setups use a compliant (spring-loaded) pen holder and a small Z offset (kukodraw: 2.64 mm lift between pen up and down).

### Gaps
- No published max motor torque for KR 3 axes, so no numeric force threshold in N can be derived.
- KRC1 collision detection behaviour (whether the option existed) was not found.
