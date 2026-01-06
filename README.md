# Arduino Core for SAMD21 and SAMD51 CPU

This repository contains the source code and configuration files of the Arduino Core
for Atmel's SAMD21 and SAMD51 processor (used on the Arduino/Genuino Zero, MKR1000 and MKRZero boards).

In particular, this adds support for the Seeed SAMD Boards such as the Seeeduino XIAO

## Custom USB Serial Number Support

This fork adds the ability to set a custom USB serial number for SAMD devices instead of using the hardware-based serial number derived from the chip's unique ID.

### Purpose

By default, SAMD microcontrollers use a serial number generated from hardware registers, which is unique for each chip but not customizable. This modification allows applications to:

- Set a user-defined USB serial number programmatically
- Maintain a consistent serial number across multiple devices for testing or deployment
- Identify devices by application-specific identifiers rather than hardware IDs

### Usage

This fork supports both **PlatformIO** and **Arduino IDE** (tested on versions 2.3.6 and 2.3.7).

#### PlatformIO
To enable custom serial number support, define `USB_CUSTOM_SERIAL` in your build flags:
```ini
# In platformio.ini
build_flags = -DUSB_CUSTOM_SERIAL
```

#### Arduino IDE

To enable custom serial number support in Arduino IDE:

1. Install this board package
- Seeed ArduinoCore-samd needs to be added as well because of tools dependencies: openocd, CMSIS, CMSIS-Atmel, arduinoOTA: https://files.seeedstudio.com/arduino/package_seeeduino_boards_index.json
- Add this package: https://github.com/mszwaj/ArduinoCore-samd-custom-sn/releases/download/1.8.5-custom-sn.1-rc1/package_mszwaj_seeeduino_xiao_custom_sn.json
- Install "Seeed SAMD Boards" and "SAMD Boards (custom SN)"
2. Select your board: **Tools → Board → SAMD Boards (custom SN) → Seeeduino XIAO (SN)**
3. Enable custom serial: **Tools → Custom Serial → Enabled**
- this setting adds USB_CUSTOM_SERIAL build flag

When **Custom Serial** is set to **Disabled**, the device will use the default hardware-based serial number.


Set your custom serial number in code using:
```cpp
char g_usbSerialNumber[33];

strcpy(g_usbSerialNumber, "MY-CUSTOM-SERIAL-001");
    
    // Your code...
```

**Note:** The custom serial number must be set before USB initialization. If `g_usbSerialNumber` is empty or `USB_CUSTOM_SERIAL` is not defined, the device will fall back to the hardware-based serial number.

#### Example
Example of usage with reading from flash before USB renumeration - keep it before setup():
```cpp
class USBSerialInitializer {
public:
  USBSerialInitializer() {
    myFlashStorage.read(data);
    if (data.magicNumber == EEPROM_MAGIC_NUMBER) {
      strcpy(g_usbSerialNumber, data.sn);
    } else {
      uint32_t* uid = (uint32_t*)0x0080A00C;
      sprintf(g_usbSerialNumber, "%08lX%08lX%08lX%08lX", uid[0], uid[1], uid[2], uid[3]);
    }
  }
};
```

### Implementation Details

- Custom serial numbers are limited to 32 characters (plus null terminator)
- The feature is opt-in via the `USB_CUSTOM_SERIAL` compile flag
- Without the flag, behavior is identical to the original Arduino Core
- Hardware serial number is used as fallback if custom serial is not set
- In Arduino IDE, the **Tools → Custom Serial** option (provided by `boards.txt`) automatically adds the `USB_CUSTOM_SERIAL` build flag when enabled

## Bugs or Issues


If you find a bug you can submit an issue here on github:

https://github.com/Seeed-Studio/ArduinoCore-samd

or if it is an issue with the upstream:

https://github.com/arduino/ArduinoCore-samd/issues

Before posting a new issue, please check if the same problem has been already reported by someone else
to avoid duplicates.

## License and credits

This core has been developed by Arduino LLC in collaboration with Atmel.

```
  Copyright (c) 2015 Arduino LLC.  All right reserved.

  This library is free software; you can redistribute it and/or
  modify it under the terms of the GNU Lesser General Public
  License as published by the Free Software Foundation; either
  version 2.1 of the License, or (at your option) any later version.

  This library is distributed in the hope that it will be useful,
  but WITHOUT ANY WARRANTY; without even the implied warranty of
  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
  See the GNU Lesser General Public License for more details.

  You should have received a copy of the GNU Lesser General Public
  License along with this library; if not, write to the Free Software
  Foundation, Inc., 51 Franklin St, Fifth Floor, Boston, MA  02110-1301  USA
```
