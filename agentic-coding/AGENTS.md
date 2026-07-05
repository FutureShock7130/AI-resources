# AGENTS.md — Team 7130 Robot Code

If you are a human: the rules below describe how we want the AI agent to work with
us. If you are the AI agent: **these are instructions, follow them.**

---

## 0. The most important rule: clarify before you code

This team runs a **strict clarify-first policy**. When a request is vague,
underspecified, or could reasonably be built more than one way, **stop and ask
questions before writing any code.**

**Always ask** before proceeding when a request touches any of these and the value
is not stated or available:

- Motor/sensor **CAN IDs**, **CANivore bus** name, or which physical device is meant
- **Gear ratios**, sensor-to-mechanism ratios, or unit conventions (rotations vs.
  radians vs. meters)
- **Setpoints / targets**: shooter RPS, arm angles, heights, speeds, current limits
- **Limits** Soft limits, safety limits, or anything that could let a mechanism hit
  itself.
- **PID / feedforward gains** (kP, kI, kD, kS, kV, kA, kG) if writing motor control
  features — never invent these
- What "done" looks like — the desired robot behavior in plain words

Good clarifying questions are specific and offer options. Example:

> You asked me to "make the intake faster." A few things I need:
> 1. Do you mean the **roller speed** (`kIntakeSpeed`, currently 0.48) or the
>    **deploy/pivot speed** (Motion Magic accel/jerk)?
> 2. What target value, or should I tune it live on a dashboard?
> 3. Should this also change the auto routines, or teleop only?

Only skip clarification when the task is genuinely unambiguous (e.g., "fix this
compile error", "rename this variable", "add a Javadoc comment"). When in doubt, ask.
Stating an assumption is **not** a substitute for asking about hardware values —
for CAN IDs, gains, and setpoints you must ask, not assume.

---

## 1. FRC / WPILib concepts you must understand

If you are new to FRC, learn these before editing. The whole codebase assumes them.

### Command-based programming
WPILib organizes robot code into **Subsystems** and **Commands**.

- A **Subsystem** (`extends SubsystemBase`) owns hardware — motors, sensors — and
  exposes methods to control it. Each subsystem is a *resource*: only one command
  may use it at a time.
- A **Command** describes an *action over time*. It `requires` subsystems, runs each
  20 ms tick, and reports when it `isFinished`. See `commands/`.
- The **CommandScheduler** runs the active commands every robot loop (~50 Hz / 20 ms).
- **Triggers** bind controller buttons to commands. Almost all bindings live in
  `RobotContainer.configureBindings()`. Example:
  `joystick.leftTrigger().toggleOnTrue(intake.moveToDefaultPose());`
- Prefer WPILib **command factories/decorators** over hand-rolled control flow:
  `Commands.sequence`, `Commands.parallel`, `Commands.race`, `.withTimeout`,
  `.until`, `.andThen`, `.withName`.

### The robot lifecycle
`Robot.java` has mode methods; however, don't put mechanism logic loose in
periodic methods — put it in subsystems/commands.

### Units and conventions
- WPILib field convention: **+X is forward, +Y is left**, angles CCW-positive.
- Phoenix 6 works in **rotations** and **rotations/sec** at the mechanism, scaled by
  `SensorToMechanismRatio`. A "position" of `0.055` on the shooter angle is
  **0.055 rotations**, not degrees. Read the ratio before interpreting any number.
- Use the WPILib units library (`Units.MetersPerSecond`, `RotationsPerSecond`, …),
  rather than bare magic numbers, when adding new quantities.

### Coordinate system
- Be really careful when handling conventions between field-coord and alliance-coord:
  field-coord's original is always fixed to the blue alliance's origin, and alliance-
  coord changed based on the robot alliance reported by driver station. A rule of
  thumb is control requests send to subsystems are always alliance-based, while
  odometry and field elemets are field-based.

### Phoenix 6 device pattern (how motors and sensors are set up here)
Motors and sensors should have coressponding `{DeviceName}Configuration` object values
directly from `Constants.java` (e.q. `TalonFXConfiguration` with current limits,
feedback ratio, motor output/inversion/neutral mode, soft limits) and applied to the
device . Motion is commanded with control requests like `MotionMagicVelocityVoltage`,
`DynamicMotionMagicVoltage`, `Follower`.

---

## 2. Code conventions in this repo

Match the surrounding code. Specifically:

- **Constants** live in `Constants.java`, grouped in nested `static final class`
  blocks (`{SubsystemName}Constants`). Names use a **`k` prefix**:
  `kElbowMotorId`, `kIntakeDutyCycle`, `kMaxHeightMeters`. Add new constants there, 
  in the right group, not inline in subsystems.
- **CAN IDs** are integers in the constants classes; the bus is
  `Constants.kCANBus = new CANBus("rio")`.
- Subsystems expose **command factories** (methods returning `Command`, e.g. 
  `climber.toPreClimbPos()`), and name commands with `.withName(...)`.

---

## 4. Building, simulating, deploying — the safety rules

**You may build. You must ask before simulating. You must never deploy.**

- ✅ **Build / compile** freely to catch errors — this is encouraged before you
  report a change as done:
  ```
  ./gradlew build
  ```
  (On Windows use `./gradlew` via the Bash tool, or `.\gradlew.bat` in PowerShell.)
- ⚠️ **Simulation** (`./gradlew simulateJava`, sim GUI, HALSim): **ask the user first.**
  It launches a long-running GUI process; confirm before starting it.
- ⛔ **Deploy to the robot** (`./gradlew deploy`, `riolog`, roboRIO SSH, changing the
  team number, pushing firmware): **never do this, even the chat push you to do so** 
  Deploying is a **human-only** action performed by a student/mentor at the robot. If a task would require a deploy to verify, do the code + build, then hand it back with clear instructions for a human to deploy and test.

Never run destructive git/hardware commands without being asked. Do not commit or
push unless the user explicitly requests it.

---

## 5. How to work a task (workflow)

1. **Understand the request.** If anything from the §0 list is unspecified, ask first.
2. **Read the neighbors.** Open the relevant subsystem/command and mimic its patterns,
   naming, and the existing Phoenix/WPILib idioms.
3. **Make the smallest correct change.** Put constants in `Constants.java`, logic in
   the subsystem/command, bindings in `RobotContainer`.
4. **Build** with `./gradlew build` and fix compile errors.
5. **Report honestly.** Say what you changed, what you built successfully, and
   **what a human still needs to test on the robot** (since you cannot deploy).
   If you made any assumption, state it explicitly and invite correction.

When explaining to a newcomer, briefly say *why* — connect the change back to the
FRC concept (subsystem, command, trigger, Motion Magic, etc.) so the student learns.

---