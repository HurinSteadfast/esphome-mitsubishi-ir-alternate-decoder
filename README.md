# ESPHome Mitsubishi IR Alternate Decoder

An alternate receive decoder for the ESPHome Mitsubishi IR climate component.

This project was created after discovering that ESPHome incorrectly decoded temperature setpoints transmitted by certain Mitsubishi Electric Fahrenheit remotes. The result was Home Assistant displaying incorrect temperatures (sometimes jumping to 90–100°F) when the physical remote was used.

This decoder correctly interprets the temperature encoding used by these remotes while remaining fully compatible with the existing ESPHome Mitsubishi transmitter.

## Tested Hardware

### Indoor Unit
- Mitsubishi Electric MSZ-JP09WA

### Remote
- Mitsubishi Electric MS16A (sticker: **MS16A 1N2M**)
- Fahrenheit mode
- 61°F–88°F temperature range
- 1°F temperature increments

## Problem

The stock ESPHome Mitsubishi receiver assumes the temperature byte is encoded as a simple Celsius offset.

That assumption is not valid for the MS16A Fahrenheit remote.

As a result:

- Home Assistant would sometimes display impossible temperatures (99°F, 100°F, etc.)
- After partially masking the byte, temperatures no longer jumped but every other button press collapsed to the same temperature because the decoder still assumed whole Celsius steps.

## Solution

This decoder uses a lookup table for the received temperature byte.

Each encoded value is translated into the correct Fahrenheit setpoint and then converted into Celsius internally before being passed to ESPHome/Home Assistant.

The result is correct synchronization of every 1°F button press from the physical remote.

## Current Status

### Receive

✅ Fully tested

- Physical remote changes immediately update Home Assistant
- All temperatures from 61°F through 88°F decode correctly
- No erroneous 90–100°F jumps
- Every 1°F increment is preserved

### Transmit

The existing ESPHome Mitsubishi transmitter appears to function correctly with this hardware.

While transmit behavior has been tested successfully in normal operation, this project specifically addresses receive-side decoding.

## Installation

This project is intended to be used as an ESPHome External Component.

Example:

```yaml
external_components:
  - source:
      type: git
      url: https://github.com/HurinSteadfast/esphome-mitsubishi-ir-alternate-decoder
    components:
      - mitsubishi
```

Then configure your climate component normally:

```yaml
climate:
  - platform: mitsubishi
    transmitter_id: remote_transmitter_1
    receiver_id: remote_receiver_1
```

## Why this project exists

The goal is not to replace ESPHome's Mitsubishi component.

The goal is to provide an alternate decoder for Mitsubishi remotes whose temperature encoding differs from the implementation currently included in ESPHome.

If additional Mitsubishi remote variants are discovered, they can be incorporated here and ultimately proposed upstream to ESPHome.

## Contributing

If you have another Mitsubishi remote that exhibits incorrect temperature synchronization, please open an issue and include:

- Indoor unit model
- Remote model number
- Whether the remote operates in °F or °C
- ESPHome logs showing received AEHA frames
- A description of the observed behavior

This information will help determine whether your remote uses the same encoding or represents another protocol variant.
