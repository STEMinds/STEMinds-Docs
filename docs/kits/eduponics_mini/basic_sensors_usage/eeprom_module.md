---
title: STEMinds - AT24C02 EEPROM
description: Atmel AT24C02 EEPROM is a memory chip that allows you to store data and retain it even when the power goes off. It is usually used to store user settings or other kind of configuration. This great chip allows us to write bytes of information and read them later, no matter if the device rebooted itself or not.
---

# AT24C02 EEPROM Module

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/AT24C02_EEPROM.jpg">
</p>

The Atmel AT24C02 EEPROM is a memory chip that allows you to store data and retain it even when the power goes off. It is usually used to store user settings or other kind of configuration.
<br/><br/>
This great chip allows us to write bytes of information and read them later, no matter if the device rebooted itself or not.
the ESP32 has the capability to store files locally using Python but we thought that adding EEPROM could be very useful.

## Specifications

The EEPROM includes many great features such as 1 million write cycles and data retention of 100 years, but also:

* Low voltage operation
* I2C-Compatible (2-wire) serial interface
* Bidirectional data transfer protocol (read and write)
* Write Protect pin for full array hardware data protection
* Ultra low active current (1mA max) and standby current (0.8μA max)
* 8-byte Page Write mode, partial page writes are also allowed

View the complete datasheet: [AT24C02 EEPROM datasheet from octopart.com](https://datasheet.octopart.com/AT24C02D-XHM-T-Microchip-datasheet-25361987.pdf)

!!! Warning "1,000,000 write lifecycle usage limit"
    1,000,000 writes might sound like a lot and it's definitely is. but, in order to keep the EEPROM healthy we do not recommend writing on it using while or for loops in milliseconds of interval which means you should keep data when it's necessary and read it when necessary, use it wisely and you should have years of possible great usage with the AT24C02 EEPROM chip!
    Reading is not limited as much as writing.

## Possible applications

There could be many reasons to use EEPROM, such as:

* authentication, keep login information or unique data that you need to access on time when device is booting;
* things that you need to check if device rebooted (if it's rebooted, you'll lose variables that were in memory, that's where EEPROM is useful for);
* remember the last state of a variable;
* save settings;
* save how many times an appliance was activated;
* any other type of data that you need to have saved permanently.

## Software explanation

!!! Info "Make sure micropython-eduponics is installed through upip"
    For our Python code, we will need to import eduponics library, make sure you followed the introduction guide on installing the library on the ESP32 Eduponics Mini board.

The software library is written by Mike Causer, you can check the Github library [here](https://github.com/mcauser/micropython-tinyrtc)
<br/><br/>
In micropython-eduponics we have a class called AT24C32N where there are multiple functions such as read and write.
The first thing to do will be to initialize our EEPROM using the I2C protocol which has 2 pins SCL and SDA.
The SCL pin would be pin 15 while the SDA pin would be pin number 4.
<br/><br/>
Then, when we have our i2c object ready, we can initialize the EEPROM.
<br/><br/>
First, we try to read 32 bytes which should be empty. Then, we'll write "hello world" which takes 11 bytes.
finally, we'll read 11 bytes which is exactly the length of the string we've added which is "hello world".

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython TinyRTC I2C Module, DS1307 RTC + AT24C32N EEPROM
    https://github.com/mcauser/micropython-tinyrtc
    MIT License
    Copyright (c) 2018 Mike Causer
                  2021 STEMinds
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

    # AT24C32A, 32K (32768 kbit / 4 KB), 128 pages, 32 bytes per page, i2c addr 0x50
    from eduponics import at24c02
    import machine
    import time

    # initialize i2c connection
    i2c = machine.I2C(scl=machine.Pin(15), sda=machine.Pin(4))

    # define the EEPROM using the I2C
    eeprom = at24c02.AT24C32N(i2c)

    # print 32 bytes
    print(eeprom.read(0, 32))

    # write "hello world" starting from sector 0
    eeprom.write(0, 'hello world')

    # read 11 bytes from the EEPROM
    print(eeprom.read(0, 11))
    ```
=== "ESP32-Arduino"
    ``` C++ linenums="1"
    /*
      *  Use the I2C bus with EEPROM 24LC64
      *  Sketch:    eeprom.ino
      *
      *  Author: hkhijhe
      *  Date: 01/10/2010
      *
      *
      */

    #include <Wire.h>

    void i2c_eeprom_write_byte( int deviceaddress, unsigned int eeaddress, byte data ) {
        int rdata = data;
        Wire.beginTransmission(deviceaddress);
        Wire.write((int)(eeaddress >> 8)); // MSB
        Wire.write((int)(eeaddress & 0xFF)); // LSB
        Wire.write(rdata);
        Wire.endTransmission();
    }

    // WARNING: address is a page address, 6-bit end will wrap around
    // also, data can be maximum of about 30 bytes, because the Wire library has a buffer of 32 bytes
    void i2c_eeprom_write_page( int deviceaddress, unsigned int eeaddresspage, byte* data, byte length ) {
        Wire.beginTransmission(deviceaddress);
        Wire.write((int)(eeaddresspage >> 8)); // MSB
        Wire.write((int)(eeaddresspage & 0xFF)); // LSB
        byte c;
        for ( c = 0; c < length; c++)
            Wire.write(data[c]);
        Wire.endTransmission();
    }

    byte i2c_eeprom_read_byte( int deviceaddress, unsigned int eeaddress ) {
        byte rdata = 0xFF;
        Wire.beginTransmission(deviceaddress);
        Wire.write((int)(eeaddress >> 8)); // MSB
        Wire.write((int)(eeaddress & 0xFF)); // LSB
        Wire.endTransmission();
        Wire.requestFrom(deviceaddress,1);
        if (Wire.available()) rdata = Wire.read();
        return rdata;
    }

    // maybe let's not read more than 30 or 32 bytes at a time!
    void i2c_eeprom_read_buffer( int deviceaddress, unsigned int eeaddress, byte *buffer, int length ) {
        Wire.beginTransmission(deviceaddress);
        Wire.write((int)(eeaddress >> 8)); // MSB
        Wire.write((int)(eeaddress & 0xFF)); // LSB
        Wire.endTransmission();
        Wire.requestFrom(deviceaddress,length);
        int c = 0;
        for ( c = 0; c < length; c++ )
            if (Wire.available()) buffer[c] = Wire.read();
    }

    void setup()
    {
        char somedata[] = "hello world"; // data to write
        // start I2C on pins 4 and 15
        Wire.begin(4,15);
        // start serial connection
        Serial.begin(115200);
        i2c_eeprom_write_page(0x50, 0, (byte *)somedata, sizeof(somedata)); // write to EEPROM
        delay(100); //add a small delay
        Serial.println("Memory written");
    }

    void loop()
    {
        int addr=0; //first address
        byte b = i2c_eeprom_read_byte(0x50, 0); // access the first address from the memory

        while (b!=0)
        {
            Serial.print((char)b); //print content to serial port
            addr++; //increase address
            b = i2c_eeprom_read_byte(0x50, addr); //access an address from the memory
        }
        Serial.println(" ");
        delay(2000);
    }
    ```
