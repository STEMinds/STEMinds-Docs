---
title: STEMinds - BME280 Temperature and Humidity sensor
description: Bosch BME280 sensor is a humidity, temperature and barometric pressure sensor especially developed for mobile applications and wearables where size and low power consumption are key design parameters.
---

# BME280 Temperature & Humidity sensor

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/bme280.png">
</p>

The Bosch BME280 sensor is a humidity, temperature and barometric pressure sensor especially developed for mobile applications and wearables where size and low power consumption are key design parameters.

The unit combines high linearity and high accuracy sensors and is perfectly feasible for low current consumption, long-term stability and high EMC robustness.

The combined sensor offers an extremely fast response time and therefore supports performance requirements for emerging applications such as context awareness, and high accuracy over a wide temperature range.

## Specifications

* 3 in 1 - temperature, humidity and barometric pressure
* Low power consumption
* Extremely fast and accurate temperature, humidity and pressure reading
* Barometric pressure operation range 300-1100hPa, temperature operation range -40-85°C.
* The humidity range is 0-100% real humidity.
* 10 Years lifespan.

View the complete datasheet at Bosch website: [BME280 3 in 1 sensor datasheet](https://www.bosch-sensortec.com/media/boschsensortec/downloads/datasheets/bst-bme280-ds002.pdf)

## Possible applications

Some of the great possible applications we can use the BME280 for are:

* Temperature and humidity measurement
* Barometric pressure measurement
* Measuring approximate height using the barometric sensor
* Weather station
* Indoor air quality monitoring
* Smart agriculture
* Green house monitoring

## Hardware explanation

There is not much information on the inside of the BME280 IC (integrated circuit) we guess it's because its Bosch patented technology. what we do know is that the tiny IC integrates 3 features inside and we can use the I2C protocol to read data from it.

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/BME280_schematic.png">
</p>

In the schematic above we can see how we've connected the BME280 to our Eduponics mini ESP32 board (MCU is where our ESP32 is). The BME280 requires only 1.8V voltage of operation, extremely low in power requirements as well as power consumption.

Next, we'll learn by code example how we can read the data from the sensor and use it for our own needs.

## Software explanation

This part might look scary a bit.

I mean, look at the length of this code! don't worry, we can go through it!

In this part we'll need 2 files, one which will be our main library code for the BME280 sensor communication and another one which will be the main code which is only 23 lines long!

For the following code create a new file and call it <b>bme.py</b> using Thonny IDE and copy paste the code into it. if we walk through the code, it's basically based on the BME280 Bosch schematics and communication methods using the I2C protocol, we have functions to read humidity, temperature and air pressure. all the functions in this code are commented and easy to understand what's going on.

=== "MicroPython"

    ``` python linenums="1"
    # This module is based on the below cited resources, which are all
    # based on the documentation as provided in the Bosch Data Sheet and
    # the sample implementation provided therein.
    #
    # Final Document: BST-BME280-DS002-15
    #
    # Authors: Paul Cunnane 2016, Peter Dahlebrg 2016
    #
    # This module borrows from the Adafruit BME280 Python library. Original
    # Copyright notices are reproduced below.
    #
    # Those libraries were written for the Raspberry Pi. This modification is
    # intended for the MicroPython and esp8266 boards.
    #
    # Copyright (c) 2014 Adafruit Industries
    # Author: Tony DiCola
    #
    # Based on the BMP280 driver with BME280 changes provided by
    # David J Taylor, Edinburgh (www.satsignal.eu)
    #
    # Based on Adafruit_I2C.py created by Kevin Townsend.
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

    import time
    from ustruct import unpack
    from array import array

    # BME280 default address.
    BME280_I2CADDR = 0x76

    # Operating Modes
    BME280_OSAMPLE_1 = 1
    BME280_OSAMPLE_2 = 2
    BME280_OSAMPLE_4 = 3
    BME280_OSAMPLE_8 = 4
    BME280_OSAMPLE_16 = 5

    BME280_REGISTER_CONTROL_HUM = 0xF2
    BME280_REGISTER_STATUS = 0xF3
    BME280_REGISTER_CONTROL = 0xF4

    MODE_SLEEP = const(0)
    MODE_FORCED = const(1)
    MODE_NORMAL = const(3)

    class BME280:

        def __init__(self,
                     mode=BME280_OSAMPLE_8,
                     address=BME280_I2CADDR,
                     i2c=None,
                     **kwargs):
            # Check that mode is valid.
            if mode not in [BME280_OSAMPLE_1, BME280_OSAMPLE_2, BME280_OSAMPLE_4,
                            BME280_OSAMPLE_8, BME280_OSAMPLE_16]:
                raise ValueError(
                    'Unexpected mode value {0}. Set mode to one of '
                    'BME280_OSAMPLE_1, BME280_OSAMPLE_2, BME280_OSAMPLE_4,'
                    'BME280_OSAMPLE_8, BME280_OSAMPLE_16'.format(mode))
            self._mode = mode
            self.address = address
            if i2c is None:
                raise ValueError('An I2C object is required.')
            self.i2c = i2c
            self.__sealevel = 101325

            # load calibration data
            dig_88_a1 = self.i2c.readfrom_mem(self.address, 0x88, 26)
            dig_e1_e7 = self.i2c.readfrom_mem(self.address, 0xE1, 7)

            self.dig_T1, self.dig_T2, self.dig_T3, self.dig_P1, \
                self.dig_P2, self.dig_P3, self.dig_P4, self.dig_P5, \
                self.dig_P6, self.dig_P7, self.dig_P8, self.dig_P9, \
                _, self.dig_H1 = unpack("<HhhHhhhhhhhhBB", dig_88_a1)

            self.dig_H2, self.dig_H3, self.dig_H4,\
                self.dig_H5, self.dig_H6 = unpack("<hBbhb", dig_e1_e7)
            # unfold H4, H5, keeping care of a potential sign
            self.dig_H4 = (self.dig_H4 * 16) + (self.dig_H5 & 0xF)
            self.dig_H5 //= 16

            # temporary data holders which stay allocated
            self._l1_barray = bytearray(1)
            self._l8_barray = bytearray(8)
            self._l3_resultarray = array("i", [0, 0, 0])

            self._l1_barray[0] = self._mode << 5 | self._mode << 2 | MODE_SLEEP
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL,
                                 self._l1_barray)
            self.t_fine = 0

        def read_raw_data(self, result):
            """ Reads the raw (uncompensated) data from the sensor.
                Args:
                    result: array of length 3 or alike where the result will be
                    stored, in temperature, pressure, humidity order
                Returns:
                    None
            """

            self._l1_barray[0] = self._mode
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL_HUM,
                                 self._l1_barray)
            self._l1_barray[0] = self._mode << 5 | self._mode << 2 | MODE_FORCED
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL,
                                 self._l1_barray)

            # Wait for conversion to complete
            while self.i2c.readfrom_mem(self.address, BME280_REGISTER_STATUS, 1)[0] & 0x08:
                time.sleep_ms(5)

            # burst readout from 0xF7 to 0xFE, recommended by datasheet
            self.i2c.readfrom_mem_into(self.address, 0xF7, self._l8_barray)
            readout = self._l8_barray
            # pressure(0xF7): ((msb << 16) | (lsb << 8) | xlsb) >> 4
            raw_press = ((readout[0] << 16) | (readout[1] << 8) | readout[2]) >> 4
            # temperature(0xFA): ((msb << 16) | (lsb << 8) | xlsb) >> 4
            raw_temp = ((readout[3] << 16) | (readout[4] << 8) | readout[5]) >> 4
            # humidity(0xFD): (msb << 8) | lsb
            raw_hum = (readout[6] << 8) | readout[7]

            result[0] = raw_temp
            result[1] = raw_press
            result[2] = raw_hum

        def read_compensated_data(self, result=None):
            """ Reads the data from the sensor and returns the compensated data.
                Args:
                    result: array of length 3 or alike where the result will be
                    stored, in temperature, pressure, humidity order. You may use
                    this to read out the sensor without allocating heap memory
                Returns:
                    array with temperature, pressure, humidity. Will be the one
                    from the result parameter if not None
            """
            self.read_raw_data(self._l3_resultarray)
            raw_temp, raw_press, raw_hum = self._l3_resultarray
            # temperature
            var1 = (raw_temp/16384.0 - self.dig_T1/1024.0) * self.dig_T2
            var2 = raw_temp/131072.0 - self.dig_T1/8192.0
            var2 = var2 * var2 * self.dig_T3
            self.t_fine = int(var1 + var2)
            temp = (var1 + var2) / 5120.0
            temp = max(-40, min(85, temp))

            # pressure
            var1 = (self.t_fine/2.0) - 64000.0
            var2 = var1 * var1 * self.dig_P6 / 32768.0 + var1 * self.dig_P5 * 2.0
            var2 = (var2 / 4.0) + (self.dig_P4 * 65536.0)
            var1 = (self.dig_P3 * var1 * var1 / 524288.0 + self.dig_P2 * var1) / 524288.0
            var1 = (1.0 + var1 / 32768.0) * self.dig_P1
            if (var1 == 0.0):
                pressure = 30000  # avoid exception caused by division by zero
            else:
                p = ((1048576.0 - raw_press) - (var2 / 4096.0)) * 6250.0 / var1
                var1 = self.dig_P9 * p * p / 2147483648.0
                var2 = p * self.dig_P8 / 32768.0
                pressure = p + (var1 + var2 + self.dig_P7) / 16.0
                pressure = max(30000, min(110000, pressure))

            # humidity
            h = (self.t_fine - 76800.0)
            h = ((raw_hum - (self.dig_H4 * 64.0 + self.dig_H5 / 16384.0 * h)) *
                 (self.dig_H2 / 65536.0 * (1.0 + self.dig_H6 / 67108864.0 * h *
                                           (1.0 + self.dig_H3 / 67108864.0 * h))))
            humidity = h * (1.0 - self.dig_H1 * h / 524288.0)
            # humidity = max(0, min(100, humidity))

            if result:
                result[0] = temp
                result[1] = pressure
                result[2] = humidity
                return result

            return array("f", (temp, pressure, humidity))

        @property
        def sealevel(self):
            return self.__sealevel

        @sealevel.setter
        def sealevel(self, value):
            if 30000 < value < 120000:  # just ensure some reasonable value
                self.__sealevel = value

        @property
        def altitude(self):
            '''
            Altitude in m.
            '''
            from math import pow
            try:
                p = 44330 * (1.0 - pow(self.read_compensated_data()[1] /
                                       self.__sealevel, 0.1903))
            except:
                p = 0.0
            return p

        @property
        def dew_point(self):
            """
            Compute the dew point temperature for the current Temperature
            and Humidity measured pair
            """
            from math import log
            t, p, h = self.read_compensated_data()
            h = (log(h, 10) - 2) / 0.4343 + (17.62 * t) / (243.12 + t)
            return 243.12 * h / (17.62 - h)

        @property
        def values(self):
            """ human readable values """

            t, p, h = self.read_compensated_data()

            return ("{:.2f}C".format(t), "{:.2f}hPa".format(p/100),
                    "{:.2f}%".format(h))
    ```

Once we have the library ready, it's time to move to our main code.
In our main code we import the library (make sure you saved it as <b>bme.py</b>) by calling from bme import * (from bme library import all the functions).

Then we need to configure the I2C connection which in our case is SCL Pin 15 and SDA pin 4, we can initalize the bme280 sensor object using those I2C credentials, we don't need to supply address because the library already includes the default address which is 0x76.

Now what left to do is to loop every second and read the sensor values. We can get pretty interesting data such as: temperature, humidity, pressure, sea level and even altitude which is calculate using the air pressure!

if you are wondering how, it's called [The Barometric Formula](https://en.wikipedia.org/wiki/Barometric_formula).

=== "MicroPython"

    ``` python linenums="1"
    from machine import I2C,Pin
    from bme import *
    from utime import sleep

    # setup I2C connection
    i2c = I2C(scl=Pin(15), sda=Pin(4))

    # Initialize BME280 object with default address 0x76
    bme280 = BME280(i2c=i2c)

    while True:
        # get the values from the BME280 library
        values = bme280.values
        altitude = bme280.altitude
        dew_point = bme280.dew_point
        sea_level = bme280.sealevel
        # print the values every 1 second
        print("------------------------")
        print("Temperature: %s" % values[0])
        print("Humidity: %s" % values[1])
        print("Pressure: %s" % values[2])
        print("Altitude: %s" % altitude)
        print("Dew point: %s" % dew_point)
        print("Sea level: %s" % sea_level)
        print("------------------------")
        print("")
        sleep(1)
    ```

In Arduino IDE it would be differently, we can use the Adafruit libraries that are included in our Eduponics-Mini git repository to import them directly and write a shorter code:

=== "ESP32-Arduino"
      ``` C++ linenums="1"
      #include <Wire.h>
      #include <Adafruit_Sensor.h>
      #include <Adafruit_BME280.h>

      #define SEALEVELPRESSURE_HPA (1013.25)

      Adafruit_BME280 bme; // I2C communication object

      unsigned long delayTime;

      void setup() {
        // start serial communication
        Serial.begin(115200);
        // start I2C on pins 4 and 15
        Wire.begin(4, 15);
        // start BME280 sensor on address 0x76
        bme.begin(0x76);  
        delayTime = 1000;
      }

      void loop() {
        printValues();
        delay(delayTime);
      }

      void printValues() {
        Serial.print("Temperature = ");
        Serial.print(bme.readTemperature());
        Serial.println(" *C");

        // Convert temperature to Fahrenheit
        /*Serial.print("Temperature = ");
        Serial.print(1.8 * bme.readTemperature() + 32);
        Serial.println(" *F");*/

        Serial.print("Pressure = ");
        Serial.print(bme.readPressure() / 100.0F);
        Serial.println(" hPa");

        Serial.print("Approx. Altitude = ");
        Serial.print(bme.readAltitude(SEALEVELPRESSURE_HPA));
        Serial.println(" m");

        Serial.print("Humidity = ");
        Serial.print(bme.readHumidity());
        Serial.println(" %");

        Serial.println();
      }
      ```
