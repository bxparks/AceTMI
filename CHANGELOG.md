# Changelog

* Unreleased
    * Rename `sendByte()` to `write()` for consistency with the API of `AceWire`
      and `TwoWire`, since the TM1637 protocol is very similar to I2C.
        * Also avoids confusion with the `send8()` and `send16()` methods of
          `AceSPI` which include the `beginTransaction()` and
          `endTransaction()`.
        * The `write()` methods do *not* include the start and stop conditions.
* 0.2 (2021-07-30)
    * Add `examples/MemoryBenchmark` and `examples/AutoBenchmark` to calculate
      memory consumption and CPU speed.
    * Invert return value of `writeByte()`, returning 1 on success and 0 on
      failure. Matches the return values of `AceWire` implementations.
    * Add GitHub workflow validation of `examples/*`.
    * Rename `SoftTmi*Interface` to `SimpleTmi*Interface`.
* 0.1 (2021-06-25)
    * First GitHub release.
* 2021-06-24
    * Initial extraction from
      [AceSegment](https://github.com/bxparks/AceSegment) library.
