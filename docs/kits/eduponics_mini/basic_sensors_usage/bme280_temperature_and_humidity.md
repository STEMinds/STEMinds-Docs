---
title: BME280 Temperature, Humidity, air pressure sensor
description: Bosch BME280 sensor is a humidity, temperature and barometric pressure sensor especially developed for mobile applications and wearables where size and low power consumption are key design parameters.
---

# BME280 Temperature & Humidity sensor

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/bme280.png">
</p>

The Bosch BME280 sensor is a humidity, temperature and barometric pressure sensor especially developed for mobile applications and wearables where size and low power consumption are key design parameters.
<br/><br/>
The unit combines high linearity and high accuracy sensors and is perfectly feasible for low current consumption, long-term stability and high EMC robustness.
<br/><br/>
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
<br/><br/>
Next, we'll learn by code example how we can read the data from the sensor and use it for our own needs.

## Software explanation

!!! Info "Make sure micropython-eduponics is installed through upip"
    For our Python code, we will need to import eduponics library, make sure you followed the introduction guide on installing the library on the ESP32 Eduponics Mini board.

For the following code create a new file and call it <b>bme.py</b> using Thonny IDE and copy paste the code into it. if we walk through the code, it's basically based on the BME280 Bosch schematics and communication methods using the I2C protocol, we have functions to read humidity, temperature and air pressure. all the functions in this code are commented and easy to understand what's going on.
<br/><br/>
In our main code we import the library (make sure you've installed micropython-eduponics through upip) by calling from eduponics import bme280.
<br/><br/>
Then we need to configure the I2C connection which in our case is SCL Pin 15 and SDA pin 4, we can initalize the bme280 sensor object using those I2C credentials, we don't need to supply address because the library already includes the default address which is 0x76.
<br/><br/>
Now what left to do is to loop every second and read the sensor values. We can get pretty interesting data such as: temperature, humidity, pressure, sea level and even altitude which is calculate using the air pressure!
<br/><br/>
If you are wondering how, it's called [The Barometric Formula](https://en.wikipedia.org/wiki/Barometric_formula).

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython BME280 temperature, humidity and air-pressure sensor
    https://github.com/STEMinds/eduponics-mini
    MIT License
    Copyright (c) 2020 STEMinds
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

    from machine import I2C,Pin
    from Eduponics import bme280
    from utime import sleep

    # setup I2C connection
    i2c = I2C(scl=Pin(15), sda=Pin(4))

    # Initialize BME280 object with default address 0x76
    bme280 = bme280.BME280(i2c=i2c)

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
