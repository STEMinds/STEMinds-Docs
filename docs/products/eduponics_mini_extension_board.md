---
title: Eduponics Mini - Extension board
description: The Eduponics mini extension board designed using ADS1115 for ADC support and MCP23017 for GPIO control with some advance functionalities such as interrupt on change and precise analog data reading, the board allows to connect extra 4 analog data sensors and 4 digital output devices with 12V support.
---

# Eduponics Mini Extension board

The Eduponics mini extension board designed using ADS1115 for ADC support and MCP23017 for GPIO control with some advance functionalities such as interrupt on change and precise analog data reading, the board allows to connect extra 4 analog data sensors and 4 digital output devices with 12V support.

While the Eduponics Mini ESP32 board allows to connect only one pump and one soil moisture sensor, we've decided to develop a custom HAT (Hardware Attached on Top) to enhance the Eduponics Mini functionalities by adding extra 4 soil moisture sensors and extra 4 pumps.

## Features

If we use simple words: the extension board allows us to control extra 4 plants (total of 5 plants) at the same time.
If we go with deeper explanation, the sensors aren't necessary needs to be soil moisture sensors or pumps, it can be any analog or digital sensors that needs to be controlled using the Eduponics Mini, giving you access to extra 4 relays and another 4 open IOs (input / output) and extra 4 analog sensors control.

This is really great addition not just to the Eduponics Mini ESP32 board but also to other microcontrollers such as Raspberry Pi that needs extra IO pins or analog reading which it's lack of.

Here is the detailed features that the Eduponics Mini extension board can offer you:

* ADS1115 ADC chipset, precise analog data reading
* MCP23017 IO chipset, allows to control extra 8 IO (input or output) devices
* External interrupt pins on data change for MCP23017
* External interrupt alert on analog data changes for ADS1115
* 4 Relay modules with XH2.54 connectors, 5V-24V support
* I2C interface - only 2 pins required
* Arduino, Raspberry Pi & ESP32 and other controllers compatible

!!! Danger "12V-24V are not the same as 5V input"
      The Eduponics mini has 2 pins one is used as 12V pin goes from the DC adapter to the relay directly allows the relay to control pumps that rated higher than 5V, the other pin is 5V which gives power to the ESP32 and other ICs.

      DO NOT connect voltage higher than 5V to the pin rated as 12V-24V as this WILL damage your Eduponics extension board and/or your Eduponics mini.
      The 12V-24V are designed to be isolated and wired through the relays ONLY and are meant to control external sensors and modules that require such voltage.

## ADS1115 16-Bit ADC

The Eduponics Mini Built-in ADC works pretty well but for the extension board we've decided to use ADS1115 to increase the precision, the ADS1115 provides 16-bit precision at 860 samples/second over the I2C protocol, the ADC includes a programmable gain amplifier, up to x16, to help boost up smaller single/differential signals to the full range

A micro-computer like Raspberry Pi for example doesn't have built-in ADC (analog to digital converter) as well as the pins might be occupied or limited, by using I2C we only need 2 pins and using the ADS1115 we can add analog capabilities to boards that lack of it.

### Features

Here are some of the specific features of the ADS1115:

* Wide supply range: 2.0V to 5.5V
* Low current consumption: Continuous Mode: Only 150µA Single-Shot Mode: Auto Shut-Down
* Internal oscillator
* Internal PGA
* I2C Interface

For the complete information, check the ADS1115 data sheet: [ADS1115 datasheet](https://www.ti.com/lit/ds/symlink/ads1115.pdf)

### ADS1X15 library

Below is the MicroPython library to control and configure the ADS1115 functionalities using the Eduponics Mini board, make sure to save the following code into a file named *ads1x15.py* we will use another example file to use this library to receive the analog data from the sensors:

=== "MicroPython"
    ``` python linenums="1"
    # The MIT License (MIT)
    #
    # Copyright (c) 2016 Radomir Dopieralski (@deshipu),
    #               2017 Robert Hammelrath (@robert-hh)
    #
    # Permission is hereby granted, free of charge, to any person obtaining a copy
    # of this software and associated documentation files (the "Software"), to deal
    # in the Software without restriction, including without limitation the rights
    # to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    # copies of the Software, and to permit persons to whom the Software is
    # furnished to do so, subject to the following conditions:
    #
    # The above copyright notice and this permission notice shall be included in
    # all copies or substantial portions of the Software.
    #
    # THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    # IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    # FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    # AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    # LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    # OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    # THE SOFTWARE.
    #
    import utime as time

    _REGISTER_CONVERT = const(0x00)
    _REGISTER_CONFIG = const(0x01)
    _REGISTER_LOWTHRESH = const(0x02)
    _REGISTER_HITHRESH = const(0x03)

    _OS_SINGLE = const(0x8000)  # Write: Set to start a single-conversion
    _OS_NOTBUSY = const(0x8000)  # Read: Bit=1 when no conversion is in progress

    _MUX_DIFF_0_1 = const(0x0000)  # Differential P  =  AIN0, N  =  AIN1 (default)
    _MUX_DIFF_0_3 = const(0x1000)  # Differential P  =  AIN0, N  =  AIN3
    _MUX_DIFF_1_3 = const(0x2000)  # Differential P  =  AIN1, N  =  AIN3
    _MUX_DIFF_2_3 = const(0x3000)  # Differential P  =  AIN2, N  =  AIN3
    _MUX_SINGLE_0 = const(0x4000)  # Single-ended AIN0
    _MUX_SINGLE_1 = const(0x5000)  # Single-ended AIN1
    _MUX_SINGLE_2 = const(0x6000)  # Single-ended AIN2
    _MUX_SINGLE_3 = const(0x7000)  # Single-ended AIN3

    _PGA_6_144V = const(0x0000)  # +/-6.144V range  =  Gain 2/3
    _PGA_4_096V = const(0x0200)  # +/-4.096V range  =  Gain 1
    _PGA_2_048V = const(0x0400)  # +/-2.048V range  =  Gain 2 (default)
    _PGA_1_024V = const(0x0600)  # +/-1.024V range  =  Gain 4
    _PGA_0_512V = const(0x0800)  # +/-0.512V range  =  Gain 8
    _PGA_0_256V = const(0x0A00)  # +/-0.256V range  =  Gain 16

    _MODE_CONTIN = const(0x0000)  # Continuous conversion mode
    _MODE_SINGLE = const(0x0100)  # Power-down single-shot mode (default)

    _DR_128SPS = const(0x0000)   # 128 /8 samples per second
    _DR_250SPS = const(0x0020)   # 250 /16 samples per second
    _DR_490SPS = const(0x0040)   # 490 /32 samples per second
    _DR_920SPS = const(0x0060)   # 920 /64 samples per second
    _DR_1600SPS = const(0x0080)  # 1600/128 samples per second (default)
    _DR_2400SPS = const(0x00A0)  # 2400/250 samples per second
    _DR_3300SPS = const(0x00C0)  # 3300/475 samples per second
    _DR_860SPS = const(0x00E0)  # -   /860 samples per Second

    _CMODE_TRAD = const(0x0000)  # Traditional comparator with hysteresis (default)

    _CPOL_ACTVLOW = const(0x0000)  # ALERT/RDY pin is low when active (default)

    _CLAT_NONLAT = const(0x0000)  # Non-latching comparator (default)
    _CLAT_LATCH = const(0x0004)  # Latching comparator

    _CQUE_1CONV = const(0x0000)  # Assert ALERT/RDY after one conversions
    # Disable the comparator and put ALERT/RDY in high state (default)
    _CQUE_NONE = const(0x0003)

    _GAINS = (
        _PGA_6_144V,  # 2/3x
        _PGA_4_096V,  # 1x
        _PGA_2_048V,  # 2x
        _PGA_1_024V,  # 4x
        _PGA_0_512V,  # 8x
        _PGA_0_256V   # 16x
    )

    _GAINS_V = (
        6.144,  # 2/3x
        4.096,  # 1x
        2.048,  # 2x
        1.024,  # 4x
        0.512,  # 8x
        0.256  # 16x
    )

    _CHANNELS = {
        (0, None): _MUX_SINGLE_0,
        (1, None): _MUX_SINGLE_1,
        (2, None): _MUX_SINGLE_2,
        (3, None): _MUX_SINGLE_3,
        (0, 1): _MUX_DIFF_0_1,
        (0, 3): _MUX_DIFF_0_3,
        (1, 3): _MUX_DIFF_1_3,
        (2, 3): _MUX_DIFF_2_3,
    }

    _RATES = (
        _DR_128SPS,   # 128/8 samples per second
        _DR_250SPS,   # 250/16 samples per second
        _DR_490SPS,   # 490/32 samples per second
        _DR_920SPS,   # 920/64 samples per second
        _DR_1600SPS,  # 1600/128 samples per second (default)
        _DR_2400SPS,  # 2400/250 samples per second
        _DR_3300SPS,  # 3300/475 samples per second
        _DR_860SPS    # - /860 samples per Second
    )


    class ADS1115:
        def __init__(self, i2c, address=0x48, gain=1):
            self.i2c = i2c
            self.address = address
            self.gain = gain
            self.temp2 = bytearray(2)

        def _write_register(self, register, value):
            self.temp2[0] = value >> 8
            self.temp2[1] = value & 0xff
            self.i2c.writeto_mem(self.address, register, self.temp2)

        def _read_register(self, register):
            self.i2c.readfrom_mem_into(self.address, register, self.temp2)
            return (self.temp2[0] << 8) | self.temp2[1]

        def raw_to_v(self, raw):
            v_p_b = _GAINS_V[self.gain] / 32767
            return raw * v_p_b

        def set_conv(self, rate=4, channel1=0, channel2=None):
            """Set mode for read_rev"""
            self.mode = (_CQUE_NONE | _CLAT_NONLAT |
                         _CPOL_ACTVLOW | _CMODE_TRAD | _RATES[rate] |
                         _MODE_SINGLE | _OS_SINGLE | _GAINS[self.gain] |
                         _CHANNELS[(channel1, channel2)])

        def read_raw(self, rate=4, channel1=0, channel2=None):
            """Read voltage between a channel and GND.
               Time depends on conversion rate."""
            self._write_register(_REGISTER_CONFIG, (_CQUE_NONE | _CLAT_NONLAT |
                                 _CPOL_ACTVLOW | _CMODE_TRAD | _RATES[rate] |
                                 _MODE_SINGLE | _OS_SINGLE | _GAINS[self.gain] |
                                 _CHANNELS[(channel1, channel2)]))
            while not self._read_register(_REGISTER_CONFIG) & _OS_NOTBUSY:
                time.sleep_ms(1)
            res = self._read_register(_REGISTER_CONVERT)
            return res if res < 32768 else res - 65536

        def read_rev(self):
            """Read voltage between a channel and GND. and then start
               the next conversion."""
            res = self._read_register(_REGISTER_CONVERT)
            self._write_register(_REGISTER_CONFIG, self.mode)
            return res if res < 32768 else res - 65536

        def alert_start(self, rate=4, channel1=0, channel2=None,
                        threshold_high=0x4000, threshold_low=0, latched=False) :
            """Start continuous measurement, set ALERT pin on threshold."""
            self._write_register(_REGISTER_LOWTHRESH, threshold_low)
            self._write_register(_REGISTER_HITHRESH, threshold_high)
            self._write_register(_REGISTER_CONFIG, _CQUE_1CONV |
                                 _CLAT_LATCH if latched else _CLAT_NONLAT |
                                 _CPOL_ACTVLOW | _CMODE_TRAD | _RATES[rate] |
                                 _MODE_CONTIN | _GAINS[self.gain] |
                                 _CHANNELS[(channel1, channel2)])

        def conversion_start(self, rate=4, channel1=0, channel2=None):
            """Start continuous measurement, trigger on ALERT/RDY pin."""
            self._write_register(_REGISTER_LOWTHRESH, 0)
            self._write_register(_REGISTER_HITHRESH, 0x8000)
            self._write_register(_REGISTER_CONFIG, _CQUE_1CONV | _CLAT_NONLAT |
                                 _CPOL_ACTVLOW | _CMODE_TRAD | _RATES[rate] |
                                 _MODE_CONTIN | _GAINS[self.gain] |
                                 _CHANNELS[(channel1, channel2)])

        def alert_read(self):
            """Get the last reading from the continuous measurement."""
            res = self._read_register(_REGISTER_CONVERT)
            return res if res < 32768 else res - 65536

        def read(self,pin):
            raw = self.read_raw(channel1=pin)
            voltage = self.raw_to_v(raw)
            return {"raw":raw,"voltage":voltage}
    ```

### ADS1X15 Example code

Now when we have the library ready, let's use it to to get the analog sensors data (both raw and voltage), voltage can be sometimes useful to create some other conversion techniques or use it for data collection purposes. We will configure I2C address on SCL pin 33 and SDA pin 32 as this is the pins the extension board connected to, then we will use the default 0x48 I2C address and set gain to 1 (default)

We will read the analog data from the sensors every second and print them into the terminal screen.

For the Arduino code, we need to download and install the following library: [ADS1115 Arduino Library](https://github.com/addicore/ADS1115)

=== "MicroPython"
    ``` python linenums="1"
    from ads1x15 import ADS1115
    from machine import I2C, Pin
    import time

    # setup I2C
    i2c = I2C(scl=Pin(33), sda=Pin(32))
    # setup adc
    address = 0x48
    gain = 1
    adc = ADS1115(i2c, address, gain)

    while True:
        print("Soil moisture 1: %s " % adc.read(0))
        print("Soil moisture 2: %s " % adc.read(1))
        print("Soil moisture 3: %s " % adc.read(2))
        print("Soil moisture 4: %s " % adc.read(3))
        time.sleep(1)
    ```
=== "Arduino"
    ``` C++ linenums="1"
    // I2C device class (I2Cdev) demonstration Arduino sketch for ADS1115 class
    // Example of reading two differential inputs of the ADS1115 and showing the value in mV
    // 2016-03-22 by Eadf (https://github.com/eadf)
    // Updates should (hopefully) always be available at https://github.com/jrowberg/i2cdevlib
    //
    // Changelog:
    //     2016-03-22 - initial release

    /* ============================================
    I2Cdev device library code is placed under the MIT license
    Copyright (c) 2011 Jeff Rowberg
    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.
    ===============================================
    Wiring the ADS1115 Module to an Arduino UNO
    ADS1115 -->  UNO
      VDD        5V
      GND        GND
      SCL        A5 (or SCL)
      SDA        A4 (or SDA)
      ALRT       2
    */

    #include "ADS1115.h"

    ADS1115 adc0(0x48);

    // Wire ADS1115 ALERT/RDY pin to pin 25
    const int alertReadyPin = 25;

    void setup() {
        //I2Cdev::begin();  // join I2C bus
        Wire.begin(33,32);
        Serial.begin(115200); // initialize serial communication

        Serial.println("Testing device connections...");
        Serial.println(adc0.testConnection() ? "ADS1115 connection successful" : "ADS1115 connection failed");

        adc0.initialize(); // initialize ADS1115 16 bit A/D chip

        // We're going to do single shot sampling
        adc0.setMode(ADS1115_MODE_SINGLESHOT);

        // Slow things down so that we can see that the "poll for conversion" code works
        adc0.setRate(ADS1115_RATE_8);

        // Set the gain (PGA) +/- 4.096V
        // Note that any analog input must be higher than –0.3V and less than VDD +0.3
        adc0.setGain(ADS1115_PGA_4P096);
        // ALERT/RDY pin will indicate when conversion is ready

        pinMode(alertReadyPin,INPUT_PULLUP);
        adc0.setConversionReadyPinMode();

        // To get output from this method, you'll need to turn on the
        //#define ADS1115_SERIAL_DEBUG // in the ADS1115.h file
        #ifdef ADS1115_SERIAL_DEBUG
        adc0.showConfigRegister();
        Serial.print("HighThreshold="); Serial.println(adc0.getHighThreshold(),BIN);
        Serial.print("LowThreshold="); Serial.println(adc0.getLowThreshold(),BIN);
        #endif
    }

    /** Poll the assigned pin for conversion status
     */
    void pollAlertReadyPin() {
      for (uint32_t i = 0; i<100000; i++)
        if (!digitalRead(alertReadyPin)) return;
       Serial.println("Failed to wait for AlertReadyPin, it's stuck high!");
    }

    void loop() {

        // The below method sets the mux and gets a reading.
        adc0.setMultiplexer(ADS1115_MUX_P0_NG);
        adc0.triggerConversion();
        pollAlertReadyPin();
        Serial.print("A0: "); Serial.print(adc0.getMilliVolts(false)); Serial.print("mV\t");

        adc0.setMultiplexer(ADS1115_MUX_P1_NG);
        adc0.triggerConversion();
        pollAlertReadyPin();
        Serial.print("A1: "); Serial.print(adc0.getMilliVolts(false)); Serial.print("mV\t");

        adc0.setMultiplexer(ADS1115_MUX_P2_NG);
        adc0.triggerConversion();
        pollAlertReadyPin();
        Serial.print("A2: "); Serial.print(adc0.getMilliVolts(false)); Serial.print("mV\t");

        adc0.setMultiplexer(ADS1115_MUX_P3_NG);
        // Do conversion polling via I2C on this last reading:
        Serial.print("A3: "); Serial.print(adc0.getMilliVolts(true)); Serial.print("mV");

        Serial.println(digitalRead(alertReadyPin));
        delay(500);
    }
    ```

## MCP23017 16-bit I/O Expander

How can we control 4 relays without taking up 4 pins (excluding GND and VCC pins)? Simply use the MCP23017 I/O expander that uses I2C for communication (just 2 pins!)

the MCP23017 can control up to 8 IO ports: 4 of the IO pins are reserved for the relays those cannot be re-wired as input or any other purposes, for the other 4 IO pins we've made pins available to connect any other extra sensors. Except that, the Eduponics Mini also has available IO ports for use which are not taken by any other application.

### Features

All the MCP23017 pins can be used for either input or output and they even offer some advance features such as external interrupt on I/O changes, the complete feature list includes:

* 16-Bit Bidirectional I/O Ports
* High-Speed I2C Interface
* Control up to eight IO devices
* Configurable interrupt output pins

To learn more, feel free to refer to the data sheet: [MCP23017 data sheet](https://ww1.microchip.com/downloads/en/DeviceDoc/20001952C.pdf)

### MCP23017 library

In order to control the MCP23017 we've prepared a library that we'll need to save as a file named *mcp23017.py* later on we'll make a smaller, main file that will allow us to use this library in order to define pins as either input or output and control the data flow through them:

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython MCP23017 16-bit I/O Expander
    https://github.com/mcauser/micropython-mcp23017
    MIT License
    Copyright (c) 2019 Mike Causer
    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE.
    """

    __version__ = '0.1.4'

    # register addresses in port=0, bank=1 mode (easier maths to convert)
    _MCP_IODIR        = const(0x00) # R/W I/O Direction Register
    _MCP_IPOL         = const(0x01) # R/W Input Polarity Port Register
    _MCP_GPINTEN      = const(0x02) # R/W Interrupt-on-Change Pins
    _MCP_DEFVAL       = const(0x03) # R/W Default Value Register
    _MCP_INTCON       = const(0x04) # R/W Interrupt-on-Change Control Register
    _MCP_IOCON        = const(0x05) # R/W Configuration Register
    _MCP_GPPU         = const(0x06) # R/W Pull-Up Resistor Register
    _MCP_INTF         = const(0x07) # R   Interrupt Flag Register (read clears)
    _MCP_INTCAP       = const(0x08) # R   Interrupt Captured Value For Port Register
    _MCP_GPIO         = const(0x09) # R/W General Purpose I/O Port Register
    _MCP_OLAT         = const(0x0a) # R/W Output Latch Register

    # Config register (IOCON) bits
    _MCP_IOCON_INTPOL = const(2)
    _MCP_IOCON_ODR    = const(4)
    # _MCP_IOCON_HAEN = const(8) # no used - for spi flavour of this chip
    _MCP_IOCON_DISSLW = const(16)
    _MCP_IOCON_SEQOP  = const(32)
    _MCP_IOCON_MIRROR = const(64)
    _MCP_IOCON_BANK   = const(128)


    class Port():
        # represents one of the two 8-bit ports
        def __init__(self, port, mcp):
            self._port = port & 1  # 0=PortA, 1=PortB
            self._mcp = mcp

        def _which_reg(self, reg):
            if self._mcp._config & 0x80 == 0x80:
                # bank = 1
                return reg | (self._port << 4)
            else:
                # bank = 0
                return (reg << 1) + self._port

        def _flip_property_bit(self, reg, condition, bit):
            if condition:
                setattr(self, reg, getattr(self, reg) | bit)
            else:
                setattr(self, reg, getattr(self, reg) & ~bit)

        def _read(self, reg):
            return self._mcp._i2c.readfrom_mem(self._mcp._address, self._which_reg(reg), 1)[0]

        def _write(self, reg, val):
            val &= 0xff
            self._mcp._i2c.writeto_mem(self._mcp._address, self._which_reg(reg), bytearray([val]))
            # if writing to the config register, make a copy in mcp so that it knows
            # which bank you're using for subsequent writes
            if reg == _MCP_IOCON:
                self._mcp._config = val

        @property
        def mode(self):
            return self._read(_MCP_IODIR)
        @mode.setter
        def mode(self, val):
            self._write(_MCP_IODIR, val)

        @property
        def input_polarity(self):
            return self._read(_MCP_IPOL)
        @input_polarity.setter
        def input_polarity(self, val):
            self._write(_MCP_IPOL, val)

        @property
        def interrupt_enable(self):
            return self._read(_MCP_GPINTEN)
        @interrupt_enable.setter
        def interrupt_enable(self, val):
            self._write(_MCP_GPINTEN, val)

        @property
        def default_value(self):
            return self._read(_MCP_DEFVAL)
        @default_value.setter
        def default_value(self, val):
            self._write(_MCP_DEFVAL, val)

        @property
        def interrupt_compare_default(self):
            return self._read(_MCP_INTCON)
        @interrupt_compare_default.setter
        def interrupt_compare_default(self, val):
            self._write(_MCP_INTCON, val)

        @property
        def io_config(self):
            return self._read(_MCP_IOCON)
        @io_config.setter
        def io_config(self, val):
            self._write(_MCP_IOCON, val)

        @property
        def pullup(self):
            return self._read(_MCP_GPPU)
        @pullup.setter
        def pullup(self, val):
            self._write(_MCP_GPPU, val)

        # read only
        @property
        def interrupt_flag(self):
            return self._read(_MCP_INTF)

        # read only
        @property
        def interrupt_captured(self):
            return self._read(_MCP_INTCAP)

        @property
        def gpio(self):
            return self._read(_MCP_GPIO)
        @gpio.setter
        def gpio(self, val):
            # writing to this register modifies the OLAT register for pins configured as output
            self._write(_MCP_GPIO, val)

        @property
        def output_latch(self):
            return self._read(_MCP_OLAT)
        @output_latch.setter
        def output_latch(self, val):
            # modifies the output latches on pins configured as outputs
            self._write(_MCP_OLAT, val)


    class MCP23017():
        def __init__(self, i2c, address=0x20):
            self._i2c = i2c
            self._address = address
            self._config = 0x00
            self._virtual_pins = {}
            self.init()

        def init(self):
            # error if device not found at i2c addr
            if self._i2c.scan().count(self._address) == 0:
                raise OSError('MCP23017 not found at I2C address {:#x}'.format(self._address))

            self.porta = Port(0, self)
            self.portb = Port(1, self)

            self.io_config = 0x00      # io expander configuration - same on both ports, only need to write once

            # Reset to all inputs with no pull-ups and no inverted polarity.
            self.mode = 0xFFFF                       # in/out direction (0=out, 1=in)
            self.input_polarity = 0x0000             # invert port input polarity (0=normal, 1=invert)
            self.interrupt_enable = 0x0000           # int on change pins (0=disabled, 1=enabled)
            self.default_value = 0x0000              # default value for int on change
            self.interrupt_compare_default = 0x0000  # int on change control (0=compare to prev val, 1=compare to def val)
            self.pullup = 0x0000                     # gpio weak pull up resistor - when configured as input (0=disabled, 1=enabled)
            self.gpio = 0x0000                       # port (0=logic low, 1=logic high)

        def config(self, interrupt_polarity=None, interrupt_open_drain=None, sda_slew=None, sequential_operation=None, interrupt_mirror=None, bank=None):
            io_config = self.porta.io_config

            if interrupt_polarity is not None:
                # configre INT as push-pull
                # 0: Active low
                # 1: Active high
                io_config = self._flip_bit(io_config, interrupt_polarity, _MCP_IOCON_INTPOL)
                if interrupt_polarity:
                    # if setting to 1, unset ODR bit - interrupt_open_drain
                    interrupt_open_drain = False
            if interrupt_open_drain is not None:
                # configure INT as open drain, overriding interrupt_polarity
                # 0: INTPOL sets the polarity
                # 1: Open drain, INTPOL ignored
                io_config = self._flip_bit(io_config, interrupt_open_drain, _MCP_IOCON_ODR)
            if sda_slew is not None:
                # 0: Slew rate function on SDA pin enabled
                # 1: Slew rate function on SDA pin disabled
                io_config = self._flip_bit(io_config, sda_slew, _MCP_IOCON_DISSLW)
            if sequential_operation is not None:
                # 0: Enabled, address pointer increments
                # 1: Disabled, address pointer fixed
                io_config = self._flip_bit(io_config, sequential_operation, _MCP_IOCON_SEQOP)
            if interrupt_mirror is not None:
                # 0: Independent INTA,INTB pins
                # 1: Internally linked INTA,INTB pins
                io_config = self._flip_bit(io_config, interrupt_mirror, _MCP_IOCON_MIRROR)
            if bank is not None:
                # 0: Registers alternate between A and B ports
                # 1: All port A registers first then all port B
                io_config = self._flip_bit(io_config, bank, _MCP_IOCON_BANK)

            # both ports share the same register, so you only need to write on one
            self.porta.io_config = io_config
            self._config = io_config

        def _flip_bit(self, value, condition, bit):
            if condition:
                value |= bit
            else:
                value &= ~bit
            return value

        def pin(self, pin, mode=None, value=None, pullup=None, polarity=None, interrupt_enable=None, interrupt_compare_default=None, default_value=None):
            assert 0 <= pin <= 15
            port = self.portb if pin // 8 else self.porta
            bit = (1 << (pin % 8))
            if mode is not None:
                # 0: Pin is configured as an output
                # 1: Pin is configured as an input
                port._flip_property_bit('mode', mode & 1, bit)
            if value is not None:
                # 0: Pin is set to logic low
                # 1: Pin is set to logic high
                port._flip_property_bit('gpio', value & 1, bit)
            if pullup is not None:
                # 0: Weak pull-up 100k ohm resistor disabled
                # 1: Weak pull-up 100k ohm resistor enabled
                port._flip_property_bit('pullup', pullup & 1, bit)
            if polarity is not None:
                # 0: GPIO register bit reflects the same logic state of the input pin
                # 1: GPIO register bit reflects the opposite logic state of the input pin
                port._flip_property_bit('input_polarity', polarity & 1, bit)
            if interrupt_enable is not None:
                # 0: Disables GPIO input pin for interrupt-on-change event
                # 1: Enables GPIO input pin for interrupt-on-change event
                port._flip_property_bit('interrupt_enable', interrupt_enable & 1, bit)
            if interrupt_compare_default is not None:
                # 0: Pin value is compared against the previous pin value
                # 1: Pin value is compared against the associated bit in the DEFVAL register
                port._flip_property_bit('interrupt_compare_default', interrupt_compare_default & 1, bit)
            if default_value is not None:
                # 0: Default value for comparison in interrupt, when configured to compare against DEFVAL register
                # 1: Default value for comparison in interrupt, when configured to compare against DEFVAL register
                port._flip_property_bit('default_value', default_value & 1, bit)
            if value is None:
                return port.gpio & bit == bit

        def interrupt_triggered_gpio(self, port):
            # which gpio triggered the interrupt
            # only 1 bit will be set
            port = self.portb if port else self.porta
            return port.interrupt_flag

        def interrupt_captured_gpio(self, port):
            # captured gpio values at time of int
            # reading this will clear the current interrupt
            port = self.portb if port else self.porta
            return port.interrupt_captured

        # mode (IODIR register)
        @property
        def mode(self):
            return self.porta.mode | (self.portb.mode << 8)
        @mode.setter
        def mode(self, val):
            self.porta.mode = val
            self.portb.mode = (val >> 8)

        # input_polarity (IPOL register)
        @property
        def input_polarity(self):
            return self.porta.input_polarity | (self.portb.input_polarity << 8)
        @input_polarity.setter
        def input_polarity(self, val):
            self.porta.input_polarity = val
            self.portb.input_polarity = (val >> 8)

        # interrupt_enable (GPINTEN register)
        @property
        def interrupt_enable(self):
            return self.porta.interrupt_enable | (self.portb.interrupt_enable << 8)
        @interrupt_enable.setter
        def interrupt_enable(self, val):
            self.porta.interrupt_enable = val
            self.portb.interrupt_enable = (val >> 8)

        # default_value (DEFVAL register)
        @property
        def default_value(self):
            return self.porta.default_value | (self.portb.default_value << 8)
        @default_value.setter
        def default_value(self, val):
            self.porta.default_value = val
            self.portb.default_value = (val >> 8)

        # interrupt_compare_default (INTCON register)
        @property
        def interrupt_compare_default(self):
            return self.porta.interrupt_compare_default | (self.portb.interrupt_compare_default << 8)
        @interrupt_compare_default.setter
        def interrupt_compare_default(self, val):
            self.porta.interrupt_compare_default = val
            self.portb.interrupt_compare_default = (val >> 8)

        # io_config (IOCON register)
        # This register is duplicated in each port. Changing one changes both.
        @property
        def io_config(self):
            return self.porta.io_config
        @io_config.setter
        def io_config(self, val):
            self.porta.io_config = val

        # pullup (GPPU register)
        @property
        def pullup(self):
            return self.porta.pullup | (self.portb.pullup << 8)
        @pullup.setter
        def pullup(self, val):
            self.porta.pullup = val
            self.portb.pullup = (val >> 8)

        # interrupt_flag (INTF register)
        # read only
        @property
        def interrupt_flag(self):
            return self.porta.interrupt_flag | (self.portb.interrupt_flag << 8)

        # interrupt_captured (INTCAP register)
        # read only
        @property
        def interrupt_captured(self):
            return self.porta.interrupt_captured | (self.portb.interrupt_captured << 8)

        # gpio (GPIO register)
        @property
        def gpio(self):
            return self.porta.gpio | (self.portb.gpio << 8)
        @gpio.setter
        def gpio(self, val):
            self.porta.gpio = val
            self.portb.gpio = (val >> 8)

        # output_latch (OLAT register)
        @property
        def output_latch(self):
            return self.porta.output_latch | (self.portb.output_latch << 8)
        @output_latch.setter
        def output_latch(self, val):
            self.porta.output_latch = val
            self.portb.output_latch = (val >> 8)

        # list interface
        # mcp[pin] lazy creates a VirtualPin(pin, port)
        def __getitem__(self, pin):
            assert 0 <= pin <= 15
            if not pin in self._virtual_pins:
                self._virtual_pins[pin] = VirtualPin(pin, self.portb if pin // 8 else self.porta)
            return self._virtual_pins[pin]

    class VirtualPin():
        def __init__(self, pin, port):
            self._pin = pin % 8
            self._bit = 1 << self._pin
            self._port = port

        def _flip_bit(self, value, condition):
            return value | self._bit if condition else value & ~self._bit

        def _get_bit(self, value):
            return (value & self._bit) >> self._pin

        def value(self, val=None):
            # if val, write, else read
            if val is not None:
                self._port.gpio = self._flip_bit(self._port.gpio, val & 1)
            else:
                return self._get_bit(self._port.gpio)

        def input(self, pull=None):
            # if pull, enable pull up, else read
            self._port.mode = self._flip_bit(self._port.mode, 1) # mode = input
            if pull is not None:
                self._port.pullup = self._flip_bit(self._port.pullup, pull & 1) # toggle pull up

        def output(self, val=None):
            # if val, write, else read
            self._port.mode = self._flip_bit(self._port.mode, 0) # mode = output
            if val is not None:
                self._port.gpio = self._flip_bit(self._port.gpio, val & 1)
    ```

### MCP23017 Example code

Once we have the library ready, it's time to use it. let's create a simple demo where we will activate and deactivate each relay with half a second interval, this will allow us to test and understand how to control the relays.

First we need to import the mcp23017 library and configure the I2C to SCL pin 33 and SDA pin 32. Next, we will configure the default 0x20 I2C address for the mcp23017.

By default, we want the relays to be close and we want to configure them as output so we use the command mcp.pin() to set the value to 0 and the mode to 0 (mode 1 means input)

If we want to use the rest of the available IO pins we can change the mcp[x] (where x is a number) to a number between 4 to 7 (including) as 0 to 3 already taken by the relays (total of 8 IO pins), the external IO pins can be used both as input and output.

For the Arduino demo - you'll need to install the arduino-mcp23017 library: [arduino-mcp23017](https://github.com/blemasle/arduino-mcp23017)

=== "MicroPython"
    ``` python linenums="1"
    from machine import Pin, I2C
    import mcp23017
    import time

    i2c = I2C(scl=Pin(33), sda=Pin(32))

    mcp = mcp23017.MCP23017(i2c, 0x20)

    # set pins to low
    mcp.pin(0, mode=0, value=0)
    mcp.pin(1, mode=0, value=0)
    mcp.pin(2, mode=0, value=0)
    mcp.pin(3, mode=0, value=0)

    # open relays
    print("Turn on relays")
    mcp[0].output(1)
    time.sleep(0.5)
    mcp[1].output(1)
    time.sleep(0.5)
    mcp[2].output(1)
    time.sleep(0.5)
    mcp[3].output(1)
    time.sleep(0.5)

    # close relays
    print("Turn off relays")
    mcp[0].output(0)
    time.sleep(0.5)
    mcp[1].output(0)
    time.sleep(0.5)
    mcp[2].output(0)
    time.sleep(0.5)
    mcp[3].output(0)
    ```
=== "Arduino"
    ``` C++ linenums="1"
    /**
     * On every loop, the state of the port B is copied to port A.
     *
     * Use active low inputs on port A. Internal pullups are enabled by default by the library so there is no need for external resistors.
     * Place LEDS on port B for instance.
     * When pressing a button, the corresponding led is shut down.
     *
     * You can also uncomment one line to invert the input (when pressing a button the corresponding led is lit)
     */
    #include <Wire.h>
    #include <MCP23017.h>

    #define MCP23017_ADDR 0x20
    MCP23017 mcp = MCP23017(MCP23017_ADDR);

    void setup() {
        Wire.begin(33,32);
        Serial.begin(115200);

        mcp.init();

        mcp.portMode(MCP23017Port::A, 0); //Port A as output
        mcp.portMode(MCP23017Port::B, 0); //Port B as output
        mcp.portMode(MCP23017Port::C, 0); //Port C as output
        mcp.portMode(MCP23017Port::D, 0); //Port D as output

        mcp.writeRegister(MCP23017Register::GPIO_A, 0x00);  //Reset port A
        mcp.writeRegister(MCP23017Register::GPIO_B, 0x00);  //Reset port B
        mcp.writeRegister(MCP23017Register::GPIO_C, 0x00);  //Reset port C
        mcp.writeRegister(MCP23017Register::GPIO_D, 0x00);  //Reset port D
    }

    void loop() {
        // open relay A
        mcp.writePort(MCP23017Port::A, 1);
        delay(500);
        // open relay B
        mcp.writePort(MCP23017Port::B, 1);
        delay(500);
        // open relay C
        mcp.writePort(MCP23017Port::C, 1);
        delay(500);
        // open relay D
        mcp.writePort(MCP23017Port::D, 1);
        delay(500);
        // close relay A
        mcp.writePort(MCP23017Port::A, 0);
        delay(500);
        // close relay B
        mcp.writePort(MCP23017Port::B, 0);
        delay(500);
        // close relay C
        mcp.writePort(MCP23017Port::C, 0);
        delay(500);
        // close relay D
        mcp.writePort(MCP23017Port::D, 0);
        delay(500);
    }
    ```
