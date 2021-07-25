# Changelog

* Unreleased
    * Add `examples/MemoryBenchmark` and `examples/AutoBenchmark` to calculate
      memory consumption and CPU speed.
    * Invert return value of `writeByte()`, returning 1 on success and 0 on
      failure. Matches the return values of `AceWire` implementations.
    * Add GitHub workflow validation of `examples/*`.
* 0.1 (2021-06-25)
    * First GitHub release.
* 2021-06-24
    * Initial extraction from
      [AceSegment](https://github.com/bxparks/AceSegment) library.
