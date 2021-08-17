# AceTMI

Unified interface for selecting different implementations for communicating
with a TM1637 LED controller chip on Arduino platforms. Uses C++ templates to
achieve minimal or zero-cost runtime overhead for the abstraction.  In more
technical terms, the library provides compile-time polymorphism instead of
runtime polymorphism to avoid the overhead of the `virtual` keyword.

The code was initially part of the
[AceSegment](https://github.com/bxparks/AceSegment) library, but was extracted
into a separate library for consistency with the
[AceWire](https://github.com/bxparks/AceWire) and
[AceSPI](https://github.com/bxparks/AceSPI) libraries. It provides the following
implementations:

* `SimpleTmiInterface.h`
    * Implements the TM1637 protocol using `digitalWrite()`.
* `SimpleTmiFastInterface.h`
    * Implements the TM1637 protocol using `digitalWriteFast()`.

The "TMI" acronym was invented by this library to name the protocol used by the
TM1637 chip. It stands for "Titan Micro Interface", named after Titan Micro
Electronics which manufactures the TM1637 chip. The TM1637 protocol is
electrically very similar to I2C, but different enough that we cannot use the
`<Wire.h>` I2C library.

The protocol implemented by this library works only for the TM1637 chip as far
as I know. Most people will want to use the `Tm1637Module` class in the
[AceSegment](https://github.com/bxparks/AceSegment) library instead of using
this lower-level library.

**Version**: 0.2 (2021-07-30)

**Changelog**: [CHANGELOG.md](CHANGELOG.md)

**See Also**:
* https://github.com/bxparks/AceSPI
* https://github.com/bxparks/AceWire

## Table of Contents

* [Installation](#Installation)
    * [Source Code](#SourceCode)
    * [Dependencies](#Dependencies)
* [Documentation](#Documentation)
* [Usage](#Usage)
    * [Include Header and Namespace](#HeaderAndNamespace)
    * [Unified Interface](#UnifiedInterface)
    * [SimpleTmiInterface](#SimpleTmiInterface)
    * [SimpleTmiFastInterface](#SimpleTmiFastInterface)
    * [Storing Interface Objects](#StoringInterfaceObjects)
* [Resource Consumption](#ResourceConsumption)
    * [Flash And Static Memory](#FlashAndStaticMemory)
    * [CPU Cycles](#CpuCycles)
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

The "Fast" version (`SimpleTmiFastInterface.h`)
depend on one of the digitalWriteFast libraries, for example:

* https://github.com/watterott/Arduino-Libs/tree/master/digitalWriteFast
* https://github.com/NicksonYap/digitalWriteFast

<a name="Documentation"></a>
## Documentation

* this `README.md` file.
* [Doxygen docs](https://bxparks.github.io/AceTMI/html)
    * On Github pages.
* Examples:
    * https://github.com/bxparks/AceSegment/tree/develop/examples/Tm1637Demo
    * https://github.com/bxparks/AceSegment/tree/develop/examples/Tm1637DualDemo

<a name="Usage"></a>
## Usage

<a name="HeaderAndNamespace"></a>
### Include Header and Namespace

In many cases, only a single header file `AceTMI.h` is required to use this
library. To prevent name clashes with other libraries that the calling code may
use, all classes are defined in the `ace_tmi` namespace. To use the code without
prepending the `ace_tmi::` prefix, use the `using` directive:

```C++
#include <Arduino.h>
#include <AceTMI.h>
using ace_tmi::SimpleTmiInterface;
```

The "Fast" versions are not included automatically by `AceTMI.h` because they
work only on AVR processors and they depend on a `<digitalWriteFast.h>`
library. To use the "Fast" versions, use something like the following:'

```C++
#include <Arduino.h>
#include <AceTMI.h>

#if defined(ARDUINO_ARCH_AVR)
  #include <digitalWriteFast.h>
  #include <ace_tmi/SimpleTmiFastInterface.h>
  using ace_tmi::SimpleTmiFastInterface;
#endif
```

<a name="UnifiedInterface"></a>
### Unified Interface

The classes in this library provide the following unified interface for handling
communication with the TM1637 chip. Downstream classes can code against this
unified interface using C++ templates so that different implementations can be
selected at compile-time.

```C++
class XxxInterface {
  public:
    void begin() const;
    void end() const;

    void startCondition() const;
    void stopCondition() const;
    uint8_t write(uint8_t data) const;
};
```

Notice that the classes in this library do *not* inherit from a common interface
with virtual functions. This saves several hundred bytes of flash memory on
8-bit AVR processors by avoiding the dynamic dispatch, and often allows the
compiler to optimize away the overhead of calling the methods in this library so
that the function call is made directly to the underlying implementation. The
reduction of flash memory consumption is especially large for classes that use
the digitalWriteFast libraries which use compile-time constants for pin numbers.
The disadvantage is that this library is harder to use because these classes
require the downstream classes to be implemented using C++ templates.

<a name="SimpleTmiInterface"></a>
### SimpleTmiInterface

The `SimpleTmiInterface` can be used like this to communicate with a TM1637
controller chip. It looks like this:

```C++
class SimpleTmiInterface {
  public:
    explicit SimpleTmiInterface(
        uint8_t dioPin,
        uint8_t clkPin,
        uint8_t delayMicros
    );

    void begin() const;
    void end() const;

    void startCondition() const;
    void stopCondition() const;
    uint8_t write(uint8_t data) const;
};
```

It is configured and used by the calling code `MyClass` like this:

```C++
#include <Arduino.h>
#include <AceTMI.h>
using ace_tmi::SimpleTmiInterface;

template <typename T_TMII>
class MyClass {
  public:
    explicit MyClass(T_TMII& tmiInterface)
        : mTmiInterface(tmiInterface)
    {...}

    void sendData() {
      // Set addressing mode.
      mTmiInterface.startCondition();
      mTmiInterface.write(addressMode);
      mTmiInterface.stopCondition();

      // Send data bytes.
      mTmiInterface.startCondition();
      mTmiInterface.write(otherCommand);
      [...]
      mTmiInterface.stopCondition();

      // Set brightness.
      mTmiInterface.startCondition();
      mTmiInterface.write(brightness);
      mTmiInterface.stopCondition();
    }

  private:
    T_TMII mTmiInterface; // copied by value
};

const uint8_t CLK_PIN = 8;
const uint8_t DIO_PIN = 9;
const uint8_t DELAY_MICROS = 100;

using TmiInterface = SimpleTmiInterface;
TmiInterface tmiInterface(DIO_PIN, CLK_PIN, DELAY_MICROS);
MyClass<TmiInterface> myClass(tmiInterface);

void setup() {
  tmiInterface.begin();
  ...
}
```

The `using` statement is the C++11 version of a `typedef` that defines
`TmiInterface`. It is not strictly necessary here, but it allows the same
pattern to be used for the more complicated examples below.

The `T_TMII` template parameter contains a `T_` prefix to avoid name collisions
with too many `#define` macros defined in the global namespace on Arduino
platforms. The double `II` contains 2 `Interface`, the first referring to the
TM1637 protocol, and the second referring to classes in this library.

<a name="SimpleTmiFastInterface"></a>
### SimpleTmiFastInterface

The `SimpleTmiFastInterface` is identical to `SimpleTmiInterface` except that it
uses `digitalWriteFast()`. It looks like this:

```C++
template <
    uint8_t T_DIO_PIN,
    uint8_t T_CLK_PIN,
    uint8_t T_DELAY_MICROS
>
class SimpleTmiFastInterface {
  public:
    explicit SimpleTmiFastInterface() = default;

    void begin() const;
    void end() const;

    void startCondition() const;
    void stopCondition() const;
    uint8_t write(uint8_t data) const;
};
```

It is configured and used by the calling code `MyClass` like this:

```C++
#include <Arduino.h>
#include <AceTMI.h>
#if defined(ARDUINO_ARCH_AVR)
  #include <digitalWriteFast.h>
  #include <ace_tmi/SimpleTmiFastInterface.h>
  using ace_tmi::SimpleTmiInterface;
#endif

const uint8_t CLK_PIN = 8;
const uint8_t DIO_PIN = 9;
const uint8_t DELAY_MICROS = 100;

template <typename T_TMII>
class MyClass {
  // Exactly same as above.
};

using TmiInterface = SimpleTmiFastInterface<DIO_PIN, CLK_PIN, DELAY_MICROS>;
TmiInterface tmiInterface;
MyClass<TmiInterface> myClass(tmiInterface);

void setup() {
  tmiInterface.begin();
  ...
}
```

<a name="StoringInterfaceObjects"></a>
### Storing Interface Objects

In the above examples, the `MyClass` object holds the `T_TMII` interface object
**by value**. In other words, the interface object is copied into the `MyClass`
object. This is efficient because interface objects are very small in size, and
copying them by-value avoids an extra level of indirection when they are used
inside the `MyClass` object.

The alternative is to save the `T_TMII` object **by reference** like this:

```C++
template <typename T_TMII>
class MyClass {
  public:
    explicit MyClass(T_TMII& tmiInterface)
        : mTmiInterface(tmiInterface)
    {...}

    [...]

  private:
    T_TMII& mTmiInterface; // copied by reference
};
```

The internal size of the `SimpleTmiInterface` object is just 3 bytes, and the size
of the `SimpleTmiFastInterface` is even smaller at 0 bytes, so we do not save much
memory by storing these by reference. But storing the `mTmiInterface` as a
reference causes an unnecessary extra layer of indirection every time the
`mTmiInterface` object is called. In almost every case, I recommend storing the
`XxxInterface` object **by value** into the `MyClass` object.

<a name="ResourceConsumption"></a>
## Resource Consumption

<a name="FlashAndStaticMemory"></a>
### Flash And Static Memory

The Memory benchmark numbers can be seen in
[examples/MemoryBenchmark](examples/MemoryBenchmark). Here are 2 samples:

**Arduino Nano**

```
+--------------------------------------------------------------+
| functionality                   |  flash/  ram |       delta |
|---------------------------------+--------------+-------------|
| baseline                        |    456/   11 |     0/    0 |
|---------------------------------+--------------+-------------|
| SimpleTmiInterface              |   1200/   14 |   744/    3 |
| SimpleTmiFastInterface          |    638/   11 |   182/    0 |
+--------------------------------------------------------------+
```

**ESP8266**

```
+--------------------------------------------------------------+
| functionality                   |  flash/  ram |       delta |
|---------------------------------+--------------+-------------|
| baseline                        | 256700/26784 |     0/    0 |
|---------------------------------+--------------+-------------|
| SimpleTmiInterface              | 257588/26796 |   888/   12 |
+--------------------------------------------------------------+
```

<a name="CpuCycles"></a>
### CPU Cycles

The CPU benchmark numbers can be seen in
[examples/AutoBenchmark](examples/AutoBenchmark). Here are 2 samples:

**Arduino Nano**

```
+-----------------------------------------+-------------------+----------+
| Functionality                           |   min/  avg/  max | eff kbps |
|-----------------------------------------+-------------------+----------|
| SimpleTmiInterface,1us                  |   752/  781/  836 |     41.0 |
| SimpleTmiFastInterface,1us              |    76/   83/   84 |    385.5 |
+-----------------------------------------+-------------------+----------+
```

**ESP8266**

```
+-----------------------------------------+-------------------+----------+
| Functionality                           |   min/  avg/  max | eff kbps |
|-----------------------------------------+-------------------+----------|
| SimpleTmiInterface,1us                  |   392/  395/  433 |     81.0 |
+-----------------------------------------+-------------------+----------+
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
