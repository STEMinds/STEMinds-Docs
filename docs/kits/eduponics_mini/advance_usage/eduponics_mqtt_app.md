---
title: STEMinds - Eduponics APP
description: Learn how to integrate the Eduponics mini kit with Eduponics app to create a remote controlled smart garden, IoT irrigation system or smart watering solution
---

# Eduponics MQTT Powered Mobile APP

<p align="left">
  <img src="/kits/eduponics_mini/images/eduponics_featured.png">
</p>

The Eduponics App is a special app we've developed in order to turn your kit into a complete autonomous smart watering solution.

By using the app and pairing it with the Eduponics mini kit you'll be able to control your plants remotely by pumping water when needed, view sensors data in real time such as: temperature and humidity, light intensity, water quantity and off course monitoring soil moisture levels.

The app can work anywhere, whenever you are at home, at work or on vocation. it doesn't require a local server or port forwarding.

The app currently support the following languages:

English, Hebrew, Mandarin (Chinese), Spanish, German, Ukrainian, Russian, Hindi, Portuguese

The language is selected by your system language (your mobile phone language). If you'd like us to add extra translations and would like to help us translate the app for more languages, feel free to contact us!

!!! Tip "For now, only Android app version is available"
      Although we developed both the Android and iOS version for the Eduponics app, we haven't released the iOS app yet for the apple appstore due to app regulations and application approval process, we plan to release it as soon as possible, stay tuned!

## Preparing the Eduponics Mini

Before we'll jump directly into our app, we need to prepare our Eduponics Mini ESP32 board first with custom software that will enable our kit to connect to the MQTT broker and publish and subscribe to and from topics.

### Connecting Eduponics Mini to WiFi

Let's repeat the same process we've used in the basic MQTT client tutorial by creating <b>boot.py</b> file, this file will first load when our Eduponics Mini restart or the power plugged in, in this file we'll configure the WiFi credentials such as ESSID (WiFi name) and the WiFi password.

Once we've connected to the WiFi using the station.connect() command, we can print our ESP32 WiFi IP address into the console.

=== "MicroPython"
    ``` python linenums="1"
    # This code should be written into boot.py file on your Eduponics Mini board.
    from umqttsimple import MQTTClient
    import network
    import esp
    esp.osdebug(None)
    import gc
    gc.collect()

    ssid = 'WIFI_NAME'
    password = 'WIFI_PASSWORD'

    station = network.WLAN(network.STA_IF)

    station.active(True)
    station.connect(ssid, password)

    while station.isconnected() == False:
      pass

    print('Connected to WiFi successfully, IP: %s' % station.ifconfig()[0])
    ```

Now every time we restart or power the Eduponics Mini board it will automatically connect to the WiFi at our home.

!!! Warning "2.54GhZ WiFi support only"
    Just reminding once again, the ESP32 support only 2.54GhZ WiFi networks. Most of the 5Ghz WiFi routers / access points allow both 5Ghz and 2.54Ghz, make sure to choose the 2.54Ghz one. to avoid connectivity issues make sure your network is operated on 2.54Ghz.


### Adding uMQTTSimple class

As previously mentioned we'll need the umqttsimple client to communicate with the MQTT broker and use functionalities such as publishing data and subscribing to topics.

We should save this python code into file we'll call <b>umqttsimple.py</b> and we will import it using the import command every time we want to use the MQTT functionalities to communicate through the MQTT network.

=== "MicroPython"
    ``` python linenums="1"
    import usocket as socket
    import machine
    import ustruct as struct
    from ubinascii import hexlify

    class MQTTException(Exception):
        pass

    class MQTTClient:

        def __init__(self, client_id=hexlify(machine.unique_id()), server="mqtt.eclipse.org", port=0, user=None, password=None, keepalive=0,
                     ssl=False, ssl_params={}):
            if port == 0:
                port = 8883 if ssl else 1883
            self.client_id = client_id
            self.sock = None
            self.server = server
            self.port = port
            self.ssl = ssl
            self.ssl_params = ssl_params
            self.pid = 0
            self.cb = None
            self.user = user
            self.pswd = password
            self.keepalive = keepalive
            self.lw_topic = None
            self.lw_msg = None
            self.lw_qos = 0
            self.lw_retain = False

        def _send_str(self, s):
            self.sock.write(struct.pack("!H", len(s)))
            self.sock.write(s)

        def _recv_len(self):
            n = 0
            sh = 0
            while 1:
                b = self.sock.read(1)[0]
                n |= (b & 0x7f) << sh
                if not b & 0x80:
                    return n
                sh += 7

        def set_callback(self, f):
            self.cb = f

        def set_last_will(self, topic, msg, retain=False, qos=0):
            assert 0 <= qos <= 2
            assert topic
            self.lw_topic = topic
            self.lw_msg = msg
            self.lw_qos = qos
            self.lw_retain = retain

        def connect(self, clean_session=True):
            self.sock = socket.socket()
            addr = socket.getaddrinfo(self.server, self.port)[0][-1]
            self.sock.connect(addr)
            if self.ssl:
                import ussl
                self.sock = ussl.wrap_socket(self.sock, **self.ssl_params)
            premsg = bytearray(b"\x10\0\0\0\0\0")
            msg = bytearray(b"\x04MQTT\x04\x02\0\0")

            sz = 10 + 2 + len(self.client_id)
            msg[6] = clean_session << 1
            if self.user is not None:
                sz += 2 + len(self.user) + 2 + len(self.pswd)
                msg[6] |= 0xC0
            if self.keepalive:
                assert self.keepalive < 65536
                msg[7] |= self.keepalive >> 8
                msg[8] |= self.keepalive & 0x00FF
            if self.lw_topic:
                sz += 2 + len(self.lw_topic) + 2 + len(self.lw_msg)
                msg[6] |= 0x4 | (self.lw_qos & 0x1) << 3 | (self.lw_qos & 0x2) << 3
                msg[6] |= self.lw_retain << 5

            i = 1
            while sz > 0x7f:
                premsg[i] = (sz & 0x7f) | 0x80
                sz >>= 7
                i += 1
            premsg[i] = sz

            self.sock.write(premsg, i + 2)
            self.sock.write(msg)
            #print(hex(len(msg)), hexlify(msg, ":"))
            self._send_str(self.client_id)
            if self.lw_topic:
                self._send_str(self.lw_topic)
                self._send_str(self.lw_msg)
            if self.user is not None:
                self._send_str(self.user)
                self._send_str(self.pswd)
            resp = self.sock.read(4)
            assert resp[0] == 0x20 and resp[1] == 0x02
            if resp[3] != 0:
                raise MQTTException(resp[3])
            return resp[2] & 1

        def disconnect(self):
            self.sock.write(b"\xe0\0")
            self.sock.close()

        def ping(self):
            self.sock.write(b"\xc0\0")

        def publish(self, topic, msg, retain=False, qos=0):
            pkt = bytearray(b"\x30\0\0\0")
            pkt[0] |= qos << 1 | retain
            sz = 2 + len(topic) + len(msg)
            if qos > 0:
                sz += 2
            assert sz < 2097152
            i = 1
            while sz > 0x7f:
                pkt[i] = (sz & 0x7f) | 0x80
                sz >>= 7
                i += 1
            pkt[i] = sz
            #print(hex(len(pkt)), hexlify(pkt, ":"))
            self.sock.write(pkt, i + 1)
            self._send_str(topic)
            if qos > 0:
                self.pid += 1
                pid = self.pid
                struct.pack_into("!H", pkt, 0, pid)
                self.sock.write(pkt, 2)
            self.sock.write(msg)
            if qos == 1:
                while 1:
                    op = self.wait_msg()
                    if op == 0x40:
                        sz = self.sock.read(1)
                        assert sz == b"\x02"
                        rcv_pid = self.sock.read(2)
                        rcv_pid = rcv_pid[0] << 8 | rcv_pid[1]
                        if pid == rcv_pid:
                            return
            elif qos == 2:
                assert 0

        def subscribe(self, topic, qos=0):
            assert self.cb is not None, "Subscribe callback is not set"
            pkt = bytearray(b"\x82\0\0\0")
            self.pid += 1
            struct.pack_into("!BH", pkt, 1, 2 + 2 + len(topic) + 1, self.pid)
            #print(hex(len(pkt)), hexlify(pkt, ":"))
            self.sock.write(pkt)
            self._send_str(topic)
            self.sock.write(qos.to_bytes(1, "little"))
            while 1:
                op = self.wait_msg()
                if op == 0x90:
                    resp = self.sock.read(4)
                    #print(resp)
                    assert resp[1] == pkt[2] and resp[2] == pkt[3]
                    if resp[3] == 0x80:
                        raise MQTTException(resp[3])
                    return

        # Wait for a single incoming MQTT message and process it.
        # Subscribed messages are delivered to a callback previously
        # set by .set_callback() method. Other (internal) MQTT
        # messages processed internally.
        def wait_msg(self):
            res = self.sock.read(1)
            self.sock.setblocking(True)
            if res is None:
                return None
            if res == b"":
                raise OSError(-1)
            if res == b"\xd0":  # PINGRESP
                sz = self.sock.read(1)[0]
                assert sz == 0
                return None
            op = res[0]
            if op & 0xf0 != 0x30:
                return op
            sz = self._recv_len()
            topic_len = self.sock.read(2)
            topic_len = (topic_len[0] << 8) | topic_len[1]
            topic = self.sock.read(topic_len)
            sz -= topic_len + 2
            if op & 6:
                pid = self.sock.read(2)
                pid = pid[0] << 8 | pid[1]
                sz -= 2
            msg = self.sock.read(sz)
            self.cb(topic, msg)
            if op & 6 == 2:
                pkt = bytearray(b"\x40\x02\0\0")
                struct.pack_into("!H", pkt, 2, pid)
                self.sock.write(pkt)
            elif op & 6 == 4:
                assert 0

        # Checks whether a pending message from server is available.
        # If not, returns immediately with None. Otherwise, does
        # the same processing as wait_msg.

        def check_msg(self):
            self.sock.setblocking(False)
            return self.wait_msg()
    ```

### Generating UUID

As we've mentioned in the basic MQTT client tutorial, we could either head to [uuidgenerator.net](https://www.uuidgenerator.net/) and copy paste the automatically generated UUID for us or generate it by ourselves.

The UUID is used to identify our device from other devices on the MQTT network.

<b>it is crucial to generate a unique UUID and not re-use it</b>.

If we want to use the second option, here is a MicroPython example code to generate unique UUID, there are few kinds: unique based on host ID and current timestamp which is what we recommend, a UUID based on MD5 hash of a namespace (if you use this method, make sure to change steminds.com to something else, something random) and the last option to generate a random UUID.

=== "MicroPython"
    ``` python linenums="1"
    import uuid

    # make a UUID based on the host ID and current time, best option.
    uuid_x = uuid.uuid1()
    print(uuid_x)

    # make a UUID using an MD5 hash of a namespace UUID and a name
    uuid_y = uuid.uuid3(uuid.NAMESPACE_DNS, 'steminds.com')
    print(uuid_y)

    # make a random UUID
    uuid_z = uuid.uuid4()
    print(uuid_z)
    ```
### Adding dependencies

In our MQTT client we'll need to use sensors such as BME280, BH1750 light sensor and more. The main code for this sensors can be long and messy, it will be frustrating to create seperate files such as bme280.py and bh1750.py so we've added all those different classes into one file we call <b>dependencies.py</b>

Make sure to save it with the same name and we will use this file to import the BME280 temperature, humidity and air pressure sensor library as well as the BH1750 light sensor library.

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython TinyRTC I2C Module, DS1307 RTC + AT24C32N EEPROM
    https://github.com/mcauser/micropython-tinyrtc
    MIT License
    Copyright (c) 2018 Mike Causer

    MicroPython BH1750 light sensor module
    https://github.com/STEMinds/eduponics-mini
    MIT License
    Copyright (c) 2020 STEMinds

    BME280 Library Final Document: BST-BME280-DS002-15
    Authors: Paul Cunnane 2016, Peter Dahlebrg 2016
    Module borrows from the Adafruit BME280 Python library. Original Copyright notices are reproduced below.
    Those libraries were written for the Raspberry Pi.
    Copyright (c) 2014 Adafruit Industries
    Author: Tony DiCola
    Based on the BMP280 driver with BME280 changes provided by
    David J Taylor, Edinburgh (www.satsignal.eu)
    Based on Adafruit_I2C.py created by Kevin Townsend.

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
    import time
    from ustruct import unpack
    from array import array
    from micropython import const

    DATETIME_REG = const(0) # 0x00-0x06
    CHIP_HALT    = const(128)
    CONTROL_REG  = const(7) # 0x07
    RAM_REG      = const(8) # 0x08-0x3F

    # BME280 default address.
    BME280_I2CADDR = 0x76

    # Operating Modes
    BME280_OSAMPLE_1 = 1
    BME280_OSAMPLE_2 = 2
    BME280_OSAMPLE_4 = 3
    BME280_OSAMPLE_8 = 4
    BME280_OSAMPLE_16 = 5

    BME280_REGISTER_CONTROL_HUM = 0xF2
    BME280_REGISTER_STATUS = 0xF3
    BME280_REGISTER_CONTROL = 0xF4

    MODE_SLEEP = const(0)
    MODE_FORCED = const(1)
    MODE_NORMAL = const(3)

    class AT24C32N(object):
        """Driver for the AT24C32N 32K EEPROM."""

        def __init__(self, i2c, i2c_addr=0x50, pages=128, bpp=32):
            self.i2c = i2c
            self.i2c_addr = i2c_addr
            self.pages = pages
            self.bpp = bpp # bytes per page

        def capacity(self):
            """Storage capacity in bytes"""
            return self.pages * self.bpp

        def read(self, addr, nbytes):
            """Read one or more bytes from the EEPROM starting from a specific address"""
            return self.i2c.readfrom_mem(self.i2c_addr, addr, nbytes, addrsize=16)

        def write(self, addr, buf):
            """Write one or more bytes to the EEPROM starting from a specific address"""
            offset = addr % self.bpp
            partial = 0
            # partial page write
            if offset > 0:
                partial = self.bpp - offset
                self.i2c.writeto_mem(self.i2c_addr, addr, buf[0:partial], addrsize=16)
                time.sleep_ms(5)
                addr += partial
            # full page write
            for i in range(partial, len(buf), self.bpp):
                self.i2c.writeto_mem(self.i2c_addr, addr+i-partial, buf[i:i+self.bpp], addrsize=16)
                time.sleep_ms(5)

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

    class BME280:

        def __init__(self,mode=BME280_OSAMPLE_8,address=BME280_I2CADDR,i2c=None,**kwargs):
            # Check that mode is valid.
            if mode not in [BME280_OSAMPLE_1, BME280_OSAMPLE_2, BME280_OSAMPLE_4,
                            BME280_OSAMPLE_8, BME280_OSAMPLE_16]:
                raise ValueError(
                    'Unexpected mode value {0}. Set mode to one of '
                    'BME280_OSAMPLE_1, BME280_OSAMPLE_2, BME280_OSAMPLE_4,'
                    'BME280_OSAMPLE_8, BME280_OSAMPLE_16'.format(mode))
            self._mode = mode
            self.address = address
            if i2c is None:
                raise ValueError('An I2C object is required.')
            self.i2c = i2c
            self.__sealevel = 101325

            # load calibration data
            dig_88_a1 = self.i2c.readfrom_mem(self.address, 0x88, 26)
            dig_e1_e7 = self.i2c.readfrom_mem(self.address, 0xE1, 7)

            self.dig_T1, self.dig_T2, self.dig_T3, self.dig_P1, \
                self.dig_P2, self.dig_P3, self.dig_P4, self.dig_P5, \
                self.dig_P6, self.dig_P7, self.dig_P8, self.dig_P9, \
                _, self.dig_H1 = unpack("<HhhHhhhhhhhhBB", dig_88_a1)

            self.dig_H2, self.dig_H3, self.dig_H4,\
                self.dig_H5, self.dig_H6 = unpack("<hBbhb", dig_e1_e7)
            # unfold H4, H5, keeping care of a potential sign
            self.dig_H4 = (self.dig_H4 * 16) + (self.dig_H5 & 0xF)
            self.dig_H5 //= 16

            # temporary data holders which stay allocated
            self._l1_barray = bytearray(1)
            self._l8_barray = bytearray(8)
            self._l3_resultarray = array("i", [0, 0, 0])

            self._l1_barray[0] = self._mode << 5 | self._mode << 2 | MODE_SLEEP
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL,
                                 self._l1_barray)
            self.t_fine = 0

        def read_raw_data(self, result):
            """ Reads the raw (uncompensated) data from the sensor.
                Args:
                    result: array of length 3 or alike where the result will be
                    stored, in temperature, pressure, humidity order
                Returns:
                    None
            """

            self._l1_barray[0] = self._mode
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL_HUM,
                                 self._l1_barray)
            self._l1_barray[0] = self._mode << 5 | self._mode << 2 | MODE_FORCED
            self.i2c.writeto_mem(self.address, BME280_REGISTER_CONTROL,
                                 self._l1_barray)

            # Wait for conversion to complete
            while self.i2c.readfrom_mem(self.address, BME280_REGISTER_STATUS, 1)[0] & 0x08:
                time.sleep_ms(5)

            # burst readout from 0xF7 to 0xFE, recommended by datasheet
            self.i2c.readfrom_mem_into(self.address, 0xF7, self._l8_barray)
            readout = self._l8_barray
            # pressure(0xF7): ((msb << 16) | (lsb << 8) | xlsb) >> 4
            raw_press = ((readout[0] << 16) | (readout[1] << 8) | readout[2]) >> 4
            # temperature(0xFA): ((msb << 16) | (lsb << 8) | xlsb) >> 4
            raw_temp = ((readout[3] << 16) | (readout[4] << 8) | readout[5]) >> 4
            # humidity(0xFD): (msb << 8) | lsb
            raw_hum = (readout[6] << 8) | readout[7]

            result[0] = raw_temp
            result[1] = raw_press
            result[2] = raw_hum

        def read_compensated_data(self, result=None):
            """ Reads the data from the sensor and returns the compensated data.
                Args:
                    result: array of length 3 or alike where the result will be
                    stored, in temperature, pressure, humidity order. You may use
                    this to read out the sensor without allocating heap memory
                Returns:
                    array with temperature, pressure, humidity. Will be the one
                    from the result parameter if not None
            """
            self.read_raw_data(self._l3_resultarray)
            raw_temp, raw_press, raw_hum = self._l3_resultarray
            # temperature
            var1 = (raw_temp/16384.0 - self.dig_T1/1024.0) * self.dig_T2
            var2 = raw_temp/131072.0 - self.dig_T1/8192.0
            var2 = var2 * var2 * self.dig_T3
            self.t_fine = int(var1 + var2)
            temp = (var1 + var2) / 5120.0
            temp = max(-40, min(85, temp))

            # pressure
            var1 = (self.t_fine/2.0) - 64000.0
            var2 = var1 * var1 * self.dig_P6 / 32768.0 + var1 * self.dig_P5 * 2.0
            var2 = (var2 / 4.0) + (self.dig_P4 * 65536.0)
            var1 = (self.dig_P3 * var1 * var1 / 524288.0 + self.dig_P2 * var1) / 524288.0
            var1 = (1.0 + var1 / 32768.0) * self.dig_P1
            if (var1 == 0.0):
                pressure = 30000  # avoid exception caused by division by zero
            else:
                p = ((1048576.0 - raw_press) - (var2 / 4096.0)) * 6250.0 / var1
                var1 = self.dig_P9 * p * p / 2147483648.0
                var2 = p * self.dig_P8 / 32768.0
                pressure = p + (var1 + var2 + self.dig_P7) / 16.0
                pressure = max(30000, min(110000, pressure))

            # humidity
            h = (self.t_fine - 76800.0)
            h = ((raw_hum - (self.dig_H4 * 64.0 + self.dig_H5 / 16384.0 * h)) *
                 (self.dig_H2 / 65536.0 * (1.0 + self.dig_H6 / 67108864.0 * h *
                                           (1.0 + self.dig_H3 / 67108864.0 * h))))
            humidity = h * (1.0 - self.dig_H1 * h / 524288.0)
            # humidity = max(0, min(100, humidity))

            if result:
                result[0] = temp
                result[1] = pressure
                result[2] = humidity
                return result

            return array("f", (temp, pressure, humidity))

        @property
        def sealevel(self):
            return self.__sealevel

        @sealevel.setter
        def sealevel(self, value):
            if 30000 < value < 120000:  # just ensure some reasonable value
                self.__sealevel = value

        @property
        def altitude(self):
            '''
            Altitude in m.
            '''
            from math import pow
            try:
                p = 44330 * (1.0 - pow(self.read_compensated_data()[1] /
                                       self.__sealevel, 0.1903))
            except:
                p = 0.0
            return p

        @property
        def dew_point(self):
            """
            Compute the dew point temperature for the current Temperature
            and Humidity measured pair
            """
            from math import log
            t, p, h = self.read_compensated_data()
            h = (log(h, 10) - 2) / 0.4343 + (17.62 * t) / (243.12 + t)
            return 243.12 * h / (17.62 - h)

        @property
        def values(self):
            """ human readable values """

            t, p, h = self.read_compensated_data()

            return ("{:.2f}C".format(t), "{:.2f}hPa".format(p/100),
                    "{:.2f}%".format(h))

    class DS1307(object):
        """Driver for the DS1307 RTC."""
        def __init__(self, i2c, addr=0x68):
            self.i2c = i2c
            self.addr = addr
            self.weekday_start = 1
            self._halt = False

        def _dec2bcd(self, value):
            """Convert decimal to binary coded decimal (BCD) format"""
            return (value // 10) << 4 | (value % 10)

        def _bcd2dec(self, value):
            """Convert binary coded decimal (BCD) format to decimal"""
            return ((value >> 4) * 10) + (value & 0x0F)

        def datetime(self, datetime=None):
            """Get or set datetime"""
            if datetime is None:
                buf = self.i2c.readfrom_mem(self.addr, DATETIME_REG, 7)
                return (
                    self._bcd2dec(buf[6]) + 2000, # year
                    self._bcd2dec(buf[5]), # month
                    self._bcd2dec(buf[4]), # day
                    self._bcd2dec(buf[3] - self.weekday_start), # weekday
                    self._bcd2dec(buf[2]), # hour
                    self._bcd2dec(buf[1]), # minute
                    self._bcd2dec(buf[0] & 0x7F), # second
                    0 # subseconds
                )
            buf = bytearray(7)
            buf[0] = self._dec2bcd(datetime[6]) & 0x7F # second, msb = CH, 1=halt, 0=go
            buf[1] = self._dec2bcd(datetime[5]) # minute
            buf[2] = self._dec2bcd(datetime[4]) # hour
            buf[3] = self._dec2bcd(datetime[3] + self.weekday_start) # weekday
            buf[4] = self._dec2bcd(datetime[2]) # day
            buf[5] = self._dec2bcd(datetime[1]) # month
            buf[6] = self._dec2bcd(datetime[0] - 2000) # year
            if (self._halt):
                buf[0] |= (1 << 7)
            self.i2c.writeto_mem(self.addr, DATETIME_REG, buf)

        def halt(self, val=None):
            """Power up, power down or check status"""
            if val is None:
                return self._halt
            reg = self.i2c.readfrom_mem(self.addr, DATETIME_REG, 1)[0]
            if val:
                reg |= CHIP_HALT
            else:
                reg &= ~CHIP_HALT
            self._halt = bool(val)
            self.i2c.writeto_mem(self.addr, DATETIME_REG, bytearray([reg]))

        def square_wave(self, sqw=0, out=0):
            """Output square wave on pin SQ at 1Hz, 4.096kHz, 8.192kHz or 32.768kHz,
            or disable the oscillator and output logic level high/low."""
            rs0 = 1 if sqw == 4 or sqw == 32 else 0
            rs1 = 1 if sqw == 8 or sqw == 32 else 0
            out = 1 if out > 0 else 0
            sqw = 1 if sqw > 0 else 0
            reg = rs0 | rs1 << 1 | sqw << 4 | out << 7
            self.i2c.writeto_mem(self.addr, CONTROL_REG, bytearray([reg]))
    ```

### Adding main MQTT client

The main MQTT client we will use now is very different from the previous basic MQTT example we've done.

In this example, we will integrate all our sensors and use them when needed, we will start by defining in our ode all the sensors such as ADC for the soil moisture sensor, light sensor, BME280 sensor and water quantity sensor.

Then we'll have multiple topics, the topics we will use are:

* plants/soil
* plants/environment
* plants/water

The plants/soil topic is used to publish soil moisture sensor data, take a look at the JSON file below:

      plant = {
          "id":0,
          "name":"Plant A",
          "enabled":1,
          "moisture":normal_reading
      }

We can name the plant as we wish and it will show on our app, enabled can be changed between 1 or 0 which will describe if the plant can be used or not (watering functionality mainly) and inside moisture we add the soil moisture sensor values.

It's import to keep the ID 0 as we use only one soil moisture, if you use the IO extension pins to add more plants, you can change the ID from 0 to 1,2,3 .. as follows:

      plants = [{
          "id":0,
          "name":"Plant A",
          "enabled":1,
          "moisture":normal_reading
        },{
          "id":1,
          "name":"Plant B",
          "enabled":1,
          "moisture":normal_reading_two
        }]

In the plants/environment topic we will publish temperature, humidity, sunlight and water quantity using the rest of the sensors that are integrated in the Eduponics Mini kit.

And finally plants/water will be used to subscribe instead of publishing, in our app we can press the watering icon which will water our plants, our ESP32 will receive the message from the plants/water topic and water the plants accordingly.

Note in water_plant() function the following lines:

      # check if soil moisture is bigger than 60
      if(int(json.loads(get_soil_moisture())["moisture"].replace("%","")) > 60):
          # enough water, stop giving water
          break;

We've set default of allowing to give water only if the plant has less than 60% soil moisture or if the water quantity sensor says there is no more water left. this configuration can be played with as well as to add automatic watering and not manual operation through the app.

Here is the complete code, make sure to save it as <b>main.py</b> as this will be our main program:

=== "MicroPython"
    ``` python linenums="1"
    """
    MicroPython MQTT Eduponics APP Client
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

    from umqttsimple import MQTTClient
    from dependencies import *
    import machine
    import time
    import json

    # set adc (analog to digital) on pin 35
    adc = machine.ADC(machine.Pin(35))
    # set 11dB input attenuation (voltage range roughly 0.0v - 3.6v)
    adc.atten(machine.ADC.ATTN_11DB)

    # Configure light sensor
    light_sensor = LightSensor()

    # Configure BME280
    # setup I2C connection
    i2c = machine.I2C(scl=machine.Pin(15), sda=machine.Pin(4))
    # Initialize BME280 object with default address 0x76
    bme_sensor = BME280(i2c=i2c)
    # initialize dht object, DHT11 coonected to IO19
    #d = dht.DHT11(machine.Pin(19))

    # define water level sensor as INPUT on IO pin number 21
    water_level = machine.Pin(21, machine.Pin.IN)

    # define pump on pin IO23 as OUTPUT
    pump = machine.Pin(23, machine.Pin.OUT)

    # MQTT Unique ID
    UUID = "YOUR_UUID_GENERATED_ID"
    # MQTT Topics
    topics = ["plants/soil","plants/environment","plants/water"]

    def get_soil_moisture():

        # sensor min and max values
        # can be changed and callibrated
        minVal = 710
        maxVal = 4095

        # read soil moisture sensor data
        val = adc.read()

        # scale the value based on maxVal and minVal
        scale = 100 / (minVal - maxVal)
        # get calculated scale
        normal_reading = ("%s%s" % (int((val - maxVal) * scale),"%"))
        # we can also get inverted value if needed
        inverted_reading = ("%s%s" % (int((minVal - val) * scale),"%"))
        # for this example we'll return only the normal reading
        # put everything in a JSON format suitable for the eduponics app
        plant = {
            "id":0,
            "name":"Plant A",
            "enabled":1,
            "moisture":normal_reading
        }
        # return the data
        return str(plant).replace("'",'"')

    def get_environmental_data():

        # get light from the light sensor
        lux = int(light_sensor.readLight())
        # get bme280 sensor data
        bme280_values = bme_sensor.values
        temperature = bme280_values[0].replace("C","")
        pressure = bme280_values[1]
        humidity = bme280_values[2].replace("%","")
        # get DHT11 sensor data
        # measure sensor data
        #d.measure()
        # get temperature and humidity
        #temperature = d.temperature()
        #humidity = d.humidity()
        # get water quantity
        water_quantity = water_level.value()

        # put all this data into a JSON object
        data = {
            "temp":temperature,
            "humidity":humidity,
            "sunlight":lux,
            "water_quantity":water_quantity
        }

        return str(data).replace("'",'"')

    def water_plant():

        # turn on the pump
        pump.value(1)
        # get water quantity
        water_quantity = water_level.value()
        # if water level is not empty, keep doing it
        while not water_quantity == 1:
            # not empty and plant water is not full yet, keep giving water
            # check if soil moisture is bigger than 60
            if(int(json.loads(get_soil_moisture())["moisture"].replace("%","")) > 60):
                # enough water, stop giving water
                break;
            # wait for half a second and check again
            time.sleep(0.5)
            # get water quantity updated value
            water_quantity = water_level.value()

        # turn the pump off, either plant has enough water or no water at all.
        pump.value(0)
        return True

    def on_message_callback(topic, msg):
        '''
        this is a callback, will be called when the app asks for certain information
        such as to water the plants when the watering button pressed
        '''
        # convert topic and message byte to string
        topic = str(topic, 'utf-8')
        msg = json.loads(str(msg, 'utf-8'))

        if(topic == "%s/plants/soil" % UUID or topic == "%s/plants/environment" % UUID):
            # Do nothing, we only publish to those topics
            pass
        elif(topic == "%s/plants/water" % UUID):
            # when the app request for plant watering it goes here
            if("key" in msg and "status" in msg):
                # valid request, let's process it
                if(msg["status"] == "pending"):
                    # it's waiting for us to water it, let's water it
                    water_plant()
                    # after watering, publish success message of watering
                    response = {"key":msg["key"],"status":"ok"}
                    client.publish("%s/plants/water" % UUID, str(response).replace("'",'"'))
        else:
            print((topic, msg))

    def connect_and_subscribe():

        print("[-] Connecting to MQTT client ...")
        # set the MQTT broker object
        client = MQTTClient()
        # set a callback for incoming messages (subscribed topics)
        client.set_callback(on_message_callback)
        # connect to the broker
        client.connect()

        # subscribe to the topics
        for topic in topics:
            client.subscribe("%s/%s" % (UUID,topic))
            print("[-] Subscribed to %s successfully" % topic)
            print("[-] Connected to %s MQTT broker successfully" % client.server)

        return client

    def restart_and_reconnect():
        # something  went wrong, reconnect in 5 seconds ...
        print('[-] Failed to connect to MQTT broker. Reconnecting...')
        time.sleep(5)
        machine.reset()

    try:
        client = connect_and_subscribe()
    except OSError as e:
        restart_and_reconnect()

    # configure few variables
    last_message = 0
    message_interval = 1
    previous_soil_moisture = "0%"

    while True:
        try:
            # check if there are new messages pending to be processed
            # if there are, redirect them to callback on_message_callback()
            client.check_msg()

            # check if the last published data wasn't less than message_interval
            if (time.time() - last_message) > message_interval:
                # get soil moisture
                current_soil_moisture = get_soil_moisture()
                # check if previous data and new data is the same
                if(previous_soil_moisture != current_soil_moisture):
                    # it's not the same, publish the new data
                    previous_soil_moisture = current_soil_moisture
                    client.publish("%s/plants/soil" % UUID, current_soil_moisture)
                    #print("[-] published soil moisture")

                # update environmetal data
                env = get_environmental_data()
                client.publish("%s/plants/environment" % UUID, env)
                #print("[-] published evironmental data")

                # update last message timestamp
                last_message = time.time()

        except OSError as e:
            # if something goes wrong, reconnct to MQTT server
            restart_and_reconnect()
    ```

## Preparing the Eduponics APP

Now once we have everything ready on our ESP32 Eduponics Mini hardware, it's time to move to the app installation and preparation.

### Downloading & Installing Eduponics APP

Currently the app is only available on the Android store and should be able to work on most if not all Android phones. To download, search "Eduponics" in playstore app and you should find it right away.

<p align="center">
  <img src="/kits/eduponics_mini/images/eduponics_mini_android_store.png">
</p>

Alternatively you can install it directly to your phone using the web browser playstore application: [Eduponics Playstore APP](https://play.google.com/store/apps/details?id=com.steminds.eduponics)

### Connecting Eduponics APP to Eduponics Mini

After downloading and launching the app successfully a popup window will show that the hardware is not initialized, this is normal and will happen on first launch of the app or if we've pressed the "wipe data" button in settings to wipe the internal memory of the app.

In this window press "Let's go!" and it will take you automatically to the settings page to bind the Eduponics Mini hardware with the APP.

<p align="center">
  <img style="max-width:50%; height:auto;" src="/kits/eduponics_mini/images/eduponics_app_setup_step_1.jpeg">
</p>

Once we've  been redirected successfully, it's time to bind the app with our hardware.
In order to do so - we'll need the UUID we've generated earlier and initialized in our eduponics mini hardware, we can either type it manually or take a picture of a QR code with the UUID inside of it.

!!! Warning "Make sure to press the 'Enter' key on the virtual keyboard"
      After you've typed the UUID completely, press the enter key on the virtual keyboard to confirm the UUID, this should complete the process.

<p align="center">
  <img style="max-width:50%; height:auto;" src="/kits/eduponics_mini/images/eduponics_app_setup_step_2.jpeg">
</p>

Once the virtual key entered successfully, we are now successfully connected to the same channel (the topics) as our Eduponics Mini hardware! it's time to go back to the "Control" tab and see if we can get real time data from our device.

<p align="center">
  <img style="max-width:50%; height:auto;" src="/kits/eduponics_mini/images/eduponics_app_setup_step_3.jpeg">
</p>

Now you can monitor your plant from everywhere, water it remotely and track the sensors data to make smarter decisions.
As the app doesn't use any external server except the MQTT broker, you can control it even outside of your home network.
