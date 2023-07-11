# Streaming audio from Linux to Windows
 <div align="right">
      https://thomasjepp.uk/2015/05/31/streaming-audio-from-linux-to-windows.html<br>
      2015-05-31
 </div>
 
## Guide - Table of contents:
* [PulseAudio on Linux](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#pulseaudio-on-linux)
  
* [Windows](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#windows)
  * [Find IP-Adress of server](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#finding-the-ip-adress-for-the-server)
  * [PulseAudio on windows](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#pulseaudio-on-windows)
  * [NSSM and making a service](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#permanent-automatic-setup)
    * [Start service](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#finally-start-the-newly-installed-service)

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
   * *[Change 192.168.1.1 to the IP of your Windows machine](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio.md#finding-the-ip-adress-for-the-server)*.

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
       * [Zip from this github - if x2go link dies](https://github.com/Skrimpton/Misc/blob/main/Stream%20linux%20audio%20using%20pulseaudio%20-%20pulseaudio-5.0-rev18%20%5Bd110723%20t055129%5D.zip)  
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
     
   * *or add others:*
     ```
     auth-ip-acl=127.0.0.1;192.168.0.0/24;10.0.0.0/24
     ```
   * *or allow all*
      ```
      auth-anonymous=1 
      ```
     * [source](https://www.freedesktop.org/wiki/Software/PulseAudio/Documentation/User/Modules/) - *do a page search for "auth-anonymous"*
     * [example](https://openwrt.org/docs/guide-user/hardware/audio/pulseaudio#configuration)
<br>

4) Test this setup by running:
   
    ```
    c:\pulse\pulseaudio.exe -F config.pa
    ```

---
### You should now be able to get Linux sound playing on your Windows PC.
---
<br>

# Permanent, automatic setup
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
