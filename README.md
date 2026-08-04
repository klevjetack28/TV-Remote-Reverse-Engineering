# TV Remote Reverse Engineering

Reverse engineering a television remote using a logic analyzer,
decoding its infrared signal timing, and reproducing its commands
with an Arduino.

[Project image or demonstration GIF]

## Overview

### The repetitive TV sequence I wanted to automate
When my wife and I need some background noise, we play the show Modern Family on Hulu. I figured I could get an Arduino to reproduce those same actions as a remote and the project was born.

### Why I chose to reverse engineer the physical remote
I knew it was possible, so I wanted to figure out how it was possible.

### How the logic analyzer became the central tool
The Logic Analyzer is one way to solve the project. It just happened to be the first tool that popped into my head at he time. Another way might be firing the remote at your own reciever and copying the input. The easiest and friendliest way to reproduce this is with the Roku API.

### What the completed system can do
The completed system is able to imitate each button on the remote and pair key presses with defined delays to creat a chain of keypresses. This automates repetitive sequences such as volume up 10.

## Project Goals

- Capture the remote's electrical signals
- Identify the timing structure of each command
- Convert captured samples into usable timing data
- Reproduce the commands with an Arduino
- Combine commands into programmable macros

## System Overview

Remote PCB
    ↓
Logic Analyzer
    ↓
Binary Capture Files
    ↓
C Signal Parser
    ↓
Arduino Timing Arrays
    ↓
Arduino + IR Driver Circuit
    ↓
Television

## Hardware

| Component | Purpose |
|---|---|
| Arduino Uno | Signal generation and macro control |
| Logic analyzer | Capturing remote output |
| IR LED | Reproducing commands |
| TIP102 transistor | Driving the IR LED |
| 4×4 keypad | Selecting macros and controls |
| 16×2 LCD | User interface |
| Resistors and wiring | Supporting circuitry |

## How It Works

### 1. Inspecting the Remote

Open the remote, identify the microcontroller, find its datasheet,
and locate the output used to drive the IR LED. If you cannot read the chip number you can trace the IR LED leads back to the chip to find the lead. 

### 2. Capturing the Signal

Connect the logic analyzer to the remote and capture each button
press. Public information for these remotes reveals they send their IR signals at 38kHz. This is importants because you need to reliables read that signal so you set your logic analyzer to match the frequency of the remote. If you are reading at too high a frequency you are going to get duplicate data, but to low and you risk missing information. 

- exported binary captures
- one capture file per command

### 3. Parsing the Captures
The first task of the Program was to get parse out just the signal from when it starts to when it ends. 
Describe the C program that:

- reads the binary files
- counts consecutive high and low samples
- converts sample counts into microseconds
- identifies marks and spaces
- handles timing variability
- outputs Arduino-compatible arrays

Include a small before-and-after example.

### 4. Reproducing the Signal

Explain how the Arduino uses hardware timers to generate the
38 kHz carrier and turns it on during marks and off during spaces.

### 5. Building the IR Driver Circuit

Describe the weak initial transmission, why direct GPIO drive was
insufficient, and how the TIP102 allowed the Arduino to control a
higher-current IR LED circuit.

### 6. Creating Macros

Explain how individual commands were combined into sequences,
including:

- command ordering
- command-specific delays
- longer delays for app loading
- the 4×4 keypad
- the LCD interface

## Signal Fundamentals

### Carrier Frequency

A short explanation of the 38 kHz carrier.

### Marks and Spaces

A clear explanation of:

- mark = carrier transmitted
- space = carrier absent
- duration encodes information

### Timing Tolerance

Explain why captured values may be close to, rather than exactly,
the nominal protocol values.

## Software

### Signal Parser

Language, inputs, outputs, and responsibilities.

### Arduino Firmware

Signal playback, timers, keypad input, LCD output, and macros.

### Automation Scripts

Any Bash scripts used to process all captures, generate source data,
or copy generated arrays.

## Challenges

### Parsing Noisy Timing Data

What happened and how you normalized the values.

### Weak IR Transmission

Why the LED only worked near the receiver and how the driver circuit
fixed it.

### Macro Timing

Why commands could not be sent immediately after one another.

### Limited Arduino Pins

Why you replaced individual buttons with a matrix keypad.

## Results

State concretely what works now:

- captured commands can be reproduced
- the television responds from normal viewing distance
- macros can execute multiple commands
- different delays can be assigned
- controls are available through the keypad

Add a demonstration image or video.

## What I Learned

- Using a logic analyzer for hardware investigation
- Reading microcontroller datasheets
- Understanding IR carrier modulation
- Processing sampled digital signals
- Using hardware timers
- Designing a transistor-driven LED circuit
- Building repeatable tooling around reverse-engineering work

## Future Development

- Independant circuit. A device that does not rely on the Arduino and runs it's own circuit.
