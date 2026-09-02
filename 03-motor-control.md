# Motor Controllers

## Summary

We review the following:
- Control system components
- Programming REV Sparkmax motor controllers
- Programming CTRE TalonFX motor controllers

## Major Control System Components

You can read the [Control System Overview from the WPILib docs](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html) for an overview. Our team has a history of using the REV components over the CTRE components, however we have been moving more towards CTRE in recent years.

- [Battery](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#robot-battery)
- [Main Breaker](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#a-circuit-breaker)
- [REV Power Distribution Hub (PDH)](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#rev-power-distribution-hub)
- [Limelight Systemcore](https://docs.wpilib.org/en/latest/docs/software/systemcore-info/systemcore-introduction.html)
- [VH-109 Radio](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#rev-radio-power-module]
- [Robot Signal Light (RSL)](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#robot-signal-light)

Lesser used components:
- [CTRE Power Distribution Panel (PDP)](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#ctre-power-distribution-panel)
- [CTRE Voltage Regular Module (VRM)](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#ctre-voltage-regulator-module)
- [REV Pneumatics Hub (PH)](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#rev-pneumatic-hub)

## Motor Controllers

When programming, you do not actually command or instruct the *motor* - your code acts on the *motor controller*.
The *motor controller* receives the full power available from the Power Distribution Hub and sends to the *motor* only what is needed to fulfill the instruction it is working to complete.

Some motors, like the Kraken X60 have an integrated motor controller (the [TalonFX](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#talonfx-motor-controller)), so the motor controller and motor appear as a single device.
Other motors have a completely separate motor controller device, so it is obvious the devices are independent, like the REV NEO motor and [REV Spark MAX](https://docs.wpilib.org/en/stable/docs/controls-overviews/control-system-hardware.html#spark-max-motor-controller) motor controller.

## Controlling Motors

We will add two motors to control, one NEO and one Kraken X60.
First, go to the WPILib extensions tab by selecting the WPILib icon on the left.
Install CTRE-Phoenix (v6) and REVLib vendor dependencies.
The project will rebuild automatically when they are incorporated.

<img src="https://i.imgur.com/HaxtsRT.png" />

Commit the two files from the vendordeps folder to the git repo.

### SparkMax

https://docs.revrobotics.com/brushless/spark-max/overview

Create two new variables to Robot.java after the Joystick variable.
```java
  public final SparkMax motor = new SparkMax(1, 1, MotorType.kBrushless);
  private final SparkMaxConfig config = new SparkMaxConfig();
```

Code Notes:
- `SparkMax motor`: This defines the interface to the motor controller
- `new SparkMax(1, 1, MotorType.kBrushless)`: Constructor of a brushless motor controller
  - 1 (first): The first `1` indicates which CAN bus this motor controller talks on
  - 1 (second): The second `1` represents the CAN ID of the SparkMax controller
  - `MotorType.kBrushless`: Most motors we are likely to use are brushless motors
- `SparkMaxConfig config`: This is the variable we use to configure a motor controller

Next we need to configure the SparkMax controller.
For now, we will put this type of configuration information inside `Robot.java` in the constructor.
This configuration represents the most common configuration items that we typically need to set for a SparkMax.
The values will depend on the specifics of the mechanism and intended operation.
Also see: [REV configuration parameter documentation](https://docs.revrobotics.com/brushless/spark-max/parameters)

```java
  public Robot() {
    config
      .smartCurrentLimit(50)
      .idleMode(IdleMode.kCoast)
      .inverted(false);
    config
      .encoder
        .positionConversionFactor(1.0)  // 1.0 means the encoder returns revolutions
        .velocityConversionFactor(1.0); // 1.0 means the encoder returns rpm
    config
      .closedLoop
        .pid(0.0, 0.0, 0.0)
        .feedbackSensor(FeedbackSensor.kPrimaryEncoder)
        .positionWrappingEnabled(false);

    motor.configure(config, ResetMode.kResetSafeParameters, PersistMode.kPersistParameters);
  }
```

Code Notes:
- `smartCurrentLimit(50)`: The motor controller will restrict voltage to limit current to about 50 amps
- `idleMode`: Motors can be configured to either coast or brake
- `inverted`: The default positive sense
- `positionConversionFactor`: Adds a scale factor to encoder position feedback
- `velocityConversionFactor`: Adds a scale factor to encoder velocity feedback
- `pid() and other parameters`: Sets closed loop feedback gains (if required)

The motor is now useable.
We want to ensure all motor controllers are commanded into a safe state when not in an OpMode.
This code will go in the `nonePeriodic` function.

```java
  public void nonePeriodic() {
    // Reset all motor controller configurations
    motor.setThrottle(0.0);
    // We would also want to reset any integrators here
  }
```

Now, let's hook this motor controller up to our joystick.
Go to the `MyTeleop.java` file and add to the `Periodic` function.
We are going to command the motor to full on when the button is pressed, and off when the button is released.

```java
  @Override
  public void periodic() {
    ...
	
    if (isPressed) {
      robot.motor.setThrottle(1.0);
    }
    else {
      robot.motor.setThrottle(0.0);
    }
  }

```

Simulate the robot code.
The simulation window will show two new devices, a motor and a motor encoder.

<img src="https://i.imgur.com/g4y8bMv.png" />

These can be expanded to see simulation parameters.
However, just because the simulation knows about the motor and encoder doesn't mean it will do anything.
You can see the setpoint change between 0 and 1.

<img src="https://i.imgur.com/JuPTxCZ.png" />

We will discuss adding simulation code in a later session to allow the motor and encoder to change when simulating.

### Kraken X60


Create two new variables to Robot.java after the Joystick variable.

```java
  public final TalonFX otherMotor = new TalonFX(3, CANBus.systemcore(1));
  private final TalonFXConfiguration otherMotorConfig = new TalonFXConfiguration();
```

Code Notes:
- `TalonFX otherMotor`: The type and name of the Kraken X60 motor controller
- `new TalonFX(3, CANBus.systemcore(1));`: First argument is the CAN ID, second argument is the CAN Bus
- `TalonFXConfiguration`: The TalonFX config variable, similar to the REV implementation

In our Robot constructor, we add the configuration details.
This configuration sets many of the common items that we will need to configure for TalonFX controlled motors.

```java
  public Robot() {
    ...
	
    otherMotorConfig
      .withMotorOutput(new MotorOutputConfigs()
        .withNeutralMode(NeutralModeValue.Brake))
      .withCurrentLimits(new CurrentLimitsConfigs()
        .withStatorCurrentLimit(50.0)
        .withStatorCurrentLimitEnable(true)
        .withSupplyCurrentLimit(50.0))
      .withClosedLoopGeneral(new ClosedLoopGeneralConfigs()
        .withContinuousWrap(true))
      .withFeedback(new FeedbackConfigs()
        .withFeedbackSensorSource(FeedbackSensorSourceValue.RotorSensor))
      .withSlot0(new Slot0Configs()
        .withKP(0.0)
        .withKI(0.0)
        .withKD(0.0));

    otherMotor.getConfigurator().apply(otherMotorConfig);
```

- CTRE Documentation on setting [current limits](https://v6.docs.ctr-electronics.com/en/stable/docs/hardware-reference/talonfx/improving-performance-with-current-limits.html)
  - Stator limit is directly proportional to torque and is a good candidate to start with

This time, we'll hook a joystick axis to the motor.
In `MyTeleop.java`:

```java
@Override
  public void periodic() {
    ...

    double trigger = robot.joystick.getRawAxis(0);
    robot.otherMotor.setThrottle(MathUtil.applyDeadband(trigger, 0.1));
  }
```

Code Notes:
- In this case we don't want noise from the joystick to transfer to the motor, so implement a [deadband](https://en.wikipedia.org/wiki/Deadband) using a MathUtil helper.
- Joysticks will often have double-sided axes (range -1.0 to 1.0), example: control stick
- Joysticks may also have single-sided axes (range 0.0 to 1.0), example: trigger
- The Driver Station application can be used to identify axis and button numbers

Simulate again and look for differences in the simulation output.
