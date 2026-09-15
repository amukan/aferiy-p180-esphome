# AFERIY P180 monitoring with ESPHome and Home Assistant

Read-only Bluetooth monitoring for the **AFERIY P180** portable power
station using an ESP32, ESPHome, and Home Assistant.

This configuration was built by reverse engineering the P180 `C305` BLE
characteristic and has been tested in real use for several days with
stable operation.

## What it does

The ESP32 keeps working as a Home Assistant Bluetooth Proxy while using
a separate BLE connection for the AFERIY P180. The configuration reads
live/status registers (`0x04`) every 5 seconds and settings/holding
registers (`0x03`) every 30 seconds.

**Monitoring only:** this project does not write settings or commands to
the power station.

### Exposed data

-   Battery state of charge
-   AC charging power
-   AC grid input power
-   DC / solar input power
-   Total input power
-   AC output power
-   DC + USB output power
-   Total output power
-   AC input voltage and frequency
-   AC output voltage and configured frequency
-   Time to full charge
-   Remaining runtime
-   AC input present / charging state
-   AC, DC and USB output enabled states
-   Read-only power and timeout settings

See [docs/register-map.md](docs/register-map.md) for the confirmed
register map.

## Requirements

-   AFERIY P180
-   ESP32 DevKit V1 or compatible ESP32
-   ESPHome
-   Home Assistant
-   [`ylianst/esp-fbot`](https://github.com/ylianst/esp-fbot) external
    ESPHome component

The tested configuration uses ESP-IDF and four BLE connections: three
Bluetooth Proxy slots plus one dedicated AFERIY connection.

## Installation

1.  Copy `aferiy-p180.yaml` to your ESPHome configuration directory.
2.  Find the BLE MAC address of your own AFERIY P180.
3.  Replace the placeholder MAC address in the `ble_client` section.
4.  Make sure your ESPHome `secrets.yaml` contains `wifi_ssid`,
    `wifi_password`, `api_encryption_key`, `ota_password`, and
    `fallback_ap_password`.
5.  Validate, compile and install the configuration with ESPHome.
6.  Add the ESPHome device to Home Assistant if it is not discovered
    automatically.

After the BLE connection is established, Home Assistant exposes the
AFERIY P180 as a separate logical device.

## How the values are calculated

``` text
AC Grid Input = R02 + R12   (when AC input is present)
Total Input   = AC Grid Input + R03
Total Output  = R12 + R78
```

AC input is treated as present for the power calculation only when
measured input voltage is above 100 V. This filters residual/induced
voltage when the AC source is off.

## Validation

The mapping was developed by comparing BLE register changes with the
P180's actual operating state and displayed values under different
conditions, including AC input, battery charging, AC loads, AC bypass
operation, DC load, combined AC + DC load, output enable states, battery
SOC, runtime/charge-time estimates, and read-only settings.

The final configuration was then left running in normal Home Assistant
use for several days and remained stable.

## Screenshots

The screenshots show the AFERIY P180 exposed as a separate ESPHome
device in Home Assistant with live and read-only entities.

![AFERIY P180 in Home Assistant --- main
entities](images/home-assistant-device-top.png)

![AFERIY P180 in Home Assistant --- additional
entities](images/home-assistant-device-bottom.png)

## Important notes

-   This is an independent community project and is not affiliated with
    AFERIY.
-   The register map is based on observed behavior of the tested P180.
-   Only registers that were sufficiently identified are exposed by the
    production YAML.
-   Unknown and diagnostic registers used during reverse engineering
    were intentionally removed.
-   The project currently provides monitoring only; it does not send
    control or settings writes to the P180.

## Register map

See [docs/register-map.md](docs/register-map.md).

## Contributing

Reports from other AFERIY P180 owners are welcome, especially
confirmations of register behavior on other firmware revisions.

## License

This project is released under the [MIT License](LICENSE).
