---
title: STEMinds - ESP32 Deep sleep functionality
description: Deep sleep mode will reduce the power consumption and your batteries will last longer. Having the ESP board in deep sleep mode means cutting with the activities that consume more power while operating but leave just enough activity to wake up the processor when something interesting happens.
---

# Wakeup from Deep sleep

<p align="center">
  <img src="/kits/eduponics_mini/images/drained_battery_illustration.jpg" width="350px">
</p>

Having the Eduponics Mini ESP32 board running on active mode with batteries it’s not ideal, since the power from batteries will drain very quickly.

If you put the board in deep sleep mode, it will reduce the power consumption and your batteries will last longer. Having the board in deep sleep mode means cutting with the activities that consume more power while operating but leave just enough activity to wake up the processor when something interesting happens.

When operating in deep sleep mode, the board have a current consumption on the μA range. With a custom and carefully designed board you can get a minimal consumption of about 5 μA.

This feature is very specific to the ESP32 other boards such as Arduino or Raspberry Pi don't have this deep sleep functionality thus not suitable for outdoor / battery powered applications.

## Wakeup sources

There are multiple ways that ESP32 can wakeup from a deep sleep, such as:

* Timer - using ESP32 internal RTC to count and wakeup after x seconds
* External wakeup - we'v dedicated a "wakeup" button on the Eduponics mini board for this purpose.
* touch pins - waking up by connecting touch button to the board

In our tutorial we'll focus on waking up by push of a button, this is useful if you want to check sensors data on the Eduponics mini board by pressing it from time to time let's say when you come home every day, this can be extremely beneficial for the battery life if you power the board with an external battery.

## Waking up by button press

We've integrated a special button on the top right side of the Eduponics mini board, it's just a simple button that is connected to IO PIN number 36, technically this button can be used for anything but we've decided to dedicate it for a special use which is waking up from deep sleep.

Before we go into deep sleep <b>we must</b> set a wake up input, failing to do so will cause our board to go into infinite loop where it wakes up and goes back to sleep.

We can do it by the function called .wake_on_ext0() supply the wakeup pin which is IO PIN 36 which we configured as input and wakeup any high means any type of button touch (LOW OR HIGH) the board will wakeup.

After configuring the button, it's safe to continue writing the software and when we ready to put the board to sleep we can use machine.deepsleep() to enter deep sleeping state, once the button is pressed the board will restart and initialize the program again.

=== "MicroPython"
    ``` python linenums="1"
    import esp32
    from machine import Pin
    from time import sleep

    wake_up = Pin(36, mode = Pin.IN)

    #level parameter can be: esp32.WAKEUP_ANY_HIGH or esp32.WAKEUP_ALL_LOW
    esp32.wake_on_ext0(pin = wake_up, level = esp32.WAKEUP_ANY_HIGH)

    #your main code goes here to perform a task
    print('Im awake now. Going to sleep in 5 seconds ...')
    sleep(5)
    print('Going to sleep now ..')
    #once the main code finished, we can set the program to enter deep-sleep
    machine.deepsleep()
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    /*
    Deep Sleep with External Wake Up
    =====================================
    This code displays how to use deep sleep with
    an external trigger as a wake up source and how
    to store data in RTC memory to use it over reboots

    This code is under Public Domain License.

    Hardware Connections
    ======================
    Push Button to GPIO 33 pulled down with a 10K Ohm
    resistor

    NOTE:
    ======
    Only RTC IO can be used as a source for external wake
    source. They are pins: 0,2,4,12-15,25-27,32-39.

    Author:
    Pranav Cherukupalli <cherukupallip@gmail.com>
    */

    #define BUTTON_PIN_BITMASK 0x200000000 // 2^33 in hex

    RTC_DATA_ATTR int bootCount = 0;

    /*
    Method to print the reason by which ESP32
    has been awaken from sleep
    */
    void print_wakeup_reason(){
      esp_sleep_wakeup_cause_t wakeup_reason;

      wakeup_reason = esp_sleep_get_wakeup_cause();

      switch(wakeup_reason)
      {
        case ESP_SLEEP_WAKEUP_EXT0 : Serial.println("Wakeup caused by external signal using RTC_IO"); break;
        case ESP_SLEEP_WAKEUP_EXT1 : Serial.println("Wakeup caused by external signal using RTC_CNTL"); break;
        case ESP_SLEEP_WAKEUP_TIMER : Serial.println("Wakeup caused by timer"); break;
        case ESP_SLEEP_WAKEUP_TOUCHPAD : Serial.println("Wakeup caused by touchpad"); break;
        case ESP_SLEEP_WAKEUP_ULP : Serial.println("Wakeup caused by ULP program"); break;
        default : Serial.printf("Wakeup was not caused by deep sleep: %d\n",wakeup_reason); break;
      }
    }

    void setup(){
      Serial.begin(115200);
      delay(1000); //Take some time to open up the Serial Monitor

      //Increment boot number and print it every reboot
      ++bootCount;
      Serial.println("Boot number: " + String(bootCount));

      //Print the wakeup reason for ESP32
      print_wakeup_reason();

      /*
      First we configure the wake up source
      We set our ESP32 to wake up for an external trigger.
      There are two types for ESP32, ext0 and ext1 .
      ext0 uses RTC_IO to wakeup thus requires RTC peripherals
      to be on while ext1 uses RTC Controller so doesnt need
      peripherals to be powered on.
      Note that using internal pullups/pulldowns also requires
      RTC peripherals to be turned on.
      */
      esp_sleep_enable_ext0_wakeup(GPIO_NUM_36,1); //1 = High, 0 = Low

      //If you were to use ext1, you would use it like
      //esp_sleep_enable_ext1_wakeup(BUTTON_PIN_BITMASK,ESP_EXT1_WAKEUP_ANY_HIGH);

      //Go to sleep now
      Serial.println("Going to sleep now");
      delay(1000);
      esp_deep_sleep_start();
      Serial.println("This will never be printed");
    }

    void loop(){
      //This is not going to be called
    }
    ```


!!! Warning "Must set button input or any wake up input before going to deep sleep"
    Failing to do so will cause infinite loop where when the board wakes up it goes back to sleep right away, it won't be possible to re-program it unless we'll clean the memory and write the firmware again.
