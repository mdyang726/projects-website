# TVC Model Rocket — Project Summary

*For website content. Summer 2026 independent engineering project.*

## Overview

I'm building a 4"-diameter model rocket with an active thrust-vector-control (TVC) system, with the eventual goal of a fully propulsive vertical landing — the same concept as a SpaceX Falcon 9 booster return, built at hobbyist scale from commercial-off-the-shelf and 3D-printed parts. A flight computer (Teensy 4.1) drives a two-axis gimbaled motor mount through a PID control loop, reading attitude from an onboard IMU, to keep the rocket vertical during flight.

The long-term architecture uses two solid motors stacked in series rather than a throttleable engine: an ascent motor for launch and a separate landing motor for the descent burn. Because the motors are fixed-thrust, control comes from *timing* — the flight computer calculates the ignition altitude for the landing burn dynamically at apogee — rather than from throttling. The first flight (targeted for August 2026) scopes this down to TVC-controlled ascent with parachute recovery only, to validate the control system before attempting the full landing sequence on a later flight.

The project is informed by an existence proof: a hobbyist (Mark, "Project Horizon") flew a rocket called Eagle using the same motor class and the same flight-computer architecture (Teensy 4.1) and achieved a successful propulsive landing after 30 iterative attempts.

## Simulation

Simulation work happened in layers, each one adding realism to the last, all in Python.

**Terminal-phase feasibility model.** Before any hardware design, I modeled the descent physics (gravity, drag, IMU double-integration for altitude) to answer one question: after the flight computer detects the ignition altitude, is there enough time before ground impact to fire the landing motor and have it produce meaningful thrust? This meant building a full latency budget — sensor loop cycle, MOSFET switching time, e-match ignition delay, motor spool-up time — and subtracting it from the physical time-to-impact. This determined that the ignition window is the single constraint the rest of the vehicle design has to satisfy, and it set starting values for apogee altitude and detection altitude before any CAD existed.

**Control law derivation.** A second script sweeps a range of possible apogee altitudes and, for each one, solves for the minimum ignition altitude that still leaves an adequate safety margin before impact. Fitting a line through the results produces a simple linear relationship, h_b = m·Ap + c, that becomes firmware: every flight, the rocket measures its own actual apogee during coast and computes its own ignition altitude from that formula, rather than flying to a fixed number. This makes the landing logic self-calibrating across flights with different peak altitudes.

**Full 2D pitch-plane flight simulation** (`tvc_sim.py`). A more complete physics model of the whole flight: the digitized F15 motor thrust curve, time-varying mass/CG/moment-of-inertia as propellant burns and motors eject, finless aerodynamics (drag, weathervane moment, pitch-rate damping), gimbaled-thrust dynamics, and the full ascent → coast → descent → landing-burn → touchdown sequence including motor ejection. The attitude controller in the sim is the same PID-with-thrust-feed-forward architecture that runs onboard.

**Stochastic / Monte Carlo layer** (`tvc_sim_stochastic.py`, `tvc_sim_sweeps.py`). Built on top of the flight sim to add real-world noise sources: IMU accelerometer and gyro noise and bias, an onboard strapdown attitude/altitude estimator running at realistic sensor rates (IMU at 1 kHz, control loop at 100 Hz, optional barometer fusion), servo dynamics (refresh rate, slew-rate limiting, first-order lag), crosswind and gusts, and motor-to-motor impulse scatter. Running hundreds of randomized flights produces a distribution of touchdown velocities rather than a single idealized answer, and sweeping individual parameters (e.g., IMU calibration quality vs. servo speed) identifies which subsystems actually matter — for example, the sweeps showed that touchdown velocity is far more sensitive to IMU accelerometer bias than to servo speed, which redirected engineering effort toward sensor fusion/calibration over faster servos.

## Design Process

The design evolved through several rounds of structured analysis and real trade-off decisions, not a single fixed plan:

- **Initial architecture (June 2026).** An early multi-perspective design review (five independent analyses plus cross-review, synthesized into a single brief) established the core architecture: shared gimbal and shared PID/TVC code for both ascent and landing burns, low center-of-gravity for passive self-righting during unpowered descent, a dynamic (not fixed) ignition-altitude control law, legs sized to survive off-nominal landing velocities rather than only the nominal case, and a five-state flight software model (boost → coast → descent → ignition → landing).

- **Airframe and stability.** Chose a 4" commercial airframe tube and nose cone specifically to avoid needing CFD — known, published aerodynamics. Because the rocket is finless and relies on active TVC rather than passive fins, the traditional "positive stability margin" goal doesn't apply the way it would to a normal model rocket; the real requirement is a torque budget where gimbal-deflected thrust can out-torque the aerodynamic destabilizing moment throughout the burn. CG/CP modeling is being done in OpenRocket, with the first pass run in July using placeholder masses pending final hardware weigh-in.

- **Launch method.** Originally planned to launch off the landing legs; ruled out once the low-CG design was confirmed to be passively unstable during ascent (the same property that helps it self-right on descent works against it on the way up). Switched to a lug/rail launch system with a custom-built launch pad, sized after determining the rocket's mass and hardware likely put it past the point where a thin launch rod would be adequate ("rod whip" risk).

- **Recovery mechanism.** Iterated toward a servo-actuated nose-cone separation system: a servo retracts a hook/pin, a compression spring provides positive separation, and the nose cone stays tethered by a Kevlar shock cord running through a ball-bearing swivel (to prevent parachute twist) with the chute packed in-line. Considered 3D-printed bulkheads (PETG, through-bolted with washers rather than threaded directly into the plastic) as an alternative to traditional plywood mounting discs.

- **Motor staging.** Adopted the same friction-fit, e-match-sandwiched motor coupling used in the reference build: the ascent motor's own thrust holds it pressed against the landing motor during boost, and the landing motor's thrust ejects the spent ascent casing when it fires — no separate mechanical release needed.

- **Scope discipline.** Deliberately descoped the first flight to TVC-during-ascent-plus-parachute-recovery only, deferring the two-motor propulsive landing stack and legs to a later flight once ascent control is proven on real flight data. This followed directly from a testing philosophy of treating every flight as a data-collection event rather than a pass/fail attempt, and maximizing what can be validated on the bench (static fires, hardware-in-the-loop state-machine testing, drop tests for leg deployment) before committing to a full-system flight.

- **Risk tracking.** Actively tracking open engineering risks such as motor thrust-to-weight margin (the vehicle's mass has to stay within a fairly narrow band for the selected Estes F15-0 motor to provide adequate liftoff acceleration), a control-loop latency bug inherited as a known failure mode from the reference build, and whether the finished vehicle's weight or motor impulse crosses from model-rocket into high-power-rocketry classification, which would trigger certification requirements.

## CAD Work

Mechanical design is done in Siemens NX, with STEP/STL export for 3D printing and for cross-checking in other tools. Parts modeled and iterated so far include:

- **TVC gimbal assembly** — inner ring, outer ring, and arm components (multiple iterations, e.g. `inner_ring_v1` → `v5`) forming the two-axis mount that lets the motor pivot under servo control.
- **Motor container** — the printed housing that holds the motor on the gimbal; currently being redesigned after the first version came out oversized.
- **Nose cone** — a parametric tangent-ogive design (hollow, printable, wall-thickness-controlled) generated from a scripted NX Open (Python/NXOpen) journal rather than built by hand, so the geometry can be regenerated instantly if the body-tube diameter or ogive length changes. Later iterations add mounting features to support the parachute-release hardware (servo, hook, spring, bulkhead).
- **Rocket body / airframe model** — full-vehicle model used to check part fit and packaging.
- **Avionics bay** — went through a full redesign: an initial two-sled, threaded-rod-rail concept was rejected in favor of a single rigid printed tray (~88 × 130 mm) that mounts directly to the airframe wall via printed bosses, laid out to fit the Teensy, IMU, barometer, both voltage regulators, the power-distribution perfboard, and the battery, with the IMU intentionally co-mounted rigidly rather than vibration-isolated.
- **Servo and servo-arm models**, including a dedicated MG90 arm part, used to verify gimbal actuation geometry and range of motion.
- **Supporting reference geometry** — cross-section diagrams (e.g. nose cone ogive arc) used to verify profiles before committing to a print.

CAD outputs feed directly back into simulation: measured and estimated part masses/dimensions are tracked in a parts-reference table used to build the OpenRocket aerodynamic/stability model, closing the loop between mechanical design and the flight simulation.
