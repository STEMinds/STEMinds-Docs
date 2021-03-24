---
title: STEMinds - BH1750 IIC light sensor
description: The BH1750 is integrated IC with photodiode that allows us to measure the intensity and Illuminance of light and understanding whenever our environment is sufficient for our plant to live or it won't be able to survive due to lack of sunlight.
---

# BH1750 Light sensor

The BH1750 is integrated IC with photodiode that allows us to measure the intensity and Illuminance of light and understanding whenever our environment is sufficient for our plant to live or it won't be able to survive due to lack of sunlight.
<br/><br/>
The BH1750 sensor is already integrated into the Eduponics Mini kit and we use I2C protocol to communicate with it.

## Sensor Specifications

The BH1750 IC includes a lot of features, let's take a look at some of them:

2. Spectral response speed is close to human eye response
3. Illuminance analog to Digital Converter (Integrated ADC)
4. Wide range and High resolution. ( 1 - 65535 lx )
5. Low consumption on "power down" mode.
6. 50Hz / 60Hz Light noise rejection functionality
7. Infrared influence is extremely minimal, no interference.

The complete BH1750 datasheet can be found here: [mouser.com BH1750 datasheet](https://www.mouser.com/datasheet/2/348/bh1750fvi-e-186247.pdf)

## Possible applications

There are a lot of applications for light sensors, not just in the field of smart agriculture, some of them are:

* Can be used as light sensor.
* In agriculture can let you know if a plant has enough light.
* Useful for measuring the intensity of light.
* Night light and photography light meters use similar sensors to "sense" the environment.
* Night lamp to turn it on when dark and turn it off when it's light
* Infrared astronomy and Infrared Spectroscopy also use similar sensors for measuring mid-infrared spectral region.
* Your laptop or phone use similar sensor to adjust LCD back-light intensity


## Photodiode explained

From Wikipedia:

>A photodiode is a semiconductor device that converts light into an electrical current. The current is generated when photons are absorbed in the photodiode. Photodiodes may contain optical filters, built-in lenses, and may have large or small surface areas.

> A photodiode is a PIN structure or p–n junction. When a photon of sufficient energy strikes the diode, it creates an electron–hole pair. This mechanism is also known as the inner photoelectric effect. If the absorption occurs in the junction's depletion region, or one diffusion length away from it, these carriers are swept from the junction by the built-in electric field of the depletion region.

>Thus holes move toward the anode, and electrons toward the cathode, and a photocurrent is produced. The total current through the photodiode is the sum of the dark current (current that is generated in the absence of light) and the photocurrent, so the dark current must be minimized to maximize the sensitivity of the device"

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/photodiode_explained.jpg">
</p>
<p align="center">
</p>
<center>
  Illustration borrowed from: [ElectronicsCoach.com](https://electronicscoach.com/photodiode.html")
</center>

<br/><br/>

The BH1750 is a very sophisticated IC (integrated circuit) that includes multiple components. Let's walk through the data sheet shown in the picture below to understand the principles better.

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/bh1750.png">
</p>

* PD - Photo diode with approximate human eye response, this is the main component in the IC that allow us to detect the amount of light.
* AMP - Integration OPAMP for converting from PD current to Voltage as the current by itself is not much use for us.
* ADC - Analog to digital converter for obtaining Digital 16bit data.
* Logic + I2C Interface - Ambient Light Calculation and I2C BUS Interface, we read the lux value through the I2C interface.
* Internal Oscillator ( 320kHz ). It is CLK for internal logic.

As we mentioned, our main component inside the IC is the PD which is photo-diode that allows us to read the amount of ambient light, the other components that are integrated in the IC allow us to convert the value and receive it in a convenient way through I2C interface.

## Software example

!!! Info "Make sure micropython-eduponics is installed through upip"
    For our Python code, we will need to import eduponics library, make sure you followed the introduction guide on installing the library on the ESP32 Eduponics Mini board.

In the following code example, we'll use a class called "LightSensor" from the micropython-eduponics library, this class we define some of the main functions that we need in order to interact with our light sensor.
<br/><br/>
The code is very straight forward: we'll create an object from the BH1750 library called "light" and execute a single "readLight" function on the object to recieve the light intensity in lux.

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython BH1750 light sensor module
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

    from eduponics import bh1750
    import machine
    import time

    # initialize the bh1750 sensor
    light = bh1750.BH1750()

    # while true, print the light values in lux
    while True:
        print(light.readLight())
        time.sleep(1)
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    #include <Wire.h>
    #include <BH1750.h>

    BH1750 lightMeter(0x5C);


    void setup(){
      // start serial communication
      Serial.begin(115200);
      // start I2C on pins 4 and 15
      Wire.begin(4,15);
      // initialize the bh1750
      lightMeter.begin(BH1750::CONTINUOUS_HIGH_RES_MODE, 0x5C, &Wire);
    }


    void loop() {

      float lux = lightMeter.readLightLevel();
      Serial.print("Light: ");
      Serial.print(lux);
      Serial.println(" lx");
      delay(3000);

    }
    ```
