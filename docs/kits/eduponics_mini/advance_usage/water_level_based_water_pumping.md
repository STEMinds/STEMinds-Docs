---
title: Water level and pump
description: Combining both functionalities - the pump and the the liquid sensor to determine whenever there is water or not then decide if we should allow or prevent water pumping.
---

# Water level activated pump

We've mentioned couple of times during our previous lessons that due to the characteristics of the pump - we cannot power it on outside of the water or it might get damaged.
<br/><br/>
Now imagine the following scenario: you control your pump while you are not at home to water your plants but you haven't combined the pump functionality with the water quantity sensor which results in the pump pumping water when no water is presented at all! That will eventually cause damage to the pump.
<br/><br/>
We are going to solve it by combining both functionalities - the pump and the the liquid sensor to determine whenever there is water or not then decide if we should allow or prevent water pumping.

## Getting the water level

First stage will be getting the amount of water from the liquid sensor, we did it earlier but let's do it once again. Using the following code we can tell if our container is empty or full and determine if it's safe to use the pump at any given time.

=== "MicroPython"
    ``` python linenums="1"
    import machine

    # define water level sensor as INPUT on IO pin number 21
    water_level = machine.Pin(21, machine.Pin.IN)

    def is_empty():
        # will return 0 if container have no water and 1 if it has water
        return water_level.value()

    if(is_empty()):
        print("The water container is empty")
    else:
        print("The water container is full")
    ```


## Activating the relay

Next step will be to activate the relay. we also learned it previously but let's take a look again at the code to make sure we know what we are doing.
<br/><br/>
Remember, the pump is powered by 12V DC adapter so make sure to plug it in when you want to use the pump functionality.

=== "MicroPython"
    ``` python linenums="1"
    import machine
    import time

    # define pump on pin IO23 as OUTPUT
    pump = machine.Pin(23, machine.Pin.OUT)

    # turn on the pump
    pump.value(1)
    # turn off the pump
    pump.value(0)
    ```

## Final program

Now when we revised previous learned lessons on how to power the pump and use the water quantity sensor - let's combine both functionalities to give water only if the bucket is not empty:

=== "MicroPython"
    ``` python linenums="1"
    import machine
    import time

    # define pump on pin IO23 as OUTPUT
    pump = machine.Pin(23, machine.Pin.OUT)

    def is_empty():
        return water_quantity.value()

    # define pump on pin IO23 as OUTPUT
    pump = machine.Pin(23, machine.Pin.OUT)
    # turn on the pump
    pump.value(1)
    while not is_empty():
        # not empty yet, keep giving water
        time.sleep(0.1)

    # oh no, now the bucket is empty, stop it!
    # turn off the pump
    pump.value(0)
    ```
