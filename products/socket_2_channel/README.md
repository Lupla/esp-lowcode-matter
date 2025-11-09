# Socket | 8 Channel

## Description

An eight-channel smart socket featuring independent relay control and a shared WS2812 RGB indicator LED:

* **Eight Relay Outputs**: Each channel is exposed as a dedicated Matter endpoint of type `On/Off Plug`
* **Matter-Only Control**: Channel state changes are driven via Matter feature updates—no local buttons required
* **Unified Status Indication**: Single WS2812 RGB LED displays aggregated socket status and system events
* **Matter Data Model Specification**:
  * **Device Type** : `On/Off Plug`

## Hardware Configuration

<img src="../../docs/images/product_socket_2_channel.png" alt="Socket 2 Channel" width="500"/>

The following hardware components are used for this product:

* **Controller**: [nanoESP32-C6](https://github.com/wuxx/nanoESP32-C6) (ESP32-C6 module with USB-C power and exposed GPIOs)
* **Power Relays**: [8 Channel Relay Module, DC 5 V, opto-isolated, high/low trigger](https://amzn.eu/d/6Cs208m)
* **Indicator**: On-board WS2812 RGB LED

### Pin Assignment

| Peripheral      | GPIO Pin | Function                         |
|-----------------|----------|----------------------------------|
| Relay 1 Control | GPIO0    | Channel 1 power switching        |
| Relay 2 Control | GPIO1    | Channel 2 power switching        |
| Relay 3 Control | GPIO2    | Channel 3 power switching        |
| Relay 4 Control | GPIO3    | Channel 4 power switching        |
| Relay 5 Control | GPIO4    | Channel 5 power switching        |
| Relay 6 Control | GPIO5    | Channel 6 power switching        |
| Relay 7 Control | GPIO6    | Channel 7 power switching        |
| Relay 8 Control | GPIO7   | Channel 8 power switching        |
| RGB LED         | GPIO8    | Unified status indication        |

> **Note**: GPIO assignments can be customized by editing the `relay_gpio_pins[]` array in **app_driver.cpp** and `INDICATOR_GPIO_NUM` define if needed.

## Understanding Code

### Initialization Sequence

The `app_driver_init()` function performs the following:

* Initializes all relay GPIOs as outputs
* Initializes the WS2812 RGB LED for combined status indication
* Leaves all channels off until Matter feature updates arrive

### Channel Control Flow

* `app_driver_set_socket_state` validates the endpoint, switches the corresponding relay, and updates the shared indicator whenever any channel is on
* `feature_update_from_system` (in `app_main.cpp`) forwards On/Off feature updates from any of the eight endpoints to the driver
* The driver no longer registers button callbacks; only Matter-sourced updates can toggle relays

### Core Functions

* **Power Control**:
  * `app_driver_set_socket_state` manages relay states individually while providing unified LED feedback
  * Feature updates from the system contain the endpoint ID, allowing the driver to map directly to the proper relay

* **Visual Indicators**:
  * `LOW_CODE_EVENT_SETUP_MODE_START`: starts blinking effect, to indicate setup mode activation (2000ms interval)
  * `LOW_CODE_EVENT_SETUP_MODE_END`: stops blinking effect, to indicate setup mode has ended.
  * `LOW_CODE_EVENT_READY`: displays full brightness white light to indicate device is ready

### Multi-Endpoint Implementation

* Endpoint mapping:
  * Endpoints 1–8 map linearly to the eight relay GPIOs defined in `relay_gpio_pins[]`
* State tracking:
  * `socket_states[]` array maintains individual relay states with `SOCKET_ENDPOINT_COUNT = 8`
  * Indicator power reflects the OR of all channel states

### Extending Functionality

To adjust the number of relay channels:

* **Matter Data Model Extension**:
  * Add/remove On/Off Plug device endpoints in the Matter data model (`data_model.zap`).
  * Run `Upload Configuration` to regenerate and push the updated data model to the device.

* **Update Relay Mapping**:
  * Modify the `relay_gpio_pins[]` array in `app_driver.cpp` to reflect the new hardware layout.
  * Ensure `SOCKET_ENDPOINT_COUNT` matches the total number of exposed endpoints.

## Related Documentation

* [Socket | 1 Channel](../socket/README.md)
* [Programmer's Model](../../docs/programmer_model.md)
* [Components](../../components/README.md)
* [Drivers](../../drivers/README.md)
* [Products](../README.md)
