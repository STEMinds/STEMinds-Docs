---
title: STEMinds - DHT11 temperature and humidity sensor
description: DHT11 Temperature & Humidity Sensor features a temperature & humidity sensor complex with a calibrated digital signal output.
---

# DHT11 Temperature & Humidity sensor

DHT11 Temperature & Humidity Sensor features a temperature & humidity sensor complex with a calibrated digital signal output.

By using the exclusive digital-signal-acquisition technique and temperature & humidity sensing technology, it ensures high reliability and
excellent long-term stability. This sensor includes a resistive-type humidity measurement
component and an NTC temperature measurement component, and connects to a high performance 8-bit microcontroller, offering excellent quality, fast response, anti-interference ability and cost-effectiveness

!!! Warning "DHT11 sensor is not included with the kit"
    We've decided to replace the DHT11 with BME280 due to BME280 being a better, high accuracy sensor with more features.
    However, we've left the ports available so you could solder your own DHT sensor if you wish. The location of the pins are (4 pins) are right below the BME280 sensor on the Eduponics Mini board.

## Comparison to BME280

When we've tested multiple temperature and humidity sensors to add to the kit we've decided to not let quality and performance down.
The DHT11 is simple to use and easy to learn sensor but not suitable for applications that require high sensitivity and reliability.

We haven't included DHT11 with the kit but we left an available pins to connect one if you wish, BME280 is more sufficient, accurate and suitable for long term industrial real life applications.

## Specifications

Each DHT11 sensor is calibrated in a special laboratory that is extremely accurate on humidity calibration.

The calibration coefficients are stored in the program OTP memory,
which is used by the sensor’s internal signal detecting process. The single-wire serial interface makes system integration quick and easy. Its small size, low power consumption, and up-to-20 meter signal transmission making it the best choice for various applications, including those most demanding ones. The component is a 4-pin single row pin package.

Here are some of the main specifications you'll need to know about the DHT11 sensor:

* 1% resolution, for example it's possible to get 25°C but not 25.3°C
* Good for 20-80% humidity readings with 5% accuracy
* Good for 0-50°C temperature readings ±2°C accuracy
* No more than 1 Hz sampling rate (once every second)

 The complete data sheet for DHT11 can be found here: [Mouser.com DHT11 datasheet](https://www.mouser.com/datasheet/2/758/DHT11-Technical-Data-Sheet-Translated-Version-1143054.pdf)

## Possible applications

The DHT11 is very useful sensor often used in environment related projects such as:

* Weather station
* Indoor air quality monitoring
* Smart agriculture
* Green house monitoring

We will use the DHT11 to measure the environment around our plant, making sure it's not too cold and not too hot either.
Regarding the humidity, multiple actions can be taken such as alert, automatic window opener and more.

## Hardware explanation

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/dht11_inside.jpg">
</p>


The DHT11 sensor can measure both temperature and humidity.

The temperature is measured with the help of a NTC thermistor or negative temperature coefficient thermistor. These thermistors are usually made with semiconductors, ceramic and polymers. The resistance of the device is inversely proportional with temperature and follows a hyperbolic curve. Temperature using NTC often found out [Steinhart Hart equation](https://en.wikipedia.org/wiki/Steinhart%E2%80%93Hart_equation).

The sensor's humidity determined using a moisture dependent resistor.
It has two electrodes with a moisture-holding substrate between them that holds the moisture. The result is the conductance and resistance change as the humidity changes.

Both these temperatures and humidity changes analyzed by an IC placed on the other side of the board. It calculates the values of both and can transmit those values to a microcontroller using only a single data line.

## Software explanation

Controlling the DHT11 is very easy due to the internal MicroPython supported library DHT.

The only thing we need is to import the DHT library that is built-in in MicroPython and initialize it using dht.DHT11() with the PIN that we use.
In our case, the Eduponics Mini uses IO PIN 19 for the DHT11 sensor.

Then, we'll use the measure function to measure the temperature and humidity, put them into variables, and finally print them on screen.

If you prefer Fahrenheit instead of Celcius, make sure to uncomment the conversion lines.

=== "MicroPython"
    ``` python linenums="1"
    import dht
    import machine

    # initialize dht object, DHT11 coonected to IO19
    d = dht.DHT11(machine.Pin(19))

    # measure sensor data
    d.measure()

    # get data
    temp = d.temperature()
    humidity = d.humidity()

    # uncomment for temperature in Fahrenheit
    #temp = (temp / 100) * (9/5) + 32
    #temp = str(round(temp, 2))

    # print temperature and humidity
    print("temperature : %s" % temp)
    print("humidity : %s" % humidity)
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    #include "DHT.h"

    #define DHTPIN 19     // Digital pin connected to the DHT sensor

    // Uncomment whatever type you're using!
    #define DHTTYPE DHT11   // DHT 11
    //#define DHTTYPE DHT22   // DHT 22  (AM2302), AM2321
    //#define DHTTYPE DHT21   // DHT 21 (AM2301)

    // Initialize DHT sensor.
    // Note that older versions of this library took an optional third parameter to
    // tweak the timings for faster processors.  This parameter is no longer needed
    // as the current DHT reading algorithm adjusts itself to work on faster procs.

    DHT dht(DHTPIN, DHTTYPE);

    void setup() {
      Serial.begin(115200);
      Serial.println(F("DHTxx test!"));
      dht.begin();
    }

    void loop() {
      // Wait a few seconds between measurements.
      delay(2000);

      // Reading temperature or humidity takes about 250 milliseconds!
      // Sensor readings may also be up to 2 seconds 'old' (its a very slow sensor)
      float h = dht.readHumidity();
      // Read temperature as Celsius (the default)
      float t = dht.readTemperature();
      // Read temperature as Fahrenheit (isFahrenheit = true)
      float f = dht.readTemperature(true);

      // Check if any reads failed and exit early (to try again).
      if (isnan(h) || isnan(t) || isnan(f)) {
        Serial.println(F("Failed to read from DHT sensor!"));
        return;
      }

      // Compute heat index in Fahrenheit (the default)
      float hif = dht.computeHeatIndex(f, h);
      // Compute heat index in Celsius (isFahreheit = false)
      float hic = dht.computeHeatIndex(t, h, false);

      Serial.print(F("Humidity: "));
      Serial.print(h);
      Serial.print(F("%  Temperature: "));
      Serial.print(t);
      Serial.print(F("°C "));
      Serial.print(f);
      Serial.print(F("°F  Heat index: "));
      Serial.print(hic);
      Serial.print(F("°C "));
      Serial.print(hif);
      Serial.println(F("°F"));
    }
    ```
