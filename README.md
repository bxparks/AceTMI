# AceTMI

Unified interface for selecting different implementations for communicating
with a TM1637 LED controller chip on Arduino platforms. Uses C++ templates to
achieve minimal or zero-cost runtime overhead for the abstraction. The code was
initially part of the [AceSegment](https://github.com/bxparks/AceSegment)
library, but was extracted into a separate library so that it can be shared with
other projects. The TM1637 uses a protocol that is electrically very similar to
I2C.

The "TMI" acronym is an invention of this library so that the TM1637 protocol
can be referenced by a name. It stands for "Titan Micro Interface" (analogous to
Serial Peripheral Interface) which is named after the compnay Titan Micro
Electronics which manufactures the TM1637 chip.

This library provides the following C++ class templates:

* `SoftTmiInterface`
* `SoftTmiFastInterface`

**Version**: 0.1 (2021-06-24)

**Changelog**: [CHANGELOG.md](CHANGELOG.md)

**Status**: Works, but needs documentation.

## Table of Contents

* [Installation](#Installation)
    * [Source Code](#SourceCode)
    * [Dependencies](#Dependencies)
* [Documentation](#Documentation)
* [Usage](#Usage)
    * [Include Header and Namespace](#HeaderAndNamespace)
* [System Requirements](#SystemRequirements)
    * [Hardware](#Hardware)
    * [Tool Chain](#ToolChain)
    * [Operating System](#OperatingSystem)
* [License](#License)
* [Feedback and Support](#FeedbackAndSupport)
* [Authors](#Authors)

<a name="Installation"></a>
## Installation

The latest stable release will eventually be available in the Arduino IDE
Library Manager. Search for "AceTMI". Click install. (It is not there
yet.)

The development version can be installed by cloning the
[GitHub repository](https://github.com/bxparks/AceTMI), checking out the
`develop` branch, then manually copying over the contents to the `./libraries`
directory used by the Arduino IDE. (The result is a directory named
`./libraries/AceTMI`.)

The `master` branch contains the stable release.

<a name="SourceCode"></a>
### Source Code

The source files are organized as follows:
* `src/AceTMI.h` - main header file
* `src/ace_tmi/` - implementation files
* `docs/` - contains the doxygen docs and additional manual docs

<a name="Dependencies"></a>
### Dependencies

The main `AceTMI.h` does not depend any external libraries.

The "Fast" version (`SoftTmiFastInterface.h`)
depend on one of the digitalWriteFast libraries, for example:

* https://github.com/watterott/Arduino-Libs/tree/master/digitalWriteFast
* https://github.com/NicksonYap/digitalWriteFast

<a name="Documentation"></a>
## Documentation

* this `README.md` file.
* [Doxygen docs](https://bxparks.github.io/AceTMI/html)
    * On Github pages.

<a name="Usage"></a>
## Usage

<a name="HeaderAndNamespace"></a>
### Include Header and Namespace

In many cases, only a single header file `AceTMI.h` is required to use this
library. To prevent name clashes with other libraries that the calling code may
use, all classes are defined in the `ace_tmi` namespace. To use the code without
prepending the `ace_tmi::` prefix, use the `using` directive:

```C++
#include <AceTMI.h>
using ace_tmi::SoftTmiInterface;
```

The "Fast" versions are not included automatically by `AceTMI.h` because they
work only on AVR processors and they depend on a `<digitalWriteFast.h>`
library. To use the "Fast" versions, use something like the following:'

```C++
#include <AceTMI.h>

#if defined(ARDUINO_ARCH_AVR)
  #include <digitalWriteFast.h>
  #include <ace_tmi/SoftTmiFastInterface.h>
  using ace_tmi::SoftTmiFastInterface;
#endif
```

<a name="SystemRequirements"></a>
## System Requirements

<a name="Hardware"></a>
### Hardware

This library has Tier 1 support on the following boards:

* Arduino Nano (16 MHz ATmega328P)
* SparkFun Pro Micro (16 MHz ATmega32U4)
* SAMD21 M0 Mini (48 MHz ARM Cortex-M0+)
* STM32 Blue Pill (STM32F103C8, 72 MHz ARM Cortex-M3)
* NodeMCU 1.0 (ESP-12E module, 80MHz ESP8266)
* WeMos D1 Mini (ESP-12E module, 80 MHz ESP8266)
* ESP32 dev board (ESP-WROOM-32 module, 240 MHz dual core Tensilica LX6)
* Teensy 3.2 (72 MHz ARM Cortex-M4)

Tier 2 support can be expected on the following boards, mostly because I don't
test these as often:

* ATtiny85 (8 MHz ATtiny85)
* Arduino Pro Mini (16 MHz ATmega328P)
* Teensy LC (48 MHz ARM Cortex-M0+)
* Mini Mega 2560 (Arduino Mega 2560 compatible, 16 MHz ATmega2560)

The following boards are **not** supported:

* Any platform using the ArduinoCore-API
  (https://github.com/arduino/ArduinoCore-api). For example:
    * Nano Every
    * MKRZero
    * Raspberry Pi Pico RP2040

<a name="ToolChain"></a>
### Tool Chain

* [Arduino IDE 1.8.13](https://www.arduino.cc/en/Main/Software)
* [Arduino CLI 0.14.0](https://arduino.github.io/arduino-cli)
* [SpenceKonde ATTinyCore 1.5.2](https://github.com/SpenceKonde/ATTinyCore)
* [Arduino AVR Boards 1.8.3](https://github.com/arduino/ArduinoCore-avr)
* [Arduino SAMD Boards 1.8.9](https://github.com/arduino/ArduinoCore-samd)
* [SparkFun AVR Boards 1.1.13](https://github.com/sparkfun/Arduino_Boards)
* [SparkFun SAMD Boards 1.8.3](https://github.com/sparkfun/Arduino_Boards)
* [STM32duino 2.0.0](https://github.com/stm32duino/Arduino_Core_STM32)
* [ESP8266 Arduino 2.7.4](https://github.com/esp8266/Arduino)
* [ESP32 Arduino 1.0.6](https://github.com/espressif/arduino-esp32)
* [Teensyduino 1.53](https://www.pjrc.com/teensy/td_download.html)

<a name="OperatingSystem"></a>
### Operating System

I use Ubuntu 20.04 for the vast majority of my development. I expect that the
library will work fine under MacOS and Windows, but I have not explicitly tested
them.

<a name="License"></a>
## License

[MIT License](https://opensource.org/licenses/MIT)

<a name="FeedbackAndSupport"></a>
## Feedback and Support

If you have any questions, comments and other support questions about how to
use this library, use the
[GitHub Discussions](https://github.com/bxparks/AceTMI/discussions)
for this project. If you have bug reports or feature requests, file a ticket in
[GitHub Issues](https://github.com/bxparks/AceTMI/issues). I'd love to hear
about how this software and its documentation can be improved. I can't promise
that I will incorporate everything, but I will give your ideas serious
consideration.

Please refrain from emailing me directly unless the content is sensitive. The
problem with email is that I cannot reference the email conversation when other
people ask similar questions later.

<a name="Authors"></a>
## Authors

Created by Brian T. Park (brian@xparks.net).
