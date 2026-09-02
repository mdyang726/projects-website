# Figure captions and alt text — TVC rocket static fire telemetry

All figures are generated from the on-board flight logs (`Static_Test_1.CSV`,
`Static_Test_2.CSV`). Two provenance caveats apply to every figure and should
stay in the captions:

- **Attitude is the complementary-filter output**, not raw accelerometer angle.
  The filter's inputs were not logged in either test.
- **Servo traces are commanded position, not measured.** There is no position
  feedback in the system; the log cannot confirm the servo reached the angle.

The shaded band marks the firmware's burn-timer window, derived from the
certified F15-0 thrust curve replayed by elapsed time. There is no load cell
in this setup — no thrust value here is measured.

---

## 01 — Test 1, full session

**Caption.** Attitude estimate across the entire Test 1 log. The estimate holds
within 0.12 deg standard deviation for 95 seconds, then diverges at t = -2.55 s
-- 2.55 seconds before `motorIgnited` latches, and 1.55 seconds before the motor
actually lit. Frame-by-frame tracking of the test video puts the airframe
motionless to within about half a degree across that entire window, while the
estimate read 79 deg in pitch and swept roll across nearly the full +/-180 deg
`atan2` output range. Attitude shown is the complementary-filter output, not raw
accelerometer angle.

**Why this is an axis-convention bug and not a calibration offset.** A
calibration offset is a constant: present from power-on, identical at every
orientation, corrected with a single subtraction. This error was absent for the
first 95 seconds and then appeared, and it produced a roll range of -176.67 to
+176.61 deg on a vehicle that was bolted down and physically still. That is the
signature of `atan2` receiving two channels that are both near zero because
gravity is resting on the third -- the output is noise-dominated and swings
across its full range. An offset cannot produce that.

**Alt text.** Line chart of pitch and roll over 95 seconds. Both traces are flat
near zero until 2.55 seconds before ignition is detected, when they abruptly
diverge, pitch climbing to 80 degrees and roll sweeping across the full plus and
minus 180 degree range.

---

## 02 — Test 1, attitude and commanded servo angle

**Caption.** Burn-window detail. Both servo channels sit at a travel limit for
100% of the burn — the inner axis used 2 of its 9 addressable positions, the
outer 2 of 9, with 4 and 7 command changes across 274 samples. Grey lines mark
the 9 integer positions `PWMServo.write()` can address. Commanded position; no
position feedback exists.

**Alt text.** Two stacked charts sharing a time axis. Upper shows pitch and roll
diverging through the burn. Lower shows both servo commands as flat steps pinned
at their extreme values, almost never changing.

---

## 03 — Test 2, baseline and divergence

**Caption.** Test 2 held a clean pre-ignition baseline — pitch −1.75° (σ 0.14),
roll −8.92° (σ 0.21) over the three seconds before ignition, body rates under
0.4°/s. At ignition the vehicle diverges immediately and monotonically, reaching
−307° in pitch by t = 2.52 s. Pitch here is unbounded gyro integration, not a
wrapped orientation reading.

**Alt text.** Line chart showing pitch and roll flat and quiet for 50 seconds
before ignition, then pitch dropping steeply and continuously to negative 307
degrees within two and a half seconds.

---

## 04 — Test 2, attitude and commanded servo angle

**Caption.** Burn-window detail for Test 2. The inner channel spent 98.9% and
the outer 96.4% of the burn at a travel limit, using 5 and 7 of 9 available
positions. Compare against figure 02: the saturation signature is nearly
identical to Test 1 despite an unrelated root cause.

**Alt text.** Two stacked charts. Upper shows pitch falling steeply through the
burn. Lower shows servo commands stepping between discrete integer levels but
spending nearly all the time at the extremes.

---

## 05 — Test 2, saturation versus divergence

**Caption.** The clearest evidence for the inverted control sign. Over the first
0.9 s after ignition the inner servo is held at its −4° rail for 70 of 71
samples — the maximum correction available to the controller — while pitch runs
monotonically from −3.25° to −195.4°, passing −45° at 0.35 s, −90° at 0.51 s and
−180° at 0.80 s. A correctly-signed loop saturating at a rail drives error toward
zero. This one grew error the entire time.

**Note on method.** A command-versus-response correlation was considered and
rejected: in a closed loop the controller responds to error and the plant
responds to command, so anticorrelation appears whether or not the sign is
inverted. The saturation-with-growing-error argument does not have that problem.

**Alt text.** Two stacked charts. Upper shows the inner servo command flat at its
minimum for nearly the whole window. Lower shows pitch falling smoothly and
continuously from near zero to negative 195 degrees over the same window.

---

## 06 — Servo quantizer usage

**Caption.** Distribution of commanded servo positions during each burn. The
2.3× servo-to-gimbal linkage ratio combined with `write()`'s integer-degree
argument collapses the ±10° gimbal range onto 9 addressable positions roughly
2.6° apart — coarser than the attitude error the controller is trying to null.
In practice neither test used more than a fraction of them; both lived at the
rails.

**Alt text.** Two horizontal bar charts side by side, one per test, showing the
percentage of burn samples spent at each commanded servo position. Nearly all
bars cluster at the extreme positions with almost nothing in between.

---

## 08 — Dead-reckoned vertical velocity

**Caption.** Accelerometer integration produced roughly −53 m/s of apparent
vertical velocity by burnout in **both** tests, on a vehicle bolted to a stand
that never physically moved. The velocity-based deployment trigger fired 0.03 s
after the arming delay expired in both runs — in flight this logic would have
deployed the parachute at burnout, at maximum velocity. `DEPLOYMENT_INHIBITED`
correctly suppressed actuation (`deploymentCommanded` never leaves 0 in either
log), but the finding rules out accelerometer integration as a deployment trigger.
A barometric altimeter is required.

**Alt text.** Line chart of dead-reckoned vertical velocity for both tests,
each falling steadily to about negative 53 metres per second by burnout, with
markers showing where the deployment trigger fired.
