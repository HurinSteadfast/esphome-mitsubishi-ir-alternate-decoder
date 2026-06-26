# ESPHome Mitsubishi IR Alternate Decoder

An alternate receive decoder for the ESPHome Mitsubishi IR climate component.

This project was created after discovering that ESPHome incorrectly decoded temperature setpoints transmitted by certain Mitsubishi Electric Fahrenheit remotes. The result was Home Assistant displaying incorrect temperatures, sometimes jumping into the 90–100°F range, when the physical remote was used.

This decoder correctly interprets the temperature encoding used by these remotes while remaining compatible with the existing ESPHome Mitsubishi transmitter.

## Tested Hardware

### Indoor Unit

- Mitsubishi Electric MSZ-JP09WA
- Manual family: MSZ-JP09WA / MSZ-JP12WA

### Remote

- Mitsubishi Electric MS16A
- Sticker text: MS16A 1N2M
- Fahrenheit mode
- 61°F–88°F temperature range
- 1°F temperature increments

## Problem

The stock ESPHome Mitsubishi receiver assumes the received temperature byte is encoded as a simple Celsius offset.

That assumption is not valid for this MS16A Fahrenheit remote.

As a result:

- Home Assistant could display impossible temperatures such as 99°F or 100°F.
- A simple nibble mask fixed the large jumps but lost 1°F precision.
- Every other button press could collapse into the same whole-Celsius value.

## Solution

This decoder uses a lookup table for the received temperature byte.

Each encoded value is translated into the correct Fahrenheit setpoint and then converted into Celsius internally before being passed to ESPHome/Home Assistant.

The result is correct synchronization of every 1°F button press from the physical remote.

## Current Status

### Receive

Fully tested.

- Physical remote changes update Home Assistant.
- All temperatures from 61°F through 88°F decode correctly.
- No erroneous 90–100°F jumps.
- Every 1°F increment is preserved.

### Transmit

The existing ESPHome Mitsubishi transmitter appears to function correctly with this hardware in normal use.

This project specifically addresses receive-side decoding.

## Installation

Use this repository as an ESPHome external component:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/HurinSteadfast/esphome-mitsubishi-ir-alternate-decoder
    components:
      - mitsubishi
```

Then configure the Mitsubishi climate component normally:

```yaml
climate:
  - platform: mitsubishi
    name: Mitsubishi Mini-Split
    transmitter_id: remote_transmitter_1
    receiver_id: remote_receiver_1
```

## Why this project exists

The goal is not to replace ESPHome's Mitsubishi component.

The goal is to provide an alternate receive decoder for Mitsubishi remotes whose temperature encoding differs from the implementation currently included in ESPHome.
