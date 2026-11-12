# KUKA KRL motion and syntax semantics on KR C controllers (for a student pencil-drawing simulator)

Source key (cited inline):
- **[E52]** KUKA *Expert Programming, KSS Release 5.2* (ProgExperteBHR5.2 09.03.00 en, KR C1/KR C2 era). Third-party copy: https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf. I read the full text after extracting it with pdftotext.
- **[R41]** KUKA *KRL Reference Guide, Release 4.1* (RefGuideR4.1 09.01.00 en, KR C1/C2/C3). Third-party copy: https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf. Full text extracted.
- **[KSS83-snippet]** KUKA *KSS 8.3 Operating and Programming Instructions*, manualzz copy: https://manualzz.com/doc/23920333/kuka-kss-8.3-robotics-operating-and-programming-instructions. The page returned HTTP 403, so I only have the search-engine snippet.
- **[QG8-snippet]** *Quickguide KRL-Syntax KUKA Roboter KSS Release 8.x*: http://www.wtech.com.tw/public/download/manual/kuka/krc4/KUKA%20KRL-Syntax%208.x.pdf. The connection was refused, so I only have the search snippet.
- **[WB-snippet]** KUKA training workbooks, from search snippets only: http://www.wtech.com.tw/public/download/manual/kuka/krc2ed05/KUKA-Workbook.pdf and https://pdfcoffee.com/workbook-p1kss8-programming-1-v1-en-pdf-free.html

Version caveat: the two primary texts I read in full are KSS 4.1/5.2 manuals (KRC1/KRC2). The KSS 8.x (KRC4) statements come only from search snippets. The core semantics below (partial aggregates, first motion, CIRC, approximation, IN/OUT) are stable across versions as far as I could verify. KSS 8.x also adds the spline motions (SPTP/SLIN/SCIRC), which I did not research.

## 1. Partial (incomplete) aggregates in PTP / LIN / CIRC: where do the missing components come from?

### Takeaway
Missing components of a motion target are "taken unchanged from the current position". The Expert manual says that for path motions this "current position" is the **last point programmed in the program**, not the measured $POS_ACT. For axis targets, the axes you omit simply do not move. The first motion should not be partial: it must be a complete PTP, with S and T for Cartesian targets.

### Cited Findings
- PTP target (R41 §2.2.32.2): "If the target position contains undefined structure components, these values are taken unchanged from the current position." The reference system for a Cartesian PTP target "is defined by the system variable $BASE." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- LIN target (R41 §2.2.29.2) uses the same wording: "If the target position contains undefined structure components, these values are taken unchanged from the current position." S and T in a POS/E6POS target "are always disregarded" for LIN. — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- CIRC (R41 §2.2.6.2): "If structure components are missing in the auxiliary point, these values are taken unchanged from the current position." The target point is handled the same way, and both points are referenced to $BASE. — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- What "current position" means for LIN (E52 §3.3.3): "the KR C... calculates a straight line from the current position (the last point programmed in the program) to the position specified in the motion command." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Axis targets (E52 §3.2, PTP): "If individual components are omitted when the axis coordinates are entered, the robot only moves the axes that have been specified; the others do not change position. With PTP {A3 45} for example, only axis 3 is moved to 45°." Values are absolute; PTP_REL is the relative form. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Cartesian targets (E52 §3.2): "It is also possible to omit individual components of the geometrical specification when entering the end point using Cartesian coordinates. The instruction PTP {Z 1300, B 180} moves the TCP in the direction of the Z axis to the absolute position 1300 mm and 'tilts' the TCP by 180°." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KSS 8.x quick guide, per the search snippet: omitted axis components of a PTP aggregate take the values of the preceding position, and `PTP {A1 -45}` is shown as valid. — [QG8-snippet](http://www.wtech.com.tw/public/download/manual/kuka/krc4/KUKA%20KRL-Syntax%208.x.pdf)
- Partial assignment to a variable is a different case (E52 §2.2.x, aggregates). "In the case of POS, E6POS, AXIS, E6AXIS and FRAME structures, missing components are not altered. In all other aggregates, missing components are set to invalid." Example: `POSITION={B 100.0,X 29.5,T 6}`. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- S and T when omitted (R41 §2.2.32.3): "If the specification for Turn T is omitted from a PTP motion, the robot will always move along the shortest path. If the specification for Status S is omitted, the status from the previous point is retained." E52 adds: "The robot retains the old value for S and selects the T value which gives the shortest possible axis trajectory." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf); [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- First motion (E52 §3.2): "the first motion instruction of a program (home run) must always be a complete PTP instruction with 'Status' and 'Turn' specified (or a complete PTP instruction with axis coordinates)." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KSS 8.3, per the search snippet: if the first motion is not the default HOME, the program must start with a complete PTP of type POS/E6POS or AXIS/E6AXIS. "'Complete' means that all components of the end point must be specified." — [KSS83-snippet](https://manualzz.com/doc/23920333/kuka-kss-8.3-robotics-operating-and-programming-instructions)
- Prerequisites (R41 §2.2.32.3): if $VEL_AXIS/$ACC_AXIS "have not been programmed before the first motion instruction, an error message will occur when the program is executed! This also applies to the system variables $TOOL and $BASE if the target position is specified in Cartesian coordinates." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)

### Inferences
- Simulator rule: keep a "last programmed target" pose P_prev, in Cartesian (in the current $BASE/$TOOL) and in axis form. For each partial aggregate, fill the missing fields from P_prev. Do not fill them from the interpolated actual position. With approximate positioning (C_DIS etc.) the actual TCP never reaches P_prev, but the programmed corner point is still the reference.
- `PTP {z -2}` therefore means: same X, Y, A, B, C (and S) as the last programmed point, Z = -2 in the current $BASE. For pencil drawing this is the usual "pen down" idiom.
- A partial aggregate as the very first motion should be flagged as an error, or at least a warning. Its missing components would come from an undefined "previous point", which on a real controller means the position the robot happens to be in (the BCO situation).

### Gaps
- No source states explicitly whether "current position" is internally $POS_RET, $POS_BACK or $POS_FOR. All three are listed in E52 Table 6 as variables that trigger an advance-run stop. The manuals only say "current position" and "last point programmed".
- I found no KSS 8.x full-text confirmation of the "previous position" wording, only the QG8 search snippet.

## 2. Changing $BASE between motions: interpretation of later partial targets, and does the robot move?

### Takeaway
Assigning $BASE (or $TOOL) is an ordinary variable assignment. It does not move the robot. Cartesian targets of motions programmed after it are interpreted in the new base. KUKA warns that changing $BASE in mid-program breaks block selection, and that a $TOOL assignment may not sit between two approximated points.

### Cited Findings
- The target reference system is always the current $BASE: "The reference system for a Cartesian target position is defined by the system variable $BASE" (PTP, LIN and CIRC syntax tables). — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- E52 §3.2 recommends the geometric (colon) operator over reassigning $BASE: "The offset occurs in the motion instruction, whereas a $BASE setting must be made before the motion instruction." Example: `PTP BASE1:{POS: X 540,...}`. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- E52 §3.2 on repeated `$BASE = ...` between `PTP POS_1` and `PTP POS_2`: this "would lead to an incorrect end point after cancellation of the POS_2 motion instruction and reselection of the POS_1 instruction, because the POS_1 motion instruction would now also relate to the new base. This also occurs ... in the event of the first motion instruction being stopped if a corresponding computer advance run is set." It concludes: "where possible, $BASE and $TOOL should only be set once, e.g. in the initialization section." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- R41 (approximate positioning): "A $TOOL statement may not appear between two approximate positioning points." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- "Changing the variable $BASE in the interrupt routine only has an effect there." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- For CIRC, "$BASE and $TOOL must be fully assigned before executing a circular motion." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Advance run: the interpreter executes ahead of the robot ($ADVANCE, set to 3 by BAS(#INITMOV,0) and 0 in the system default), so statements are evaluated before earlier motions finish. Only the instructions in E52 Table 6 stop the advance run. That table lists $POS_ACT, $POS_RET, $POS_BACK, $POS_FOR, $ACT_BASE, $ACT_TOOL, $IN/$OUT, WAIT, HALT and others. $BASE and $TOOL themselves are not listed. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- Simulator rule: at the `$BASE = X` statement, change only the interpreter state. The pen does not move. The next Cartesian target is mapped to world as `X : target`.
- For a partial target after a base change, R41's rule ("taken unchanged from the current position") together with "reference system = $BASE" implies this: the previous programmed point is expressed in the **new** base, missing components are filled from those coordinates, and the given ones are overwritten. The result is a world point that keeps the previous point's coordinates along the omitted axes of the new base. That is generally not "the same numbers as before in the old base". I could not find this spelled out in a manual. Treat it as the most plausible reading and offer it as a simulator option, or flag partial targets right after a base change as a warning.
- For students: tell them to set $BASE/$TOOL once at the start, or to use `BASE_X:{...}` offsets, which is exactly KUKA's advice.

### Gaps
- No primary text says outright whether the "previous point" used for filling is converted into the new base or kept numerically. This needs a test on a real controller or a forum confirmation.

## 3. $TOOL frame convention and TCP orientation A, B, C

### Takeaway
A, B, C are Z-Y'-X'' intrinsic Euler angles: A about Z, then B about the new Y, then C about the new X. KUKA's convention puts the tool's working direction on the **X axis** of the tool frame, pointing out of the tool. Some KSS versions and configurations allow Z instead.

### Cited Findings
- E52 §3.1: "The angles designated A, B and C in the KR C... describe rotations about the coordinate axes Z, Y and X. The order of the rotations must be retained. 1. Rotation through angle A about the Z axis 2. Rotation through angle B about the new Y axis 3. Rotation through angle C about the new X axis". It says this matches roll-pitch-yaw, with C = roll, B = pitch, A = yaw. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- E52 §3.1, tool coordinate system: "has its origin at the tip of the tool (Tool Center Point: TCP). The orientation can be selected in such a way that its X axis is identical to the tool direction and points out of the tool." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- E52 (technology chapter): "The tool direction is usually the X axis of the calibrated tool. Depending on the configuration, the Z axis of the tool may also be used as the tool direction." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- $ORI_TYPE=#VAR "is achieved by rotating and pivoting the tool direction". $APO.CORI refers to the "dominant orientation angle (swiveling or rotation of the longitudinal tool axis)". — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KUKA's ABC calibration methods (2-point / World 5D) jog "along the tool working direction" (KUKA College slides, from a search snippet only). — [USP copy of KUKA slides](https://edisciplinas.usp.br/mod/resource/view.php?id=2397395)

### Inferences
- Pencil simulator: model the pencil along the +X axis of $TOOL. With A = 0, B = 90, C = 0 in a base whose Z points up, the tool X axis points down along -Z. This is the classic "tool pointing down" pose: Rz(0)·Ry(90°) maps X to -Z. Pose matrix R = Rz(A)·Ry(B)·Rx(C).
- A TCP pose {X,Y,Z,A,B,C} describes the $TOOL frame relative to $BASE: world_T_tcp = $BASE · pose, and flange = world_T_tcp · inv($TOOL).

### Gaps
- Third-party blogs disagree on whether KSS 8.x lets users pick Z as the tool direction (a machine-data option). I found no official KSS 8 text, so the version-specific behavior is unverified.

## 4. BCO run (block coincidence)

### Takeaway
BCO = "block coincidence". Before a program runs, the robot must be moved onto the programmed path. That move is made at reduced speed with Start held down, and for a PTP block it is a PTP move. The first motion must be a complete PTP (HOME by convention), because LIN/CIRC ignore S and T and cannot define the arm configuration.

### Cited Findings
- E52 §3.2: "Before a program can be executed, block coincidence (BCO) must be established, i.e. correspondence between the current robot position and the programmed position. Since the BCO run does not represent a programmed, tested motion, it must be executed with the Start key held down ... and at automatically reduced velocity. When the robot has reached the programmed path, the motion is stopped and the program can be continued by pressing the Start key again." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "No BCO run is carried out in 'Automatic External' mode!" — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "It is thus advisable to program a 'Home' run as the first motion instruction" (sample programs use `PTP HOME ;BCO run`). — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- LIN (E52 §3.3.3): "In the case of linear motions, the status of the end point is the same as that of the start point ... A PTP motion (e.g. HOME run) with complete position specification (coordinate specification including status) must therefore be programmed before the first LIN instruction." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- R41 (PTP, LIN, CIRC): "The first motion instruction of a program must therefore always be a PTP instruction specifying S and T." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Training workbook, per the search snippet: "A BCO run is executed as a PTP motion ... If the selected motion block contains LIN or CIRC, the BCO run is executed as a LIN motion". BCO runs are needed after program selection or reset, and after a motion command is modified. — [WB-snippet](http://www.wtech.com.tw/public/download/manual/kuka/krc2ed05/KUKA-Workbook.pdf)
- A BCO run also triggers an advance-run stop. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- KSS 8.3: the first motion must be a complete PTP, either POS/E6POS or AXIS/E6AXIS (see §1). — [KSS83-snippet](https://manualzz.com/doc/23920333/kuka-kss-8.3-robotics-operating-and-programming-instructions)

### Inferences
- Simulator: require the first motion to be a PTP with a complete AXIS/E6AXIS target, or a complete POS/E6POS with S and T. If the first motion is LIN/CIRC or a partial PTP, raise an error modeled on the controller's BCO/"first motion must be complete PTP" check.

### Gaps
- I did not find the exact KSS message text or number shown when the first motion is LIN/CIRC or incomplete. Industrialmonitordirect.com mentions a "BCO programmed path reached" message, but that source is unreliable. Don't invent a number. Use a generic message such as "First motion must be a complete PTP (BCO)".

## 5. CIRC: auxiliary point, orientation, CA, degenerate cases

### Takeaway
The circle is defined by start (= previous programmed point), aux and end. Only the aux point's X, Y, Z are used. CA sets the actual arc angle along that circle, and its sign sets the direction. Orientation follows $ORI_TYPE (#CONSTANT/#VAR/#JOINT) and $CIRC_TYPE (#BASE space-related / #PATH path-related). The three points must be distinct and non-collinear.

### Cited Findings
- E52 §3.3.4: "To define a circle or arc in space unambiguously, three points are needed which are different from one another and do not lie on a straight line. The start point of a circular motion is again formed, as with PTP or LIN, by the current position." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "When the controller calculates the motion path, only the translational components (X, Y, Z) of the auxiliary point are evaluated." R41 adds: "The orientation angles and the angle status specifications S and T for an auxiliary point are always disregarded." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf); [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Syntax (R41): `CIRC Auxiliary_Point, Target_Position <,CA Circular_Angle> <Approximate_Positioning>`. Aux and target are POS/E6POS/FRAME only (Cartesian), in $BASE. The approximation keyword is C_DIS (default), C_ORI or C_VEL. — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- CA (E52): "The geometry of the arc is defined, as always, by means of start, auxiliary and end points. The actual end position on the arc, however, where the motion ends, is determined by the programmed circular angle ... The programmed end orientation is then reached at the actual end point." CA > 0: start → aux → end direction. CA < 0: start → end → aux, the opposite direction. "The value for the circular angle is unlimited. In particular, full circles (> 360°) can be programmed." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- R41: "When specifying a circular angle, the programmed target position is not generally the real target position." The unit is degrees. — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- CIRC_REL: "Relative specifications for auxiliary and end positions ... relate to the respective start position. ... axis-specific position specifications are not permissible." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- $ORI_TYPE (E52 §3.3.1): "#CONSTANT: The orientation remains constant during the CP motion. The programmed orientation is disregarded for the end point and that of the start point is used." "#VAR (default): ... the orientation of the tool changes continuously from the start position to the end position." "#JOINT" is recommended near singularities. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- $CIRC_TYPE (E52 §3.3.4): "#BASE: Space-related orientation control during the circular motion. This value is also set during initialization, by BAS(#INITMOV,0) ... the orientation is controlled relative to the current base system ($BASE)." "#PATH: ... the longitudinal axis of the tool is moved relative to the plane and tangent of the circle", using the tangent/normal/binormal moving frame. "$CIRC_TYPE is meaningless ... with $ORI_TYPE = #JOINT". Table 5 shows the system default of $CIRC_TYPE as #PATH and the BAS(#INITMOV,0) value as #BASE. The table is garbled in the extracted text, but the #BASE/BAS link is stated explicitly in the body text. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- Simulator geometry: compute the circle through P0 (previous programmed point), Pa (aux XYZ) and Pe (end XYZ). The swept angle without CA runs from P0 through Pa to Pe. With CA, sweep |CA| degrees from P0, in the P0→Pa direction if CA > 0 and the opposite way if CA < 0, then stop. Interpolate orientation from start to the programmed end orientation, which is reached at the actual end point.
- If the aux point is not between start and end, nothing special happens geometrically. The arc is simply the one passing start → aux → end, which can be the major arc (> 180°). That behavior follows from the three-point definition.
- If the points are collinear or coincident (|cross| ≈ 0), raise an error. The manual requires three distinct, non-collinear points.
- For a pencil (rotationally symmetric tool), use $ORI_TYPE=#CONSTANT and $CIRC_TYPE=#BASE so the pencil keeps a constant tilt. #PATH would rotate the pencil about the circle normal.

### Gaps
- I found no exact controller error text or number for collinear or too-close points. A third-party blog mentions a "mid point and end point same" error, but I could not verify it. I also found no documented minimum distance or angle tolerance.

## 6. Approximate positioning: C_PTP, C_DIS, C_VEL, C_ORI and $APO

### Takeaway
Approximation replaces the sharp corner at an approximated point with a controller-generated blend. For LIN-LIN it is a parabola, tangent to both segments. The blend starts no earlier than $APO.CDIS mm before the point (C_DIS), $APO.CORI degrees (C_ORI) or $APO.CVEL % of $VEL (C_VEL). It is limited to half of each block. For drawing, this means **corners get rounded and the pen never touches the programmed corner point.** It also requires an advance run, and any advance-run-stopping statement right after the motion cancels the blend.

### Cited Findings
- E52 §3.5: "points for which exact positioning is not necessary can be approximated. The robot takes a shortcut ... The approximate positioning contour is automatically generated by the controller. The programmer can only influence the beginning and the end of approximate positioning." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "To make approximate positioning possible, a computer advance run ($ADVANCE) of at least 1 must be set. If the computer advance run is too small, the message 'Approximation not possible' appears and the robot is positioned exactly at each point." "If, after an approximate positioning instruction, you program an instruction which automatically stops the advance run, approximate positioning is not possible." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- PTP-PTP (E52 §3.5.1): blending begins when the leading axis is within an angle of the point, set per axis by $APO_DIS_PTP[i] (e.g. 90°) times $APO.CPTP %. "The greater the value of $APO.CPTP, the more the path is rounded." "Approximate positioning can never take place over the middle of the block!" The syntax is `PTP POINT4 C_PTP`. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- LIN-LIN (E52 §3.5.2): "The controller calculates a parabolic path for the approximate positioning contour." $APO.CDIS (REAL, mm) goes with C_DIS: "the controller leaves the individual block contour, at the earliest, when the distance from the end point falls below the value in $APO.CDIS." $APO.CORI (REAL, °) goes with C_ORI. $APO.CVEL (INT, % of $VEL, during the deceleration phase) goes with C_VEL. "In certain circumstances, the system may shorten approximate positioning (middle of the block, symmetry criterion), but will never lengthen it." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "If $ACC and $VEL are identical for both individual blocks and the lengths of the blocks are sufficient, the approximate positioning parabola is symmetrical with the bisector of the angle ... With short blocks, the start of approximate positioning is limited to half the length of the block." "The transitions ... are continuous and tangential." The contour "is independent of override alterations". — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- R41: "A maximum of half the programmed distance may be approximated." Also: "The smaller $APO.CPTP is, the smaller the approximate positioning range will be." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- CIRC-CIRC and CIRC-LIN use the same $APO criteria and keywords. The contour is generally not symmetrical. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- PTP → CP blending needs both keywords on the PTP, e.g. `PTP P C_PTP C_DIS`: "The option Approximate_Positioning must also be specified for approximate positioning in a subsequent CP block." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- CP → PTP blending is only possible if no axis rotates more than 180° in the CP block and S stays unchanged. Otherwise the controller shows "CP/PTP approximation not possible" and positions exactly. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Statements that stop the advance run, and therefore block approximation (E52 Table 6): WAIT, HALT, $IN/$OUT access, $POS_ACT/$POS_RET/$POS_BACK/$POS_FOR, $AXIS_ACT etc., and PULSE. `WAIT SEC 0` is the documented trick to force an advance-run stop. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- Why it matters for drawing: with `LIN P C_DIS` and $APO.CDIS = 10, a square drawn with a pencil comes out with rounded corners of roughly 10 mm, capped at half the shortest edge. A "pen up" move `LIN {Z 10} C_DIS` blended into the next XY move makes the pencil lift diagonally, which drags a tail on the paper. Students should use exact positioning (no C_xxx) on pen-down and pen-up moves and on sharp corners.
- Simulator minimum model for LIN-LIN: cut each segment at d = min($APO.CDIS, L_in/2, L_out/2) from the corner and join the cut points with a quadratic Bézier whose control point is the corner. This is tangent at both ends and symmetric when the cut lengths are equal, which matches "parabolic" and "symmetrical with the bisector". C_VEL and C_ORI depend on dynamics, so approximate them or treat them like C_DIS.
- If the next statement stops the advance run (WAIT, $OUT, $POS_ACT read, etc.), the simulator should ignore C_xxx and stop exactly, with a warning "Approximation not possible".

### Gaps
- The exact KSS 8 blend shape for LIN/CIRC is not reproducible from the manual. The manual says only "parabolic", and it is dynamics-dependent. KSS 8 spline approximation (SLIN etc.) differs and was not researched.

## 7. KRL syntax rules relevant to students

### Takeaway
A program file is `DEF name(params)` … `END`, with all declarations before the first executable statement. `;` starts a comment. `==` compares and `=` assigns. INT/INT division truncates. Parameters are by reference (:OUT) by default and by value with :IN. Variables in the .src are local to that program; .dat variables are visible to its local subprograms; $CONFIG.DAT and GLOBAL make things global.

### Cited Findings
- File structure (E52 §1): "The inner structure of a KRL file comprises the declaration section, the instruction (or statement) section and up to 255 local subprograms and functions." "Every file begins with the declaration 'DEF' and ends with 'END'". Names are up to 24 characters and must not be keywords. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- DECL placement: "Declarations are already evaluated before program execution, i.e. during compilation. No instructions may therefore be located in the declaration section. The first instruction is the beginning of the instruction section." Also: "A value assignment to a variable is an instruction and must therefore never be located in the declaration section." The declaration of variables "must always take place before" the INI fold. Missing declarations produce errors such as "2249" in the E52 example. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- "The keyword DECL can be omitted when declaring data types that are predefined" (e.g. `INT I`). — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Data lists: "Only value assignments with '=' are allowed in data lists." — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Comments: "A comment is text that is ignored by the compiler. It is separated from the program code in a program line by means of the ';' character." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Operators: `==` is "Equal to". The relational operators `==, <, >, <=, >=, <>` have the lowest priority (7). Assignment uses `=`. — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf); [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Integer division: "The result of an arithmetic operation is only INT if both operands are of the data type INTEGER. If the result of an integer division is not an integer, it is cut off at the decimal point. If at least one of the two operands is REAL, the result too will be of the data type REAL." Example: "7/4 = 1". — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Scope (R41 §2.1.x): data objects are "only recognized locally, with one exception: variables declared in the $CONFIG.DAT file are also recognized globally". The GLOBAL keyword makes a declaration global. "A local variable is valid in the program code containing the declaration of the variable and situated between the keywords DEF and END[DEF]." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Scope (E52 §6.1): "All variables declared in the data list of the main program are recognized in local subprograms and functions. Variables which have been declared in the main program (SRC file) are so-called 'runtime variables' and may only be used in the main program. Attempting to use these variables in the subprogram causes a corresponding error message." Variables in the main program are also not visible in global subprograms. A local subprogram can be made callable from outside with GLOBAL. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- Parameters (E52 §6.2): "'Call by value' is entered ... by the keyword IN after every variable in the parameter list. 'Call by reference' is obtained by entering the word OUT. OUT is also the default setting. Example: DEF CALCULATE(X:OUT,Y:IN,Z:IN,B)". — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)
- R41 on IN: "A value cannot be returned to the calling module in the case of an IN parameter (call by value) ... the system automatically converts the type ... Arrays cannot be transferred as an input parameter (IN)." On OUT: "A variable name is transferred here (call by reference). The variable must have a value at the time of the subprogram call ... the data types of the current and formal parameters must be identical." — [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Loops (E52 §4): `FOR I=1 TO 16 ... ENDFOR` (STEP optional), `WHILE cond ... ENDWHILE`, `REPEAT ... UNTIL cond` (always runs at least once), and `LOOP ... ENDLOOP` (endless). "Any loop can be terminated prematurely by using the EXIT statement ... resumed after the ENDLOOP statement." `IF ... THEN ... ELSE ... ENDIF` and `SWITCH ... CASE ... DEFAULT ... ENDSWITCH` also exist. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf); [R41](https://robot.zaab.org/wp-content/uploads/2014/04/KRL-Reference-Guide-v4_1.pdf)
- Number literals: binary `'B1011010'` and hex `'H5A'`. — [E52](https://icdn.tradew.com/file/201606/1569362/pdf/7033432.pdf)

### Inferences
- Case-insensitivity: both manuals print keywords in upper case and the sample code mixes case (`DEF Globals()`). KRL identifiers and keywords are case-insensitive in practice, but neither manual I read states it in one sentence. Implement case-insensitive matching and cite it as common practice, not as a manual quote.
- `IF A = 5 THEN` is a syntax error in KRL, because `=` is only assignment and a condition needs `==`. That follows from the operator definitions, and the simulator should report it clearly.
- Simulator: reject statements before or between declarations, apart from the allowed ones. Truncate INT/INT division (`7/4` gives 1) and consider a student warning when that INT result is assigned to a REAL (see Gaps).

### Gaps
- No explicit manual sentence on case-insensitivity was found.
- `REAL R; R = 7/4`: the manual says the result is INT (1) and then assigned. Whether R becomes 1.0 follows from the rule, but there is no explicit example.
- FOR loop details (STEP sign, whether the loop variable must be INT, its value after the loop) were not extracted verbatim.
- KSS 8.x differences in scope rules (e.g. GLOBAL in .dat requiring `DEFDAT name PUBLIC`) were not verified from a primary source.
