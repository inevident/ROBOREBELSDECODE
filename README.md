# roborebels 14450 - decode (2025-26)

code for our ftc robot this season. we took it to worlds (lovelace division) in 2026.

this is built on first's ftc sdk ([FtcRobotController](https://github.com/FIRST-Tech-Challenge/FtcRobotController)), so most of the repo is their stuff, and so are the 2020-2024 commits in the history. everything we wrote is in `TeamCode/src/main/java/org/firstinspires/ftc/teamcode/`.

## branches

- `v3-worlds` (default) - what ran at worlds. turret, shooting while moving, auto aim in auto
- `v2` - states code (march 2026), no turret yet
- `master` - early season

## where stuff is

if you only read one thing, read `Subsystems/Limelight.java` and `updateAimingSystem()` in `Robot.java`. that's the shooting while moving part.

- `Robot.java` - sets up the hardware and subsystems. `updateAimingSystem()` gets the robot's velocity from pedro, rotates it into the robot's frame using heading, converts in/s to m/s, and passes it to the limelight code. then it sets turret power from the aim pid
- `Subsystems/Limelight.java` - the aiming math
  - splits robot velocity into the part toward the goal and the part sideways to it (turret angle + where the tag is in the camera)
  - distance to the goal comes from the tag's vertical angle and the camera-to-tag height
  - lead angle is sideways velocity times estimated flight time, so the turret aims ahead of the goal
  - flywheel speed comes from distance, minus a correction for how fast we're moving toward or away from the goal
  - turret pid (p, d, plus a static friction term), wraps angles to 0-360, and holds the last target if it loses the tag
  - the rgb light shows whether it's locked on
- `Subsystems/Outtake.java` - two motor flywheel and the turret. flywheel speed control is feedforward (kV * target) plus p on the error. turret ticks to degrees is in here too
- `Subsystems/Intake.java` - intake motor plus the flapper and cycler servos. small state machine that cycles the gate to feed balls into the shooter
- `TeleOp/BaseTeleop.java` - driver controls. field centric drive off the imu. shooter speed comes from the limelight by default, and gamepad2 can switch to manual presets if vision is acting up
- `Auton/` - `BaseAuton` has the shoot and intake helpers, `BaseClose15` and `BaseFar15` are the actual autos. the red and blue files are just wrappers, blue mirrors the poses
- `AllianceColor.java` - all the red vs blue differences (limelight pipeline, aim offset, turret home angle, pose mirroring)
- `pedroPathing/Constants.java` - our tuned pedro constants. `Tuning.java` is pedro's tuning opmodes from their quickstart, not ours

## testing / tuning

no unit tests, everything got tested on the robot. most constants are `@Configurable` so we could change them live in panels without redeploying.

- `Testing/FlywheelPIDTesting.java` - set a target speed in panels and graph target vs actual vs power. this is how kV and kP got tuned
- `Testing/ServoTest.java` and `Testing/SensorTest.java` - finding servo positions and checking sensors

## running it

open it in android studio and deploy `TeamCode` to the control hub. hardware config names have to match: `fl fr bl br`, `flywheel1 flywheel2`, `turret`, `intake`, `flapper`, `cycler`, `tiltA tiltB`, `rgb`, `limelight`, `imu`.
