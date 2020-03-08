# MicroPython ESP8266/ESP32 Driver for TTP229-BSF 16-key Capacitive Keypad in Serial Interface Mode

This MicroPython driver is for TTP229-BSF capacitive keypad on ESP8266/ESP3. Tested on both ESP8266 (WeMos D1 mini) and ESP32 (BPI:bit) with v1.11 firmware.

The serial interface of TTP229-BSF <b>is not I2C</b> but it allows you to control this keypad via only 2 wires. Not to be confused with TTP229-LSF, which is a true I2C device.

## Hardware Configuration

TTP229-BSF's serial interface supports either 8 or 16 key mode (see [datasheet](https://www.sunrom.com/get/611100)), as well as single/multiple key mode. For some of the settings hardware configuration is required:

![11-1035x1200](https://user-images.githubusercontent.com/44191076/69064016-6ec49c00-0a58-11ea-9b46-c10f4f1a9cdf.jpg)

**Connect these places with either simple wires or soldering. Do not use resistors.**

* TP2 not connected: 8 keys
* TP2 connected (pulled low): 16 keys

* TP3 and TP4 not connected: single key mode
* TP3 and TP4 both connected (pulled low): multiple key mode

Without any modification the TTP229-BSF is in 8-key/single mode. Connect the TPx pins by wires or soldering to enable these options.

## Wiring

* VCC: 3.3V
* GND: GND
* SCL: scl pin (output)
* SDO (not SDA): sdo pin (input)

## Example

```python
from machine import Pin
from TTP229_BSF import Keypad

scl_pin = Pin(5, Pin.OUT)
sdo_pin = Pin(4, Pin.IN)

keypad = Keypad(scl=scl_pin, sdo=sdo_pin, inputs=16, multi=False)

while True:
    print(keypad.read())
    # return a index number like 15 in single mode
    # return a list like (0, 2, 11, 15) in multiple mode
```

<b>input</b> parameter:

* **input=8** (default): 8 key mode
* **input=16**: 16 key mode

<b>multi</b> parameter:

* **multi=False** (default): single mode
* **multi=True**: multiple mode

In single mode keypad.read() will return the index of the pressed key (0~15). Return -1 when no key is pressed.

In multiple mode keypad.read() will return a list containing all the indexes of pressed keys. Return an empty list when no key is pressed.

If the TTP229-BSF is configured to multiple mode, read it in single mode will return the lowest index of all pressed keys. Read in multiple mode for a keypad configured in single mode, you'll get a list containing only one key index if any key is pressed.

There's also a <b>raw</b> parameter, default False. When set as True, keypad.read() will return the raw 8 or 16 key list indicating all keys' status (from key 0 to 15; value 1 = not pressed, 0 = pressed).

```python
keypad = Keypad(scl=scl_pin, sdo=sdo_pin, inputs=16, multi=False, raw=True)
# return a list like (0, 1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 0, 1, 1, 1, 0)

while True:
    print(keypad.read())
```

If the keypad is configured to multiple mode and you read it in single/raw mode, you'll still get a list which shows multiple key pressing results.
