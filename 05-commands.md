# Commands V3

## Summary

In this lesson, we cover:
- Commands Overview
- Organizing device with mechanisms
- Implementing commands
- Command scheduling

## Commands Overview

The WPILib command framework provides a way to organize code into executable chunks of functionality that can be composed flexibly.
The commands framework has been on Version 2 for some years which is good, but has had some useability issues.
[Version 3](https://github.com/wpilibsuite/allwpilib/blob/main/design-docs/commands-v3.md) has just been released in the 2027 game which improves greatly on the flexibility and ease of implementation.

The command framework provides a convenient method to write code that is event driven.
The general approach is:
- Create mechanism code that controls one or more motors
- Write command code for those mechanisms that exposes a function
  - Motor on
  - Command motor to RPM
- Create higher level mechanisms that provide commands to integrate mechanism functionality to meet robot task needs
  - Navigate to a point and then fire
  - Point in a direction, then pick up a game piece
  - etc

Op Mode code is then created that connects controller inputs to commands, or to configure which commands execute in an auto.
Utility op modes are created for each mechanism to ensure commands and mechanisms work properly.
Once commands are proven to function, they can be integrated into teleop op modes.
Autonomous op modes are created from functional teleop commands.

A new aspect in commands v3 is the ability to use coroutines to control command execution flow.
Coroutines are a form of cooperative multi-tasking that allows a function to choose to yield execution to other functions, starting up at the same point on the next cycle.
This opens up a lot of flexibility that we will get to over time.

## Mechanisms

This lesson will make use of the op-telem-fault project that was created in lesson 4.
Open up the project in VSCode and make sure you have committed all prior work.

Open the WPILib Vendor Dependency tab on the left side and configure for the following dependencies:
- Delete Commands V2 (if installed)
- Enable Commands V3
- Enable RevLIB

Check those vendorlib changes into the repo.
**Best Practice Note:** Always commit vendorlib changes separate from other code changes.

In the explorer window, create a new folder under the robot directory called `mechanisms`.
We will create a simple mechanism to turn a motor on and off.

