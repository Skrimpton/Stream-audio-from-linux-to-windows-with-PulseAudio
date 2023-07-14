# Streaming audio from Linux to Windows
 <div align="right">
      Almost all of this is copied and pasted from Thomas Jepp. I just added some minor points<br>
      https://thomasjepp.uk/2015/05/31/streaming-audio-from-linux-to-windows.html<br>
      2015-05-31
 </div>
 
## Guide - Table of contents:

* [PulseAudio on Linux](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#pulseaudio-on-linux)
  
* [Windows](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#windows)
  * [Find IP-Adress of server](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#finding-the-ip-adress-for-the-server)
  * [PulseAudio on windows](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#pulseaudio-on-windows)
  * [NSSM and making a service](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/blob/main/README.md#nssm-and-making-a-service---permanent-automatic-setup)
    * [Start service](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#finally-start-the-newly-installed-service)

<br>

## Further reading - PulseAudio documentation
<table>
 <tr>
   <th><a href="https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/">PulseAudio docs - Overview</a></th>
   <th>https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/</th>
 </tr>
 <tr>
   <th><a href="https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/">PulseAudio docs - Modules</a></th>
   <th>https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/</th>
 </tr>
 <tr>
   <th><a href="https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Network/">PulseAudio docs - Network</a></th>
   <th>https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Network/</th>
 </tr>
</table>

<br>

## Background

Having multiple PCs all needing to play audio is a pain - especially when you use a headset.

This is how I used to stream audio from my NUC running Linux to my Windows desktop - where the headset was plugged in.

* *Link and/or photo missing, I guess. There is no further explanation as to "how"*
<br>

## Options:
There are a few options for getting audio from Linux to Windows:

   * Using an actual cable
     - this doesn’t work when you have more devices than you have line in jacks, and you tend to get analog noise
   * Using JACK
     - this is rather more complicated than I would like, and doesn’t integrate very well with the Linux or Windows ends
   * Using PulseAudio
     - the Windows port of PulseAudio isn’t as well maintained as I’d like, but this integrates well with Linux

<br>

Given these choices, I ended up using PulseAudio.

<br>

## PulseAudio on Linux

Setting up the Linux side of this is really easy:

1) Open
   ```
   /etc/pulse/client.conf
   ```
2) and add:
   ```
   default-server = 192.168.1.1
   ```
   * *[Change 192.168.1.1 to the IP of your Windows machine](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#finding-the-ip-adress-for-the-server)*.

3) Run:
   ```
   killall pulseaudio
   ```
<br>

# Windows
## Finding the ip adress for the server
1) Press the windows key
   
2) Type:
   ```
   cmd
   ```

3) Open CMD and run:
   ```
   ipconfig
   ```
4) Look for the block of addresses matching current/desired network
   ```
   Ethernet adapter Ethernet                      ← This is the block for cable
   Ethernet adapter Wireless LAN adapter Wi-Fi    ← Would ofc be Wireless internet
   ```
5) Use the numbers from this line:
   ```
   IPv4 Address. . . . . . . . . . . : they.will.be.here
   ```
<br>

## PulseAudio on Windows

PulseAudio isn’t well maintained on Windows - the binaries linked from the official site are very old - for PulseAudio 1.1.

However, I found a much newer set of binaries from the X2Go project: http://code.x2go.org/releases/binary-win32/3rd-party/pulse/.

<br>

## To set up PulseAudio 5.0 (or 1.1) on Windows, do the following:

1) Download pulseaudio-5.0-rev18.zip from either:
   * x2go
       *  http://code.x2go.org/releases/binary-win32/3rd-party/pulse/pulseaudio-5.0-rev18.zip
       * [This github with zip from x2go - if x2go link dies](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/archive/refs/heads/main.zip)  
   *  ***or Freedesktop.org***
     
       *  [direct link to zip](http://bosmans.ch/pulseaudio/pulseaudio-1.1.zip)
       *  https://www.freedesktop.org/wiki/Software/PulseAudio/Ports/Windows/Support/
     
2) Extract it and copy the pulse folder to
    ```
    C:\pulse
    ```
3) Create a config.pa file in that folder with these contents:
      
    ```
    load-module module-native-protocol-tcp port=4713 auth-ip-acl=127.0.0.1;192.168.1.0/24
    load-module module-esound-protocol-tcp port=4714 auth-ip-acl=127.0.0.1;192.168.1.0/24
    load-module module-waveout
    ```

   * Replace 192.168.1.0/24 with your local subnet.
  
  ---
  
   * *You can alternatively add another subnet with ; and the new subnet:*
     ```
     auth-ip-acl=127.0.0.1;192.168.0.0/24;10.0.0.0/24
     ```
   * *If you have problems, you can also try allowing all connections:* *<br><ins> **IMPORTANT NOTE:**</ins> Could be a security risk, as mentioned in the source.*
      ```
      auth-anonymous=1 
      ```
     * [source](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Network/) - Section: Authorization
     * [example](https://openwrt.org/docs/guide-user/hardware/audio/pulseaudio#configuration)
 ---
 
4) Test this setup by running:
   
    ```
    c:\pulse\pulseaudio.exe -F config.pa
    ```

<br>

---
### You should now be able to get Linux sound playing on your Windows PC.
---

<br>

# NSSM and making a service - Permanent, automatic setup
For a permanent setup we need to create a Windows service rather than running PulseAudio in a command prompt.

I use NSSM to run arbitrary programs as services.

<br>

1) Download the latest version of NSSM.
   * https://nssm.cc/download
2) Extract it and copy nssm.exe from the win32 folder to:
    ```
    c:\pulse
    ```
3) Run:
    ```
     c:\pulse\nssm.exe install PulseAudio
    ```
4) Fill in the following details on the <ins>___Application___</ins>-tab
    * Path:
      
      ```
      c:\pulse\pulseaudio.exe
      ```
    * Startup directory:

      ```
      c:\pulse
      ```
    * Arguments:

      ```
      -F c:\pulse\config.pa
      ```

5) On the Details tab, fill in:
    * Display name:
      
      ```
      PulseAudio
      ```
6) Now click <ins>___Install service___</ins>.

<br>

## Finally, start the newly installed service

* either through ___Services___ in
  
    ```
    Administrative Tools
    ```
* or by running
  
    ```
    net start PulseAudio
    ```

## Guide - Table of contents:
* [Back to top](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#pulseaudio-on-linux#streaming-audio-from-linux-to-windows)
  
* [PulseAudio on Linux](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#pulseaudio-on-linux)
  
* [Windows](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#streaming-audio-from-linux-to-windows)
  * [Find IP-Adress of server](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#finding-the-ip-adress-for-the-server)
  * [PulseAudio on windows](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#pulseaudio-on-windows)
  * [NSSM and making a service](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/blob/main/README.md#nssm-and-making-a-service---permanent-automatic-setup)
    * [Start service](https://github.com/Skrimpton/Stream-audio-from-linux-to-windows-with-PulseAudio/tree/main#finally-start-the-newly-installed-service)
