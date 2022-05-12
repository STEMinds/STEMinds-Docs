---
title: Water liquid level sensor
description: No contact with liquid makes the module suitable for hazardous applications such as detecting toxic substances, strong acid, strong alkali and all kinds of liquid in an air tight container under high pressure. It's an industrial grade IP65 certified sensor that could be used for many industrial applications.
---

# Water liquid level sensor

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/water_liquid_level_sensor.jpg">
</p>

No contact with liquid makes the module suitable for hazardous applications such as detecting toxic substances, strong acid, strong alkali and all kinds of liquid in an air tight container under high pressure. It's an industrial grade IP65 certified sensor that could be used for many industrial applications.

## Specifications

Note that the sensor accepts 5V input and gives 5V output, in the Eduponics Mini kit we've added a voltage step-down from 5V to 3.3V, if you want to use the sensor for other applications that require microcontrollers you should consider adding a step-down circuit as well to prevent damage to your electronics.

Some of the cool features the sensor has:

* Red LED Indicator (have water LED is on, no water LED is off)

* Intelligent liquid level sensitivity adjustment

* High stability, high sensitivity, strong interference ability,

* Protection from external electromagnetic interference

* Strong compatibility, through a variety of non-metallic containers

* Sensing distance of around 12mm;

* Ability to detect liquid, powder or particles.

## Possible applications

Because the sensor is an external sensor, it allows us to monitor things without exposing the sensor to the water, some of the most useful applications could be:

* Monitor water level for general purpose
* Monitor if there is a flow of water in a pipe (water go through)
* Monitor drinking water in the coffee machine / pet bowl
* Monitor water level inside aquarium, refill if needed.
* Monitor dangerous chemicals (liquids)


## Connection diagram

The sensor can be attached to any non-metal surface which means plastic, glass, ceramic, rubber and more ...
The sensor sensitivity can be adjust so the sensor can both sense through thick and thin material, whatever your water container is - we got your back.
our sensor should be able to sense beyond most common materials.

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/water_quantity_sensor_connection1.jpg">
</p>

Another option would be to attach a silicon or glue on the front side of the sensor to attach it with a distance or directly to the plastic surface where metal surface is present.

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/water_quantity_sensor_connection2.jpg">
</p>

In our kit, we've included a special glue circles that can be glued on the sensor itself so it can be attached with ease to any surface.

!!! Warning "Metal surfaces note"
    Don't attach the sensor directly to metal surface, it won't work.
    it will work just fine with plastic, glass ceramic and more, for metal you'll need a plastic or other material connector between the two metal sides, look at the second picture for reference.

## Sensitivity adjustment

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/water_quantity_sensor_adjustment.jpg">
</p>

In some cases, sensitivity adjustment might be required. this is due to different material or different thickness of the water container. don't you worry, this can be easily done but taking off the cover where the indication LED is and softly using a screw driver to adjust the sensitivity of the sensor.
<br/><br/>
Rotate clockwise to reduce the sensitivity and to the opposite direction to increase it. you can use water inside of the container to see which one works best for you, when there is water the sensor LED should be turned on.

## Hardware explanation

The Non-contact type liquid level sensor, utilizes advance signal processing technology by using a powerful chip with high-speed operation capacity to achieve non-contact liquid level detection.
<br/><br/>
The sensor itself have 4 pins: GND, VCC, DATA and Natural that we don't connect.
<br/><br/>
The sensor requires 5V input to operate and it will output 5V as well, as the ESP32 IO pins can only withstand 3.3V over the IO pins we added voltage step down from 5V to 3.3V, this is something you need to remember in case you want to use the sensor with different hardware and different applications.

!!! Warning "5V to 3.3V step down is required for other usages"
    If you want to use the sensor with Arduino or Raspberry Pi in different projects, make sure to step the voltage down from 5V output to 3.3V or lower to prevent damage to the IO pins of your micro-controller.
    This step down circuit already included in Eduponics mini development board so you don't need to worry about it.

## Plugging-in the water quantity sensor

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/connecting_liquid_level_sensor.jpeg">
</p>

Plugging in the sensor is very straight forward, using the XH2.54 interface we can plug it right in. the sensor have 4 pins in total: GND, VCC, DATA and NO (natural pin that we connect to ground as well).
<br/><br/>
If the Eduponics Mini board is powered on using either DC12V or USB Type-C or both and the sensor isn't connected to any surface the LED on top of the sensor should glow RED, once it touch a liquid surface such as bottle of water with water inside the LED will turn off.

## Software explanation

The code is very straight forward, the water level sensor is connected to IO pin number 21.
we'll set this pin as input and create a function called is_empty() the sensor will return 0 if he can detect water and 1 if there is no water detected.
which means, the function is_empty() will return True if the water container is empty or False if the water container is full.
<br/><br/>
Finally, we'll use the function to print into the console if the state of our water container.

=== "MicroPython"
    ``` python linenums="1"
    import machine

    # define water level sensor as INPUT on IO pin number 21
    water_level = machine.Pin(21, machine.Pin.IN)

    # this function will return 0 if container have no water and 1 if it has water
    def is_empty():
        return water_level.value()

    # check if the water container is empty is not
    if(is_empty()):
        print("The water container is empty")
    else:
        print("The water container is full")
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    // set the water sensor on IO pin number 21
    const int water_sensor = 21;

    void setup() {
      Serial.begin(115200);
      // configure pump as output
      pinMode(water_sensor, INPUT);
    }

    int is_empty(){
      return digitalRead(water_sensor);
    }

    void loop() {
      // if digital read is 0 means no water
      // if digital read is 1 means there is water
      if(is_empty()){
        Serial.println("The water container is empty");
      }else{
        Serial.println("The water container is full");
      }
      // wait 100 miliseconds and check again
      delay(100);
    }
    ```
