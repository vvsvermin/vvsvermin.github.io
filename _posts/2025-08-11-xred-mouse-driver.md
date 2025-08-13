---
title: "Xred malware in gaming mouse configuration tool"
date: "2025-08-11 00:00:00"
categories: [misc]
tags: [misc]
---

# Sample Info

| SHA256|
|---------------------------------------------------------------------------------|
|7eb0582843dda8500cae54d240eddb728fd146584735ebe65605efecc5e1b376 |

* Size:  2.95MB
* First seen:  2025-07-02 09:10:37 UTC
* File Type:  .exe

# About

Amidst July, I found a reddit post notifying users of Endgame Gear's OP1w 4K V2 that the mouse configuration tools has been bundled with XRed malware. Endgame Gear is a gaming Company, which is well known for its mice. This was strange, since the download came from the official Endgame Gear website and not off some shady 3rd party distributor. The company did eventually post a [statement](https://www.endgamegear.com/en-at/security-update), however, they did not state how this incident may have happened. According to [malpedia](https://malpedia.caad.fkie.fraunhofer.de/details/win.xred) the XRed backdoor has been around since at least 2019. Linked on the website is an older report from [eSentire](https://www.esentire.com/blog/xred-backdoor-the-hidden-threat-in-trojanized-programs). Similar to our current situation, the sample from eSentire disguises itself as a Synaptics driver. For the most part, the inner part of the backdoor has not changed, but there are some key minor differences.

# Analysis

The infected file is compiled with delphi (unlike the legitimate version, which is compiled with Borland C++). Just throwing it into IDA  and trying to analyse it will not be a pleasant experience. Luckily, [this video](https://www.youtube.com/watch?v=04RsqP_P9Ss) from OALabs explains how to make things easier. Otherwise, there is no forms of obfuscation.

Just like other samples, the malware features the ability to download URLs 
![Alt text](/assets/2025-08-11-xred/image-1.png )*Ability to download from URLs*

List of URLs:

```
hxxp://freedns[.]afraid[.]org/api/?action=getdyndns&sha=a30fa98efc092684e8d1c5cff797bcc613562978

hxxps://docs[.]google[.]com/uc?id=0BxsMXGfPIZfSVlVsOGlEVGxuZVk&export=download

hxxps://www[.]dropbox[.]com/s/n1w4p8gc6jzo0sg/SUpdate[.]ini?dl=1

hxxp://xred[.]site50[.]net/syn/SUpdate[.]ini

hxxps://docs[.]google[.]com/uc?id=0BxsMXGfPIZfSVzUyaHFYVkQxeFk&export=download

hxxps://www[.]dropbox[.]com/s/zhp1b06imehwylq/Synaptics[.]rar?dl=1

hxxp://xred[.]site50[.]net/syn/Synaptics[.]rar

hxxps://docs[.]google[.]com/uc?id=0BxsMXGfPIZfSTmlVYkxhSDg5TzQ&export=download

hxxps://www[.]dropbox[.]com/s/fzj752whr3ontsm/SSLLibrary[.]dll?dl=1

hxxp://xred[.]site50[.]net/syn/SSLLibrary[.]dll
```



Additionally, it sets up an USB Hook to copy itself into plugged in USB drives, and afterwards sets up [autorun.inf](https://en.wikipedia.org/wiki/Autorun.inf) to spread itself via USB drives. Additionally, it logs keystrokes.
![Alt text](/assets/2025-08-11-xred/image.png)*USBHook, KEYBOARDHook, and Ability to send Mails*

Furthermore, this malware possesses the ability to send out emails to the following email-addresses: 
```
xredline1@gmail[.]com
xredline2@gmail[.]com
xredline3@gmail[.]com
```

Strings suggest that the malware is capable of a few other things as well.

![Alt text](/assets/2025-08-11-xred/image4.png)*Other remotely executable commands*

To maintain persistence, it drops itself into "C:\ProgramData\Synaptics\" as "Synaptics.exe" and is added into the registry "SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run". This ensures that the application runs after starting the computer.

![Alt text](/assets/2025-08-11-xred/image3.png)*Autostart on startup*

Taking a look at RCDATA, we can extract three files: 

# KBKHS.dll

| SHA256|
|---------------------------------------------------------------------------------|
| b9eae90f8e942cc4586d31dc484f29079651ad64c49f90d99f86932630c66af2 |

* Size:  15.00KB
* File Type:  .dll

This dll contains only two functions. Both access the file mapping called "ElReceptor" (created in the main .exe) and post messages which then could be accessed by the main process. This only happens when triggered by a certain code: Either the code is 0 ([HC_ACTION: The wParam and lParam parameters contain information about a keystroke message.](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/legacy/ms644984(v=vs.85))) or it is equal to 5

![Alt text](/assets/2025-08-11-xred/image5.png)*Write into shared memory under specific circumstance*

# XLSM.xlsx

| SHA256|
|---------------------------------------------------------------------------------|
| 8e574b4ae6502230c0829e2319a6c146aebd51b7008bf5bbfb731424d7952c15 |

* Size:  17.96KB
* File Type:  .xlsx

Uses VBA macros to modify registry keys and disable the macro warning in Word and Excel


![Alt text](/assets/2025-08-11-xred/image6.png)*Modify registry key and disable macro warning*

Also checks if "Synaptics.exe" (itself) is placed in certain directories. If not, it downloads them from Dropbox or Google Docs (same links as above).


# Endgame Gear OP1w 4k v2 Configuration Tool

| SHA256|
|---------------------------------------------------------------------------------|
| d5e5e314beecb61afc1726579b7e86d625cf409c41f4e9dd9719f3d1f6889538 |

* Size:  2.22MB
* File Type:  .exe

This file is the legitimate configuration tool, without any malware bundled in it. Unlike [eSentire's](https://www.esentire.com/blog/xred-backdoor-the-hidden-threat-in-trojanized-programs) sample, this one has the legitimate file bundled with it, and does not download it beforehand.


# Final Words

After this incident, I might conclude that even if the source of the download is legitmate, it's probably best to run every application through an antivirus. 