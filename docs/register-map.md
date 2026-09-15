# AFERIY P180 confirmed register map

This document contains the mappings represented as confirmed in the
tested production configuration.

## Live / status registers --- function `0x04`

  -----------------------------------------------------------------------
  Register                Meaning                 Conversion / notes
  ----------------------- ----------------------- -----------------------
  `R02`                   AC charging power       W

  `R03`                   DC / solar input power  W

  `R08`                   AC input voltage        raw / 10 = V

  `R09`                   AC input frequency      raw / 100 = Hz

  `R10`                   Actual AC output        raw / 10 = V
                          voltage                 

  `R11`                   Configured AC output    raw / 10 = Hz; 500 →
                          frequency               50.0 Hz, 600 → 60.0 Hz

  `R12`                   AC output power         W

  `R31`                   Battery SOC             \%

  `R53`                   AC / charging status    group `0x28` = AC input
                                                  present; bit `0x40` =
                                                  active charging

  `R71`                   Time to full charge     minutes

  `R72`                   Remaining runtime       minutes

  `R75`                   Output status           bit `0x04` = DC; `0x08`
                                                  = USB; `0x10` = AC

  `R78`                   DC + USB output power   W
  -----------------------------------------------------------------------

## Settings / holding registers --- function `0x03`

  Register   Meaning                              Conversion / notes
  ---------- ------------------------------------ ---------------------------------------
  `H23`      Maximum DC / XT60 charging current   A
  `H24`      AC no-load auto-off timeout          minutes; 0 = Never
  `H25`      Screen-off timeout                   stored in seconds; exposed as minutes
  `H26`      Lower battery discharge threshold    raw / 10 = %
  `H27`      Upper battery charge threshold       raw / 10 = %
  `H28`      Full-device auto power-off timeout   minutes
  `H29`      USB no-load auto-off timeout         minutes; 0 = Never
  `H30`      DC no-load auto-off timeout          minutes; 0 = Never

## Derived values

``` text
AC Grid Input = R02 + R12   when AC input is present
Total Input   = AC Grid Input + R03
Total Output  = R12 + R78
```

The parser treats AC as present for the power calculation when
`R08 / 10 > 100 V`, filtering residual/induced voltage when the source
is off.

## BLE packet parsing

The tested P180 `C305` packet contains 80 16-bit registers beginning at
byte 6.

-   Function `0x04`: live/status input registers
-   Function `0x03`: holding/settings registers
