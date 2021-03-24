---
title: STEMinds - Advanced MicroPython and ESP32 tools
description: Let's learn about some advanced tools such as detecting the UART manually and operating the ESP32 with Ampy (an Adafruit Tool) something that can be useful if you want to work outside of the Thonny IDE environment.
---


# Advanced programming tools

Previously we've discussed how to get started with Eduponics mini IDE, if you consider yourself an advanced programmer, you might want to take things one step further. We'd like to introduce some advanced tools such as detecting the UART manually and operating the ESP32 with Ampy (an Adafruit Tool) something that can be useful if you want to work outside of the Thonny IDE environment.

!!! Warning "Manual work from now on"
    We strongly suggest you to start with Thonny IDE if you are a complete beginner
    the following steps will explain about Ampy, Putty and some other tools that can help you install and run scripts manually on your Eduponics kit
    this might be very useful for advanced users but not so friendly for new users without previous experience.

## Connecting to the kit through UART

Once we have the necessary drivers installed, it's time to connect to our board through UART for the first time. Firstly, we'll need to find the UART port.
<br/><br/>
Baud rate also stands for bits per second and is a number related to the speed of data transmission in a system. The rate indicates the number of electrical oscillations per second that occurs within a data transmission. The higher the baud rate - the higher the transfer rate of bits per second.
<br/><br/>
In our Eduponics mini kit, the UART baud rate is <b>115200</b> which we will use by default to communicate with our ESP32 microcontroller.
If we use a different baud rate we might get nonsense or "spaghetti" feedback instead of what we expect the board to print back to us in response.

### in Mac OSX and Linux:

Run the following command in terminal to find connected UART devices
to make the process easier, make sure only the Eduponics Mini kit is connected to your computer by USB. Else, you might see other unrelated devices that might confuse you.

    ls /dev/*usb*

the results should look similar to this:

    /dev/cu.usbserial-14310		/dev/tty.usbserial-14310

In our example the UART name will be "tty.usbserial-14310" which we will use to connect to the Eduponics mini.
<br/><br/>
Now when we have the right UART name, we can easily run the following command in terminal to connect to the ESP32 MicroPython command-line interface:

    screen tty.usbserial-14310 115200

Of course, you will need to change the number "14310" to the one that shows up on your machine - it might look different. "115200" stands for the communication baud rate we use.

!!! info "Small tip for alternatives to screen"
    On Linux, "picocom" or "minicom" may be used instead of "screen". The USB serial address might also be listed as /dev/ttyUSB01 or a higher increment for ttyUSB. Additionally, the elevated permissions to access the device (e.g. group uucp/dialout or use sudo) may be required.

This can be done by typing the following commands

    sudo usermod -a -G dialout $user
    sudo chmod a+rw /dev/ttyUSB0


### In Windows

In Windows it's a little different, let's follow the steps below:

1. Open Device Manager.
2. Click on View in the menu bar and select Show hidden devices.
3. Locate Ports (COM & LPT) in the list.
4. Check for the com ports by expanding the COM menu option.

Once you've located the right COM port, it's time to download putty to connect to the UART port.
<br/><br/>
PuTTY is an SSH and telnet client, developed originally by Simon Tatham for the Windows platform. PuTTY is an open-source software that is available with source code and is developed and supported by a group of volunteers.
<br/><br/>
We use PuTTY to communicate through UART with the Eduponics Mini, in OSX or Linux operation systems - a small piece of software called "screen" is already included, that allows us to communicate through UART without the need of external software
<br/><br/>
You can download Putty from here: [Putty official website](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

<p align="center">
  <img src="https://cdn.steminds.com/docs/kits/eduponics_mini/putty_example.png">
</p>

Once you have Putty installed, you can proceed to entering the COM port that we found earlier in the device manager (it may not necessarily be COM4, it could be a different number)
also make sure to set the baud rate as 115200 as we mentioned earlier.
<br/><br/>
Tick the connection type checkbox as "Serial" once you are ready click the "Open" button to intialize the connection.

!!! info "Thonny IDE detects the UART name automatically"

If you use the Thonny IDE for your programming needs, in the interpeter it will already show you the port name and device information without needing to look for it manually.

## Adafruit MicroPython Tool

Adafruit MicroPython Tool (ampy) - Utility helps you to interact with a MicroPython board over a serial connection. This appreciable utility can be useful to send, read, and run files on our Eduponics mini board with ease.
<br/><br/>
Ampy is meant to be a simple command line tool to manipulate files and run code on a MicroPython board over its serial connection. With ampy you can send files from your computer to the board's file system, download files from a board to your computer, and even send a Python script to a board to be executed.
<br/><br/>
We can find the ampy github repository here: [https://github.com/scientifichackers/ampy](https://github.com/scientifichackers/ampy)
<br/><br/>
The installation is very straight forward, let's make sure we have all the pre-requisites such as pyhon-pip, pyserial and python3.

### on MacOS or Linux

in a terminal run the following command:

    pip3 install --user adafruit-ampy

Note on some Linux and Mac OSX systems you might need to run as root with sudo:

    sudo pip3 install adafruit-ampy

### On Windows:

do the following

    pip3 install adafruit-ampy

### Installing from source

If you'd like to install from the Github source then use the standard Python setup.py install:

    git clone https://github.com/scientifichackers/ampy.git
    cd ampy
    python3 setup.py install

## Ampy Usage information

Once installed verify you can run the ampy program and get help output:

    ampy --help

You should see usage information displayed like below:


    Usage: ampy [OPTIONS] COMMAND [ARGS]...

      ampy - Adafruit MicroPython Tool

      Ampy is a tool to control MicroPython boards over a serial connection.
      Using ampy you can manipulate files on the board's internal filesystem and
      even run scripts.

    Options:
      -p, --port PORT  Name of serial port for connected board.  [required]
      -b, --baud BAUD  Baud rate for the serial connection. (default 115200)
      -d, --delay DELAY Delay in seconds before entering RAW MODE (default 0)
      --help           Show this message and exit.

    Commands:
      get  Retrieve a file from the board.
      ls   List contents of a directory on the board.
      put  Put a file on the board.
      rm   Remove a file from the board.
      run  Run a script and print its output.

### Running files


To run files with ampy we first need to make a file.
let's make a python script and call it <b>test.py</b> this file can be created in text editor or your IDE / Code editor.
<br/><br/>
inside the python script let's write the following:

``` python linenums="1"
print("hello world")
```

Make sure to save it. Once we have test.py file with the print hello world command inside of it, we can run it on our microcontroller using ampy by running the following commands:

!!! info "Serial port name"
    You will need to change the USB Serial port name as it shows on your computer

In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> run test.py

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> run test.py


Once you'll run the code a response should come back with the text "hello world"

!!! info "Running doesn't mean save the code"
      Running the code on your ESP32 Eduponics Mini board doesn't save the code inside the board's memory - it will only execute it.
      To save the code and run it later, proceed to the following steps where we will explain how to "push" files.

### Pushing files

Pushing files is very similar to executing (running) files except we put them into our ESP32 device without actually running them, later on we can run them directly from the ESP32 board.

In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> put test.py

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> put test.py

Again, please do remember you need to change the COM port if you are on windows or type the correct serial number if you are on Linux / OSX.

### Getting files

Getting files is the process of taking a file we have previously pushed into our ESP32 board but for some reason we don't have it anymore or we changed our local file and want to get the file that is currently inside the board. To do that, we'll use the "get" command to get the file from the ESP32 board into our local machine.
<br/><br/>
In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> get test.py

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> get test.py

Please note that "get test.py" has nothing to do with your local file test.py on your machine, by running "get test.py" it will overwrite any existing files named test.py, if you want to name it differently add another line at the end to state the new file name as follows:
<br/><br/>
In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> get test.py new_test.py

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> get test.py new_test.py

### Removing files

Sometimes we don't need the files we pushed earlier into our board, they take up unnecessary flash space and therefore we should delete them. to do so, we can use the "rm" command which stands for removal of existing files on the board
<br/><br/>
In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> rm test.py

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> rm test.py

Note, this will not affect any local file on your machine named "test.py" it will only remove the "test.py" file from the memory of the ESP32 chipset.

### Listing files

The final step will be listing the files, let's say you previously pushed some files and removed some files and it's been some time or you've got confused what files we have in the ESP32 board and what files are missing ... all of this can be solved with one simple command "ls" which stands for list.
<br/><br/>
In Mac OSX / Linux:

    ampy --port /dev/tty.usbserial-<NUMBER_GOES_HERE> ls

In Windows:

    ampy --port <COM_PORT_NAME_GOES_HERE> ls

As you can see, we don't need to mention any file that's because we just want to get the name of all the files that already exist on the ESP32 board.
<br/><br/>
That's all the Ampy commands that you'll need, make sure to practice by pushing, getting, removing and listing files before you move to your own project to avoid mistakes!

## Coming up next

Congratulations! you've made your first step in command line programming, hardware and the STEM world!
<br/><br/>
Next, we'll start by introducing each sensor that is available on top of the Eduponics mini kit. After you'll master the science behind the sensors and how to program them yourself. We'll then move on to more advanced examples, combining multiple sensors together to achieve a fully functional IoT plant monitoring system.
