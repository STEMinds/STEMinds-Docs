---
title: Introduction to ESP32 MQTT
description: MQTT is a standard for moving data between an IoT device and a server. MQTT has become the de-facto IoT standard for connecting all sorts of IoT devices.
---

# Introduction to MQTT

<p align="center">
  <img src="/images/kits/eduponics_mini/mqtt_illustration.jpeg">
</p>

MQTT is a standard for moving data between an IoT device and a server. Originally developed in 1999 by Andy Standford-Clark and Arlen Nipper to monitor oil and gas pipelines over remote satellite connections, MQTT has become the de-facto IoT standard for connecting all sorts of IoT devices. Today, all major IoT platforms, IoT cloud services providers, and many IoT edge gateways and devices support connectivity with MQTT.

## What is MQTT

MQTT is a publish/subscribe protocol that is lightweight and requires a minimal footprint and bandwidth to connect an IoT device. Unlike HTTP’s request/response paradigm, MQTT is event driven and enables messages to be pushed to clients. This type of architecture decouples the clients from each other to enable a highly scalable solution without dependencies between data producers and data consumers.
<br/><br/>
When using HTTP/HTTPS you need to pull data at interval and check whenever there is new data waiting for you, the requests often contain headers and more info which make the task quite heavy for small IoT nodes that need a lightweight and fast solution.
<br/><br/>
By Using MQTT, an IoT device can subscribe to a channel or publish to it (or both) when publishing a message, all the subscribed devices will get it almost instantly.

## MQTT Benefits

* Lightweight and efficient to minimize resources required for the client and network bandwidth.
* Enables bidirectional communication between devices and servers. Also, enabling broadcasting messages to groups of things.
* Scales to millions of things (devices).
* MQTT specifies Quality of Service (QoS) levels to support message reliability.
* MQTT supports persistent sessions between device and server that reduces reconnection time required over unreliable networks.
* MQTT messages can be encrypted with TLS and support client authentication protocols.

## MQTT Vocabulary

When we talk about MQTT we need to clarify some vocabulary to make sure we all on the same page:

* MQTT - The protocol name we use to communicate between IoT devices.
* Message - When a device / server want to send packet of data, we call it a message.
* Topic - a URL where we subscribe / publish data to such as: sensors/light/
* Publish - When a device want to send a message, the action of sending a message called "Publish".
* Subscribe - When a device want to listen to new incoming messages, the action is called "Subscribe".
* Broker - The MQTT Server that handle the publish/subscribe transactions
* Client - The end device (any IoT device that we use to send or receive messages)
* QoS - Quality of service, a method to give a priority to messages (which one is more important to be received or sent first)

## MQTT useful applications

There are a lot of very useful and interesting applications that MQTT can be used for, particularly in the world of IoT. For example:

* Sending lightweight sensor data such as: temperature, humidity, light state etc ...
* Receiving sensors data and processing it
* Sending message to multiple (millions) of devices at the same time
* Receiving message in real time from a central server

If this applications don't tell you much, don't worry about it.
<br/><br/>
Later on in our tutorials we will focus on our Eduponics Mini kit and will learn how to send the sensors data from the ESP32 Eduponics mini board to our mobile app through a dedicated STEMinds broker we've prepared in advance.

## MQTT Software

<p align="center">
  <img src="/images/kits/eduponics_mini/MQTT_mosquitto_logo.png">
</p>

While MQTT is the name of the protocol itself there are huge variety of softwares we can use to implement the MQTT protocol into our code, the most popular one called "Mosquitto", many services and broker providers use this software and in result it has the highest community support.
<br/><br/>
[Eclipse Mosquitto](https://mosquitto.org/) is the freely available broker software we use to connect the Eduponics mini to the cloud and communicate with it through our dedicated Eduponics mobile app.

## MQTT protocol

<p align="center">
  <img src="/images/kits/eduponics_mini/MQTT_demo.jpeg">
</p>

The MQTT protocol is deadly simple to use, as we've mentioned there are 2 main things we need to remember: Subscribe & Publish.

### MQTT Publish

When we want to send data (it can be any data, such as: temperature and humidity, light, state etc ...) we use publish. The data we publish called "Message" and we publish it into a url like channel for example
<br/><br/>
Imagine we want to turn on the living light, we can publish message to sensors/living_room/ topic with the message '1':

    sensors/living_room/light/1

Any other device that is subscribed to /sensors/living_room/light will receive the state 1 or 0 and will be able to process commands from there.
<br/><br/>
There are couple of things to mind when designing topics, Here is a good article from HiveMQ on best practices when designing MQTT topics:

[MQTT Topics & Best Practices - MQTT Essentials](https://www.hivemq.com/blog/mqtt-essentials-part-5-mqtt-topics-best-practices/)

### MQTT Subscribe

The other thing to mind is the Subscribe functionality, when we want to receive message that been broadcast by other devices or the server itself through the MQTT network we need to subscribe to a topic. This is fairly easy as we don't need to send any data just receive it.

For example, we can subscribe to the following topic:

    sensors/living_room/light

Any device can post a message to that topic and we'll receive it almost instantly, then we can decide what to do with the information given. should we report to the database? should we give feedback? should we just ignore it? it all depends on the application we are trying to archive.

## Coming up next

In the next couple of lessons we'll build our first MQTT basic client and finally we'll connect our Eduponics Mini to the Eduponics Mobile APP for full functionality!
