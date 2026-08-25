# Joystick

## Summary

This session covers the following information:
- Code organization
- Starter project
- Joystick inputs
- Exercise: Develop periodic robot code to control a motor at a speed
  - Configure the motor controller
  - Set the position/velocity calibration

## Starter Project

We will start by using the WPILib Visual Studio Code application to generate a new project. 

Generating the project:
- Open WPILib VSCode
- Click the WPILib icon in VSCode
- Select: WPILib: Create A New Project
- Template project
- OpMode Robot
- Set base folder: Set this to a location that you will be able to remember
- Project name: timed-robot
- Team number: 538
- Enable desktop support - Check this box

There are many files and folders in the project.
The most interesting files will be within the `src/main/java/first/robot folder`.
For this session, we will be using `Robot.java` and `opmode/MyTeleop.java`.

Before we get started, initialize a git repository in the directory.
Open the source control tab on the left.
Select to initialize a git repo.
Add all files

Write a message: "Empty starter project"
Select 'Commit'

## Robot Program Structure

OpModes are new in 2027 WPILib.
OpMode is short for Operational Mode, and WPILib provides this capability as a new method to organize code based on desired subsets of capability.
We will learn more about how to use OpModes as we go further in the lessons.

First, we will review `Robot.java`.
The `package` line at the top the file indicates where in the folder structure the file can be found. 
In this case, the file is in the `frc/robot` directory.
```java
package frc.robot;
```

Lines starting with `import` are used to reference code that exists in places other than the file you have open.
In this case, the file references the `OpModeRobot` code from WPILib.
```java
import edu.wpi.first.wpilibj.OpModeRobot;
```

Lines that start with two-forward slashes (`//`) are comments - which means they are not executed code. 
Rather, they are notes to/from the person who wrote the code. 
Examples include:
```java
// Copyright (c) FIRST and other WPILib contributors.
// Open Source Software; you can modify and/or share it under the terms of
// the WPILib BSD license file in the root directory of this project.
```

Other comments are marked by starting with a single forward slash and an asterisk (`/*`) and ending with an asterisk and a single forward slash (`*/`). 
These are multiline comments. 
Examples include:
```java
/** This function is called exactly once when the DS first connects. */
```

The bulk of the code in the file is in a `class`, specifically the class named `Robot`. 
The `class` consists of everything between the opening and closing lines, contained in a set of curly braces (`{}`):
```java
public class Robot extends OpModeRobot {
    ...
}
```

The `Robot` class is the first item created.
It will serve as a container for all mechanisms, controllers, or any other objects that could be used in different modes.

Next we will review `MyTeleop.java`.
The `@Teleop` symbol indicates that this class can be used as a teleop OpMode.
Options for OpModes are: `@Teleop`, `@Autonomous`, and `@Utility`.
The OpMode constructor receives a reference to the Robot which may be saved into a local private variable for later use.
```java
@Teleop
public class MyTeleop extends PeriodicOpMode {
  private final Robot robot;

  /** The Robot instance is passed into the opmode via the constructor. */
  public MyTeleop(Robot robot) {
    this.robot = robot;
  }
  ...
}
```

Other functions:
- `disabledPeriodic()` - Called periodically (on every DS packet) while the robot is disabled.
- `start()` Called once when the robot is enabled.
- `periodic()` Called periodically (set time interval) while the robot is enabled.
- `end()` Called when the robot is disabled (after previously being enabled). 
- `close()` Called when the opmode is de-selected / no additional methods will be called. 

## Joystick Input

We will add code to allow the robot to accept inputs from a joystick.
Any device that can connect to your computer that runs Driver Station via a USB connection (more or less) can be considered a `Joystick`, the most common being flight-sticks and gamepads.
The joystick will be created as a public object in `Robot.java` so that it is available in the teleop OpMode we are creating as well as any other mode we choose to create later.

In `Robot.java`:
```java
public final Joystick joystick = new Joystick(0);
```

Code Notes:
- `public`: the object will be accessable to anyone who has access to the `Robot` object
- `Joystick`: the `type` of the object that is being created
- `joystick`: the name of the object that is created
- `new`: a keyword that creates new objects
- `Joystick(0)`: the constructor which creates an object that can talk to USB input channel 0

Moving to `opmode/MyTeleop.java`, we will read from a button input on the joystick.
The `robot` object is accessible in `MyTeleop.java` because the constructor copied a reference to it to a local variable.
We want to read from the joystick during the `periodic` function which will allow the code to take an action.

```java
@Override
public void periodic() {
  /* Called periodically (set time interval) while the robot is enabled. */
  boolean isPressed = robot.joystick.getRawButton(1);
  SmartDashboard.putBoolean("Button", isPressed);
}
```

Code Notes:
- `boolean isPressed`: Creates a new variable named `isPressed` which is of the type `boolean`
- A `boolean` may be either `true` or `false`
- `robot.joystick`: we access the joystick by using dot-notation to get its public reference from the robot
- `getRawButton(1)`: Returns the state of button #1 as a boolean, `true` if pressed
- `SmartDashboard.putBoolean`: Publishes the state to telemetry
- The `periodic` function will be called when this OpMode is active at 50Hz, or every 20 milliseconds (by default)

## Simulate

To test the code, we can run the simulation.
Click the WPILib icon and select Simulate Robot Code.

<img src="https://i.imgur.com/o9Zkm1T.png" />

A selection pop-up will show up after the code builds.
In this case, we will just keep the default Sim GUI selection.
Click OK

<img src="https://i.imgur.com/ltZNvZL.png" />

Drag system joystick or keyboard to Joystick[0]
Keyboard inputs: z, x, c, v for buttons 0-3

<img src="https://i.imgur.com/wQL6Idp.png" />

In the upper left, select Teleoperated:

<img src="https://i.imgur.com/cHMnhZP.png" />

Then select the dropdown and click `MyTeleop`

<img src="https://i.imgur.com/s3GY0nI.png" />

OpMode will change from `NONE` to `GOOD`

<img src="https://i.imgur.com/oGpU7jb.png" />

Now that the OpMode is active, you can find the `Button` telemetry in the Network Tables Transitory Values section:

<img src="https://i.imgur.com/tmnjaSm.png" />

Click `x` (assuming a keyboard input) and see if the telemetry changes.
Also watch the boxes on the Joysticks window to see if the inputs light up.

## Check in the code

Select the Source Control button on the left bar.
You should see a list of all modified files.
Selecting any one of those files will open up a diff view showing the FROM/TO code changes.

Add all files.
Add a message describing what you changed.
Commit the code.
