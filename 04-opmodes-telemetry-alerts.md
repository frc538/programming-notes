# Op Modes, Telemetry, and Faults or Alerts

## Summary

We learn about the following:
- Use of op modes to organize code and separate development
- Generating and interpreting telemetry
- Detecting faults and setting alerts for drivers

This lesson requires the use of WPILib 2027 Alpha 7 or later.
We will look at the different elements, and then create a project that uses them.

## Create a new template project

Open '2027_alpha7 WPILib VS Code'

- WPILib button
- WPILib: Create new project
- Template
- Java
- Op Mode Robot
- Pick your base library where other code lives: e.g. C:/robotics
- Project name: op-telem-fault
- Team number: 538
- Enable desktop support
- Create Project

Use the current window.

Open the project in Github Desktop and check in the blank template.

## Op Modes

### Background

You have seen a little bit of Op Modes with the starter project already.
Today we are going to explore a bit more of the details and ways we will be using Op Modes to organize code.
The biggest impact with the use of Op Modes during typical development will be the ability to fully develop and validate code for unique subsystems without conflicting with code for other subsystems.

A robot program may contain many different implementations of each type which are selectable from the driver station via drop-downs.
Example:

<a img="https://i.imgur.com/Z6S4u6m.png" />

<a img="https://i.imgur.com/JMWUv8G.png" />

There are three types of Op Modes, and these are some example uses:
- Autonomous
  - Simple auto 
	- Drive forward for 2 seconds
  - Complex auto
	- Drive to a scoring position, score
	- Drive to a scoring position, score, then collect more game pieces, repeat
    - Different options for different starting positions, or alliance strategies
	- Test versions that are not ready for prime-time
- Teleop
  - Manual mode
    - Simple mechanism control with direct operator inputs
    - Some closed loop control where necessary for basic control of the subsystems
  - Smarter modes
	- Some mechanisms are orchestrated with composite commands built from manual mode 
  - Smartest modes
    - Operator commands queue, interleave, or abort from autonomous composite commands
- Utility
  - Test modes for individual subsystems
  - Pre-teleop modes for testing composite commands
  - Prototype code for assessing control performance with different gains or parameters
  
All mechanisms will be developed with utility modes first in individual files.
Mechanism controls will graduate to a teleop mode when:
- integration testing is successful
- a control button/axis map is coordinated with the team (drive team if available)

Each op mode will be defined in an individual file.
To create an OpMode, create a class that *implements* `OpMode`, and place one of the following decorators on the line before the class definition: `@Teleop`, `@Autonomous`, or `@Utility`.

### Implement a Utility OpMode

We will now create a utility Op Mode.
In the explorer window, right-click on the opmode folder and click on `New File`.

<a img="https://i.imgur.com/je0os0x.png" />

Name the file `UtilityTest.java`.
A new window will open with some initial code that defines a class named similar to the filename.

<a img="https://i.imgur.com/iqX1slk.png" />

Add the word `@Utility` on the line above the class definition.
Add `extends PeriodicOpMode` between the class name and `{`.
You will need to add import lines (shown below), or click in the red-underlined words, type ctrl-. and select to import Utility.

<a img="https://i.imgur.com/6MZRvfc.png" />

Your file should look like this:
```java
package first.robot.opmode;

import org.wpilib.opmode.PeriodicOpMode;
import org.wpilib.opmode.Utility;

@Utility
public class UtilityTest extends PeriodicOpMode {
    
}
```

The OpMode will show up in the driver station automatically in the dropdown when you run it:
<a img="https://i.imgur.com/JMWUv8G.png" />

## Telemetry 

### Background

Telemetry allow us to publish data from the robot to a dashboard or recorded file.
[WPILib Telemetry documentation](https://github.com/wpilibsuite/allwpilib/blob/main/telemetry/doc/telemetry.md) gives insight into the design of the Telemetry API.

### Add a Counter to UtilityTest

We will add a counter that increments when the UtilityTest mode is enabled, and resets to zero any time the op mode is not enabled.
First, we add some variables inside the UtilityTest class:
```java
    private Robot robot;
    private final TelemetryTable telemetry = Telemetry.getTable("UtilityTest");
    private int counter = 0;
```

You will need to configure imports for `Robot`, `TelemetryTable`, and `Telemetry`.
```java
import org.wpilib.telemetry.Telemetry;
import org.wpilib.telemetry.TelemetryTable;

import first.robot.Robot;
```

Create a class constructor to initialize telemetry any time the OpMode is entered.
```java
    public UtilityTest(Robot robot) {
        this.robot = robot;

        telemetry.log("counter", counter);
    }
```

Create an `end` and `periodic` functions, including an `@Override` decorator over each function.
The `@Override` keyword informs the compiler that the function is intended to override a function that is inherited.
If you make a typo in the function name, the use of `@Override` will cause the compiler to report an error.

The `end` function will set counter to 0, and the `periodic` function will increment it each time it is called.

```java
    @Override
    public void end() {
        counter = 0;

        telemetry.log("counter", counter);
    }

    @Override
    public void periodic() {
        counter = counter + 1;

        telemetry.log("counter", counter);
    }
```

You may notice that we had to repeat the `telemetry.log()` call multiple times.
While that is not a big deal with a single variable, it might end up with a lot of duplicate lines.

We will move that log call to a helper function:
```java
    private void logTelemetry() {
        telemetry.log("counter", counter);
    }
```

Replace the other 3 locations where `telemetry.log()` is called with a call to `logTelemetry()`.

Simulate the code.
This time, we'll open up AdvantageScope to see the signal.
First, connect to the simulator.

<a img="https://i.imgur.com/NHWyxw7.png" />

If AdvantageScope sees the simulation data, the x axis will start growing.
Before the UtilityTest is selected, UtilityTest will not show up in the signal pick list.
Switch to Utility mode, and select UtilityTest.

AdvantageScope will show Telemetry->UtilityTest->counter in the signal pick list when you expand it.

<a img="https://i.imgur.com/CVhCeUY.png" />

Drag the counter signal to the 'Left Axis" area.

Enable the DriverStation.
The counter will start counting up, looking like a ramp on the plot.
Disable the DriverStation, the counter will reset to 0.

<a img="https://i.imgur.com/C4lBkW5.png" />

## Faults & Alerts
