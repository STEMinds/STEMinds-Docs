---
title: STEMinds - Eduponics Mini IoT Smart agriculture Kit
description: Eduponics mini is a smart agricultural IoT learning kit based on the ESP32, it can be used for: smart watering solution, smart garden, IoT learning and development. Eduponics mini supports the MicroPython programming language and can can help you learn to code in no time!
---

# STEMinds Eduponics Mini Kit

The Eduponics Mini is smart agriculture learning kit based on the ESP32 Microcontroller, it features many built in sensors as well as WiFi, Bluetooth and low power consumption support to enable real life IoT applications.

This page provides you the basic knowledge and tools to get started with your Eduponics Mini kit.

Usually, companies print this type of page as a small manual delivered together with the product. At STEMinds we care about the environment, and one of the ways we support this cause is by saving on unnecessary paper prints.

By having the basic manual and instructions here we can assure you that we will always keep it up to date and working.

## Eduponics Mini Development board

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/eduponics_mini.png">
</p>

Except for the sensors and modules included in the Eduponics mini kit, the main thing is our custom made Eduponics Mini Board.
The Eduponics Mini board is a custom printed circuit board (PCB) that consists of 2 layers with multiple connectors and sensors.

On the top side of the board you'll find:

* ESP32 Micro Controller (already pre-installed with MicroPython)
* Water quantity, soil moisture and pump XH2.54 interfaces
* Tiny relay that is used for controlling the pump
* ON/OFF Switch
* 12V DC Power interface
* USB Type-C programmable interface (UART)
* Reset Button
* Wake up button (can be used with deep sleep functionality)
* RTC DS1307 Module (coin-cell battery not included with the kit)
* BH1750 I2C Light sensor
* BME280 Temperature, humidity and barometric pressure sensor
* Extension for DHT11 / DHT22 sensor (DHT sensor not included)
* AT24C02 EEPROM chip
* IO Expansion pins to connect extra sensors or modules

## Unpacking the kit

Once you unpack the package, you will find it contains the following:

* Custom Eduponics Mini ESP32 development board
* 12V2A DC Power adapter
* USB Type-C data cable
* Soil moisture sensor
* 12V Submersible water pump
* Touch-less water quantity sensor
* Irrigation Water hose

All those words might sound like a foreign language to you but we can assure you that at the end of the course not only will you understand their meaning
but you'll also be able to control those sensors, program them and make your inventions completely by yourself!

## Open source software

All our code examples and drivers are completely open source, while our Eduponics mobile app is not open sourced for security reasons we do provide an MQTT library to control every aspect of the app remotely such as:

* Add and change sensors data
* Change plants name
* Water plants remotely
* Control an unlimited amount of plants

All the code is available in our git repository: [STEMinds Eduponics Mini repository](https://github.com/STEMinds/Eduponics-Mini)

## Eduponics Mobile app

<p align="left">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/eduponics_featured.png">
</p>

The Eduponics Mini kit can be integrated with Eduponics app that allows it to be controlled from anywhere in the world using the MQTT communication protocol.
This allows users and students to get sense of real life useful IoT applications that can be used in our daily lives.

## Programing language

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/micropython.jpg">
</p>

For this kit we've decided to use the MicroPython language which is based on Python3 but designed for microcontrollers.

Python is the most popular programming language that exists today, millions of people use it for automation, AI, websites and programs that they use in their daily life.

The Firmware we use is generic ESP32 firmware which allows you to program both in MicroPython using Thonny IDE or C++ using Arduino IDE.

We currently officially support MicroPython but we've also added most of the examples for Arduino IDE in C++ programming language, in our Github repository you might find other supported ESP32 firmwares that could allow you to program in many different programming languages and explore other ways to interact with the Eduponics Mini!
