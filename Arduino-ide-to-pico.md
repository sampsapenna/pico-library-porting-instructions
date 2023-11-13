# Porting from Arduino IDE to Pico SDK
Arduino IDE libraries have been originally developed to be quite compatible
between microcontrollers, and especially with 8-bit MCU support in mind.

Additionally some hardware features are poorly implemented when a more
capable platform is considered.

As such, it makes sense to port the needed libraries to Pico SDK in case
they're not available – the conversion can be one with relative ease.

## Wire.h
Arduino IDE's i2c library is somewhat more low-level than the one provide
by Pico SDK (`hardware/i2c.h`). With Pico, the hardware bus controller takes
care of the initiation handshakes etc., which means the bus commands typically
used with `wire.h` aren't necessary.

Additionally, due to the hardware implementation with the `RP2040` chip used
in Pico, it's not necessary to send contents one character at a time. The
write and read functions used in `hardware/i2c.h` can deal with buffering
multiple bytes at once.

## Port IO and digitalWrite/digitalRead
Pico deals with IO a bit differently. While for most situations directly
mapping `digitalWrite` and `digitalRead` to their respective functions
in Pico SDK is more than adequate, port IO can sometimes prove to be
a challenge.

RP2040 has a considerably high clock speed compared to the usual AVRs,
which means it should be fine to skip port IO altogether, opting instead
for multiple sequential calls to `gpio_put`/`gpio_get`.
Moving over to port IO on Pico should be doable as well, though, in case
there's a significance to being able to write multiple bits at once
with a single instruction (e.g. emulating a byte-parallel bus)

## ADC
RP2040 has a 12-bit ADC operating at 0-3.3 volts, compared to most
Arduino boards, which use a 10-bit ADC operating at 0-5 volts (built in
to the AVR chips used).

This leads to some necessary changes in the code, as the measured range
changes from 0-1023 to 0-4095 and the measure voltage to the aforementioned
ranges. Any math converting the ADC input to meaningful values needs
to take this into account, so if the library uses the ADC for anything,
the conversion math will usually be off.

Additionally it will be useful to check that whatever sensor is being
used doesn't exceed the maximum voltage of 3.3 V. If that's the case,
a simple voltage divider can be added to drop the voltage to suitable
levels.

## Timing
Any libraries using timing based on the amount of instructions has to be
altered, as the execution time varies between the architectures. For most
cases the speed difference is high enough that the timing can be achieved
via sleeping instead of buffering calls with `noop`. Your mileage may
vary, however.
