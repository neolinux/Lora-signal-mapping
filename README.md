# LoRa Signal Mapping

This project is designed to map the signal strength between two LoRa devices and log the GPS coordinates in 3D.

The project is divided between three hardware parts
 - sender
 - receiver
 - thethingsnetwork RPi gateway

Overview of the project in a block diagram (for now):
![N|Solid](https://raw.githubusercontent.com/IRNAS/Lora-signal-mapping/master/images/protocol.png)

## Sender 
The sender is the main component of this project. This devices is placed on the drone and it is logging GPS data along with LoRa packet signal strength.

Senders task:
 - Setup LoRa, GPS and SD Card protocol
 - Wait for GPS to fix
 - Send GPS data through LoRa and write to .kml file (SD)
 - Retrieve LoRa signal power

### Hardware 
The hardware consists of:
 - Arduino UNO
 - Dragino LoRa/GPS module for Arduino (http://wiki.dragino.com/index.php?title=Lora/GPS_Shield)
 - DIY SD Card shield
 
![N|Solid](https://raw.githubusercontent.com/IRNAS/Lora-signal-mapping/master/images/sender_setup.jpg)

### Software
The software is written in C/C++ in the Arduino IDE.

It is divided into 3 parts. ***LoRa***, ***GPS*** and ***SD Card***
#### LoRa
LoRa is the main character in this project and it is used to send data and retrieve the power of the signal in dBm. 

***Receiving*** works through the ```onReceive()``` interrupt function, where it executes when we receive a packet from the receiver part.

***Sending*** works through a custom function  ```void send_gps(float lat, float lon, float alti, float speed) ``` where the parameters are packaged and sent to the receiver. 

For LoRa communication we are using the ```<LoRa.h>``` library.

#### GPS
The GPS is used for logging the current position and speed. This data is processed through the ```gps_loop()``` function. 
If the GPS is ***fixed*** only then it will get the coordinates and send it through the LoRa system and if an SD Card is available it will write it onto there too.

For debugging purposes we are using the ```<SoftwareSerial.h>``` library for GPS and the standard SerialPort for serial debugging. 

For GPS we are using the ```<TinyGPS.h>``` library.

#### SD Card
The SD Card is a very useful part of this project, because it will log the position and the power of the LoRa signal independently from the LoRa system. 
It will create a ```.kml``` file and that file can be opened with ***Google Earth*** and inspect the signal strength based on the color of the pins.
See the [color table](https://raw.githubusercontent.com/IRNAS/Lora-signal-mapping/master/images/color.png) and an [example](https://github.com/IRNAS/Lora-signal-mapping/blob/master/example/generated_3d.KML) of the kml file

For SD Card communication we are using the ```<SD.h>``` library.

## Receiver
This devices is placed "on the ground" and it is logging everything that is sent through the LoRa network. It is filters the incoming data and if everything is right it will log the GPS data and display it on the screen (SSD1306). 

With the [python tool](https://github.com/IRNAS/Lora-signal-mapping/blob/master/python_earth/test.py) it is capable of live tracking through the serial port. 

### Hardware
The hardware consists of:
 - Arduino PRO Mini
 - LoRa device
 - SSD1306 [datasheet](https://cdn-shop.adafruit.com/datasheets/SSD1306.pdf)
 - USB to UART converter
 
![N|Solid](https://raw.githubusercontent.com/IRNAS/Lora-signal-mapping/master/images/gps_writing_oled.jpg)

### Software
The software depends on the LoRa receiving a packet from the sender. 
It is also using the ```onReceive()``` function to receive a packet when avaible. 
In this function we are checking if the received ```char data[packetSize]``` has the required data, if it does we are converting all the values to floats, showing it on the display and parsing it through the serial port.

The display we are using is the ***SSD1306 I2C version***.
The library used is ```<U8g2lib.h>```.

Every 500ms it is sending a ***status*** (about receiving valid GPS data) and ***receive power*** information.

## Raspberry Pi gateway
We are using a Raspberry Pi 3 with [ic880a](https://github.com/ttn-zh/ic880a-gateway/wiki) board for the gateway.

The software used is [resin.io](https://resin.io/).

![N|Solid](https://raw.githubusercontent.com/IRNAS/Lora-signal-mapping/master/images/rpi_gateway.jpg)

---
All projects of Institute IRNAS are as usefully open-source as possible.

Hardware including documentation is licensed under CERN OHL v.1.2. license <http://www.ohwr.org/licenses/cern-ohl/v1.2>.

Firmware and software originating from the project is licensed under GNU GENERAL PUBLIC LICENSE v3
<http://www.gnu.org/licenses/gpl-3.0.en.html>.

Open data generated by our projects is licensed under CC0 <https://creativecommons.org/publicdomain/zero/1.0/legalcode>.

All our websites and additional documentation are licensed under Creative Commons Attribution-ShareAlike 4 .0 Unported License
<https://creativecommons.org/licenses/by-sa/4.0/legalcode>.

What this means is that you can use hardware, firmware, software and documentation without paying a royalty and knowing that 
you'll be able to use your version forever. You are also free to make changes but if you share these changes then you have to 
do so on the same conditions that you enjoy.

Koruza, GoodEnoughCNC and IRNAS are all names and marks of Institut IRNAS Rače. You may use these names and terms only to 
attribute the appropriate entity as required by the Open Licences referred to above. You may not use them in any other way 
and in particular you may not use them to imply endorsement or authorization of any hardware that you design, make or sell.
