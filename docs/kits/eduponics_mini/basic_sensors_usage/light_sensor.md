---
title: STEMinds - BH1750 IIC light sensor
description: The BH1750 is integrated IC with photodiode that allows us to measure the intensity and Illuminance of light and understanding whenever our environment is sufficient for our plant to live or it won't be able to survive due to lack of sunlight.
---

# BH1750 Light sensor

The BH1750 is integrated IC with photodiode that allows us to measure the intensity and Illuminance of light and understanding whenever our environment is sufficient for our plant to live or it won't be able to survive due to lack of sunlight.

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
  <img src="/kits/eduponics_mini/images/photodiode_explained.jpg">
</p>
<p align="center">
</p>
<center>
  Illustration borrowed from: [ElectronicsCoach.com](https://electronicscoach.com/photodiode.html")
</center>


The BH1750 is a very sophisticated IC (integrated circuit) that includes multiple components. Let's walk through the data sheet shown in the picture below to understand the principles better.

<p align="center">
  <img src="/kits/eduponics_mini/images/bh1750.png">
</p>

* PD - Photo diode with approximate human eye response, this is the main component in the IC that allow us to detect the amount of light.
* AMP - Integration OPAMP for converting from PD current to Voltage as the current by itself is not much use for us.
* ADC - Analog to digital converter for obtaining Digital 16bit data.
* Logic + I2C Interface - Ambient Light Calculation and I2C BUS Interface, we read the lux value through the I2C interface.
* Internal Oscillator ( 320kHz ). It is CLK for internal logic.

As we mentioned, our main component inside the IC is the PD which is photo-diode that allows us to read the amount of ambient light, the other components that are integrated in the IC allow us to convert the value and receive it in a convenient way through I2C interface.

## Software example

In the following code example, we'll create a class called "LightSensor". in this class we will define some of the main functions that we need.

In the initializer, we will define the addresses that we'll need to communicate through I2C. The addresses are mentioned in the BH1750 script such as measurement resolutions and low power mode by turning off the sensor after successful measurement.

The next step will be to create a function called readLight() which will communicate through I2C to get the data from the BH1750 sensor. Last by not least, we will make a function called convertToNumber that will take the reading from readLight and convert it into a readable format - 2 bytes of data into a decimal.

For the main code, we can either call this function from another file or use it as is by creating an object called sensor which will initialize LightSensor class and then call sensor.readLight() to get the value of Illuminance in the room in lx.

=== "MicroPython"
    ``` python linenums="1"
    import machine

    class LightSensor():

        def __init__(self):

            # Define some constants from the datasheet

            self.DEVICE = 0x5c # Default device I2C address

            self.POWER_DOWN = 0x00 # No active state
            self.POWER_ON = 0x01 # Power on
            self.RESET = 0x07 # Reset data register value

            # Start measurement at 4lx resolution. Time typically 16ms.
            self.CONTINUOUS_LOW_RES_MODE = 0x13
            # Start measurement at 1lx resolution. Time typically 120ms
            self.CONTINUOUS_HIGH_RES_MODE_1 = 0x10
            # Start measurement at 0.5lx resolution. Time typically 120ms
            self.CONTINUOUS_HIGH_RES_MODE_2 = 0x11
            # Start measurement at 1lx resolution. Time typically 120ms
            # Device is automatically set to Power Down after measurement.
            self.ONE_TIME_HIGH_RES_MODE_1 = 0x20
            # Start measurement at 0.5lx resolution. Time typically 120ms
            # Device is automatically set to Power Down after measurement.
            self.ONE_TIME_HIGH_RES_MODE_2 = 0x21
            # Start measurement at 1lx resolution. Time typically 120ms
            # Device is automatically set to Power Down after measurement.
            self.ONE_TIME_LOW_RES_MODE = 0x23
            # setup I2C
            self.i2c = machine.I2C(scl=machine.Pin(15), sda=machine.Pin(4))

        def convertToNumber(self, data):

            # Simple function to convert 2 bytes of data
            # into a decimal number
            return ((data[1] + (256 * data[0])) / 1.2)

        def readLight(self):

            data = self.i2c.readfrom_mem(self.DEVICE,self.ONE_TIME_HIGH_RES_MODE_1,2)
            return self.convertToNumber(data)

    sensor = LightSensor()
    light = sensor.readLight()
    print("Light in the room: %slx" % light)
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
