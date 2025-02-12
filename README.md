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
On the Kali laptop I ran the following command to locate the flipper:
```
ls /dev/serial/by-id
```
The output of this command should provide you with the unique name for your flipper. With that you can run the following screen command to enter in to the UART port:
```
screen /dev/serial/by-id/usb-Flipper_Devices_Inc.Flipper_UNIQUE_ID_HERE
```
After running the screen command and letting the netgear boot up, I was dropped into an root shell on the netgear:
```
BusyBox v1.4.2 (2018-12-10 10:26:59 UTC) Built-in shell (ash)
Enter 'help' for a list of built-in commands.

     MM           NM                    MMMMMMM          M       M
   $MMMMM        MMMMM                MMMMMMMMMMM      MMM     MMM
  MMMMMMMM     MM MMMMM.              MMMMM:MMMMMM:   MMMM   MMMMM
MMMM= MMMMMM  MMM   MMMM       MMMMM   MMMM  MMMMMM   MMMM  MMMMM'
MMMM=  MMMMM MMMM    MM       MMMMM    MMMM    MMMM   MMMMNMMMMM
MMMM=   MMMM  MMMMM          MMMMM     MMMM    MMMM   MMMMMMMM
MMMM=   MMMM   MMMMMM       MMMMM      MMMM    MMMM   MMMMMMMMM
MMMM=   MMMM     MMMMM,    NMMMMMMMM   MMMM    MMMM   MMMMMMMMMMM
MMMM=   MMMM      MMMMMM   MMMMMMMM    MMMM    MMMM   MMMM  MMMMMM
MMMM=   MMMM   MM    MMMM    MMMM      MMMM    MMMM   MMMM    MMMM
MMMM$ ,MMMMM  MMMMM  MMMM    MMM       MMMM   MMMMM   MMMM    MMMM
  MMMMMMM:      MMMMMMM     M         MMMMMMMMMMMM  MMMMMMM MMMMMMM
    MMMMMM       MMMMN     M           MMMMMMMMM      MMMM    MMMM
     MMMM          M                    MMMMMMM        M       M
       M
 ---------------------------------------------------------------
   For those about to rock... (%C, %R)
 ---------------------------------------------------------------
root@R7800:/#
```
Next I ran the <i>show nvram</i> command from Matt's video, and I found the following entries in the output:
```
wl_wpa2_psk=[REDACTED]
wla_wpa2_psk=[REDACTED]
```
The two redacted values were the same, I tested them out and they were the pre shared key for the wireless network.

# Password Cracking
