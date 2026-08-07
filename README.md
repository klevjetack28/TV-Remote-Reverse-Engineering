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
press. Public information for these remotes reveals they send their IR signals at 38kHz. This is importants because you need to reliables read that signal so you set your logic analyzer to match the frequency of the remote. I chose to sample the keypresses at 100kHz to guarantee I got the whole signal. Each datafile is one keypress. because the scale is so small, it is easiest to program a script to parse an array of files than one big file determining when one ends and another begins. This is why we stick to one keypress one datafile. 

// all this means is when it pulses HIGH LOW HIGH LOW, my output might be HIGH HIGH HIGH LOW LOW HIGH HIGH. This guarentees we got the whole signal and didn't miss anything. The good part about this too is that we can calculate about how many extra signals are read to translate our 100kHz into the actual remote sequence with ~99% accuracy. This is because we know the frequency it is sending at 38kHz and we are reading at 100kHz so we divide what we are reading by what we are sending to find for every one tick the remote sends we read 2.6 ticks. This is why we have 1 or 2 extra reads. I ised this to reason that after the hello sequence when the remote sends high and low signals, not extended bursts, I need a way to tear back all those extra values. 


### 3. Parsing the Captures
The first task of the Program was to get parse out just the signal from when it starts to when it ends. 
The base signal is extracted by forming a string of 1's and 0's for my recipe function to cook. The recipe function is what is responsible for identifying the marks and spaces of the signal, and converting them back into their original 38kHz frequencies. Marks and spaces are terms used to define when the signal is pulsing and when it is hanging. A signal is primarily held high. This is why when marks are pulses; Sections of the signal alternating between high and low. Spaces are hanging; a continuous high. Each remote has a header to the keypress protocol. This header is commonly a long mark followed by a medium space and alternates for a good chunk before sending the selected sequence. My scripts counts each tick that the signal was high and adds it to an array before switching into the oposite reading mode. It goes through the whole signal and stores each mark and soace into a single array. The odd indexes are marks and even indexes are spaces. This one array stores the signal array as the number of ticks each mark or space in sequence lasts. This it because we can calculate the length in time a single tick is, calculate the total time because we have number of ticks and how much each tick is worth, and turn our signal that was retrieved at 100kHz into a signal that mimics the IR remote because we know how long each mark/space lasts. The final part of the script was packaging it all to copy into the Arduino program. The C script was responsible for outputting an arduino compatible array into it's own file. I used a shell script to finish the transaction by copying and concating them to the clipboard for me to seamlessly copy and paste into the Arduino program.

### 4. Reproducing the Signal
The remote it selft sends the signal in pulses. What this means is that during a make the remote is pulsing HIGH LOW HIGH LOW for the entire clip. Spaces would be pulsing LOWs providing the signal HIGH HIGH HIGH HIGH. This is important becuase I need a way to reproduce this pulse on the Arduino. The standard way is to turn on a pin and turn it back off but these function calls require too much time to execute at microsecond precision. Arduino has a defined registers for this exact purpose that we can manipulate. The way I solved this problem was using the registers to set the PWM of the pin at 38kHz and modulated the index so odd indexes it pulses and even indexes it hangs high.

The other difficult part of using the Arduino to reproduce the signal was the rating of the IR LED were not supported with a standard Arduino. 

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
