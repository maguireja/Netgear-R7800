# netgear
Notes for Netgear R7800

# Thirfting
I found a Netgear R7800 at a local thrift store, it was priced at $9.99 but it was 20% off day so I was out the door for less than $9 after tax.
<img src="https://github.com/maguireja/netgear/blob/b1546eec3635e6b43bf14a6fd5d3226c1ab586a2/netgear.png">

# Powering up
I plugged in the router and to my surprise, the router had not been reset to a factory default before being donated. I observed the following SSID broadcasted:
<img src="https://github.com/maguireja/netgear/blob/92d5ef63adb5aaa382886b7a4d19818efef5f901/SSID.png">
...do with that what you will...

# Password Recovery
Now that I realized the router was still configured, I wondered if I could recover the password for the wireless network, as well as any other configured passwords. I followed along with Matt Brown's video here which was fantastic. I recommend subscribing to Matt's channel he has some really good content.
Video here: https://youtu.be/7iuwY3hIcHw?si=osBN4qiOezMgb0qB

# Locating UART
I opened up the case and found four pins that look like UART, but they were not labeled. I was able to follow the steps in Matt's video and use a multimeter to locate the Ground, Send and Recieve pins.
<img src="https://github.com/maguireja/netgear/blob/4a948d08c5084d1741fed89ba6b057aac21a280b/R7800%20pins.png">

Next I used a Flipper Zero and a laptop running Kali Linux to connect to the UART port. I used the Flipper's USB-UART Bridge Module with the following config:
```
Baud 115200
```

# Dumping NVRAM

# Password Cracking
