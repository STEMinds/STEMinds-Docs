---
title: STEMinds - DS1307Z Real time clock (RTC) Module
description: The DS1307 Serial Real-Time Clock is a low-power, full binary-coded decimal (BCD) clock/calendar plus 56 bytes of NV SRAM. Address and data are transferred serially via a 2-wire, bi-directional bus. The clock/calendar provides seconds, minutes, hours, day, date, month, and year information.
---

#DS1307Z RTC Module

The DS1307 Serial Real-Time Clock is a low-power, full binary-coded decimal (BCD) clock/calendar
plus 56 bytes of NV SRAM. Address and data are transferred serially via a 2-wire, bi-directional bus.
The clock/calendar provides seconds, minutes, hours, day, date, month, and year information. The end of
the month date is automatically adjusted for months with fewer than 31 days, including corrections for
leap year.
<br/><br/>
The clock operates in either the 24-hour or 12-hour format with AM/PM indicator. The
DS1307 has a built-in power sense circuit that detects power failures and automatically switches to the
battery supply.
<br/><br/>
While the ESP32 have a built-in RTC inside, the RTC inside of the ESP32 is mostly used for counting (like a timer) but what if we need to know the exact time of the day or the date when our ESP32 wakeup in order to figure out if it's time to do something? for this purpose we've added extra component - our DS1307Z RTC module.
<br/><br/>
The Internal ESP32 RTC can be connected to a battery but this coin battery will power the entire ESP32 board and not just the RTC functionality which will force the ESP32 to constantly be in sleep mode in order not to waste energy which is something unreasonable to do in case of our application.

!!! Warning "RTC Lithium coin cell battery not included in Eduponics Mini kit."
    Due to shipping regulations and policies we are not able to ship the lithium batteries inside the kit,
    the RTC module requires the coin cell battery to operate and save data even when the ESP32 is off.

## Specifications

The DS1307Z RTC includes bunch of useful features, such as:

* Real-time clock (RTC) counts seconds, minutes, hours, date of the month, month, day of the week, and year with leap-year
* compensation valid up to 2100 56-byte, battery-backed, nonvolatile (NV)
* RAM for data storage Two-wire serial interface
* Programmable squarewave output signal
* Automatic power-fail detect and switch circuitry
* Consumes less than 500nA in battery backup mode with oscillator running

For the complete data sheet check here: [DS1307 data sheet from sparkfun.com](https://www.sparkfun.com/datasheets/Components/DS1307.pdf)

## Possible applications

The RTC module is super useful, here are some of the possible applications you can archive using it:

* Store data and time to refer to without wireless connectivity
* Use the stored date and time to check if it's "time" to perform scheduled operation
* Use time and date for logging, if something happens you will know when it did.
* Any other application that require time or date to be pulled in no time.

## Hardware explanation

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/RTC_ DS1307_schematic.jpg">
</p>

The simple circuit the two inputs X1 and X2 are connected to a 32.768 kHz crystal oscillator as the source of the chip. VBAT is connected to positive culture of a 3V battery chip. VCC power to the I2C interface is 3.3V which is given by the ESP32 board. If the power supply VCC is not granted read and writes are inhibited.
<br/><br/>
CN1 represents the IO connections for the I2C interfae which we use to control the module and read/write from it.
<br/><br/>
On the Eduponics Mini board the RTC module and the crystal oscillator is at the front while the coin battery cell is mounted at the back.


## Connecting the battery to the RTC module

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/3V-Coin-Cell-Battery-CR1220.jpg">
</p>

The RTC coin battery is <b>not included</b> with the kit due to logistics concerns when shipping Lithium batteries, The battery type is CR1220 and can be found in probably any convenient shop near your home. The picture above is for illustration purpose only, there are many brands such as Panasonic, Murata, Cellewell etc ... all should work as long as it's 3V and same size (the model should be CR1220).

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/eduponics_mini_plugging_rtc.png">
</p>

The battery case can be found on the bottom side of the Eduponics Mini development board.
Make sure to plug the battery in the right direction (ground bottom side, positive side up).
<br/><br/>
Without the coin cell battery the RTC functionality will work but we won't be able to save any data after our device reboots or disconnect from power.

## Software explanation

!!! Info "Make sure micropython-eduponics is installed through upip"
    For our Python code, we will need to import eduponics library, make sure you followed the introduction guide on installing the library on the ESP32 Eduponics Mini board.

The software library is written by Mike Causer, you can check the Github library [here](https://github.com/mcauser/micropython-tinyrtc)
<br/><br/>
The RTC communicates through I2C protocol that has multiple address for multiple functions such as halt (power on/off) register time and date and read data (control register).
<br/><br/>
First, we'll need to define the I2C device: The RTC connected to SCL IO PIN 15 and SDA IO PIN  4.
After we've configured that successfully we can call the .halt() function with the argument "False" which means don't halt the device or in other words - turn it on.
<br/><br/>
Then, we need to set the timestamp (current time and date for our device to remember) this can be done only once or if we need to change it later on to something different we can repeat it as many times as we want but that won't be necessary in most applications.
<br/><br/>
A good resource to get exact time and date would be [epochconverter.com](https://www.epochconverter.com/) the last 2 zeroes in the time and date format can be left zeroed.
<br/><br/>
Finally, we can call .datetime() command to get the current time and date from the RTC module, it will update in real time even if we reboot or disconnect our device (as long as the coin battery lives)

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython DS1307 RTC module
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

    import machine
    from Eduponics import ds1307
    import time

    # configure i2c pin
    i2c = machine.I2C(scl=machine.Pin(15), sda=machine.Pin(4))
    # configure the RTC using the i2c configuration
    ds = ds1307.DS1307(i2c)
    # power on the chip
    ds.halt(False)
    # set the current time (change it to the time you need)
    now = (2020, 9, 7, 6, 14, 28, 0, 0)
    # print current date and time
    while True:
        print(ds.datetime())
        time.sleep(1)
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    #include "Wire.h"
    #define DS3231_I2C_ADDRESS 0x68
    // Convert normal decimal numbers to binary coded decimal
    byte decToBcd(byte val)
    {
      return( (val/10*16) + (val%10) );
    }
    // Convert binary coded decimal to normal decimal numbers
    byte bcdToDec(byte val)
    {
      return( (val/16*10) + (val%16) );
    }
    void setup()
    {
      // start I2C on pins 4 and 15
      Wire.begin(4,15);
      Serial.begin(115200);
      // set the initial time here:
      // DS3231 seconds, minutes, hours, day, date, month, year
      // setDS3231time(20,11,18,7,24,10,20);
    }
    void setDS3231time(byte second, byte minute, byte hour, byte dayOfWeek, byte
    dayOfMonth, byte month, byte year)
    {
      // sets time and date data to DS3231
      Wire.beginTransmission(DS3231_I2C_ADDRESS);
      Wire.write(0); // set next input to start at the seconds register
      Wire.write(decToBcd(second)); // set seconds
      Wire.write(decToBcd(minute)); // set minutes
      Wire.write(decToBcd(hour)); // set hours
      Wire.write(decToBcd(dayOfWeek)); // set day of week (1=Sunday, 7=Saturday)
      Wire.write(decToBcd(dayOfMonth)); // set date (1 to 31)
      Wire.write(decToBcd(month)); // set month
      Wire.write(decToBcd(year)); // set year (0 to 99)
      Wire.endTransmission();
    }
    void readDS3231time(byte *second,
    byte *minute,
    byte *hour,
    byte *dayOfWeek,
    byte *dayOfMonth,
    byte *month,
    byte *year)
    {
      Wire.beginTransmission(DS3231_I2C_ADDRESS);
      Wire.write(0); // set DS3231 register pointer to 00h
      Wire.endTransmission();
      Wire.requestFrom(DS3231_I2C_ADDRESS, 7);
      // request seven bytes of data from DS3231 starting from register 00h
      *second = bcdToDec(Wire.read() & 0x7f);
      *minute = bcdToDec(Wire.read());
      *hour = bcdToDec(Wire.read() & 0x3f);
      *dayOfWeek = bcdToDec(Wire.read());
      *dayOfMonth = bcdToDec(Wire.read());
      *month = bcdToDec(Wire.read());
      *year = bcdToDec(Wire.read());
    }
    void displayTime()
    {
      byte second, minute, hour, dayOfWeek, dayOfMonth, month, year;
      // retrieve data from DS3231
      readDS3231time(&second, &minute, &hour, &dayOfWeek, &dayOfMonth, &month,
      &year);
      // send it to the serial monitor
      Serial.print(hour, DEC);
      // convert the byte variable to a decimal number when displayed
      Serial.print(":");
      if (minute<10)
      {
        Serial.print("0");
      }
      Serial.print(minute, DEC);
      Serial.print(":");
      if (second<10)
      {
        Serial.print("0");
      }
      Serial.print(second, DEC);
      Serial.print(" ");
      Serial.print(dayOfMonth, DEC);
      Serial.print("/");
      Serial.print(month, DEC);
      Serial.print("/");
      Serial.print(year, DEC);
      Serial.print(" Day of week: ");
      switch(dayOfWeek){
      case 1:
        Serial.println("Sunday");
        break;
      case 2:
        Serial.println("Monday");
        break;
      case 3:
        Serial.println("Tuesday");
        break;
      case 4:
        Serial.println("Wednesday");
        break;
      case 5:
        Serial.println("Thursday");
        break;
      case 6:
        Serial.println("Friday");
        break;
      case 7:
        Serial.println("Saturday");
        break;
      }
    }
    void loop()
    {
      displayTime(); // display the real-time clock data on the Serial Monitor,
      delay(1000); // every second
    }
    ```
