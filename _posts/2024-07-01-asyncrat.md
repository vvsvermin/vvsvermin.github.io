# Sample Info

| SHA256|
|---------------------------------------------------------------------------------|
|9c457b1cd061ae951fbed7841149b247e085befa6e2c5170058ce35cdebce548 |

* Size:  48640 bytes
* First seen:  2024-06-25 04:15:54 UTC
* File Type:  .exe

# About

AsyncRAT is a Remote Access Tool (RAT) designed to remotely monitor and control other computers through a secure encrypted connection. It is an open source remote administration tool, however, it could also be used maliciously because it provides functionality such as keylogger, remote desktop control, and many other functions that may cause harm to the victim’s computer. In addition, AsyncRAT can be delivered via various methods such as spear-phishing, malvertising, exploit kit and other techniques. (Source: https://malpedia.caad.fkie.fraunhofer.de/details/win.asyncrat)

# Analysis

I mainly want to focus on the config extraction in this report, as this is the most interesting part. Vital symbol names were renamed during analysis to be more readable, which makes the program flow easier understandable for the reader.

So, lets head straight into the analysis:


![image](https://github.com/santanaworld/re-stuff/assets/97342354/db6c6f7e-546f-40f6-8435-a3b66c5d289a)


This is how the starting point/Main looks like. In the beginning, the malware sleeps for 3 seconds ("repeat" is an int equal to 3). Then, the malware starts the initialization.

![image](https://github.com/santanaworld/re-stuff/assets/97342354/075785ca-9fe6-42a7-812b-935301407b6a)

This is how it looks like. There are many base64 strings, which make up the config when decrypted via AES. Line 19 extracts the AES-Key via simple base64 decryption of a string. 
Let's check out the decryption mechanism.



The flow is pretty simple: 
First, we derive the key used for decrypting via PBKDF2-HMAC. This happens before the decrypt function is executed.
![image](https://github.com/santanaworld/re-stuff/assets/97342354/da9c110b-48a4-45e4-882f-5b0f0c237cab)

Then we use this derived key for the actual config extraction. To obtain the IV, we simply need to get the bytes [32:48] from the input data. The first 32 bytes are used for verification purposes. 
The encrypted message starts at [48:]

![image](https://github.com/santanaworld/re-stuff/assets/97342354/8c92674f-d7c7-40f2-baee-9185f933c490)

I have attached a python script, which replicates the decryption mechanism used in the malware sample.
Using the script we can obtain crucial information for the malware's flow. (The server signature and certificate can be ignored)

![image](https://github.com/santanaworld/re-stuff/assets/97342354/d125d589-0724-4e99-8ed4-8ecc7bf696ee)


And thus, the initialization is complete. Now, we will see how the value of "Anti", "Install", "Pastebin" and "BDOS" affect the program flow.

![image](https://github.com/santanaworld/re-stuff/assets/97342354/a81a422d-6f51-4dbd-b635-7ea084df40d4)


-"Anti" decides if the malware checks for any kind of debugging or vm upon execution. But, since the value of "Anti" is null, the Convert.ToBoolean returns "False", meaning it won't check for anything.
-The same thing for "Pastebin" (decides whether the stolen information is stored in a pastebin link or sent to a server) and "BDOS" (executes rtlsetprocessiscritical which may result in a blue screen of death)
-"Install" however, is "true". Let's see what it does:


![image](https://github.com/santanaworld/re-stuff/assets/97342354/734f5728-2deb-4fcc-8616-c25268650d91)


First, it compares the path to the "%appdata%/svchost.exe". if it's not the same, the process gets killed.
Then it sets itself up for persistence. If the malware was run as administrator, it will execute a cmd command to run the virus on startup. If not, it adds a registry key to SOFTWARE\Microsoft\Windows\CurrentVersion\Run, which does the same thing.
The malware then adds a .bat file to %temp% which runs the file in %appdata% and then deletes itself.

Furthermore, the malware uses SetThreadExecutionState to prevent the infected system from sleeping. 


# Conclusion

The malware uses AES to decrypt vital information and depending on the hackers preference, it can check for debuggers and VM's before launching the truly malicious part. It also has different ways of keeping itself persistent

I won't go further investigating the C2. Functionalities of this RAT are pretty much keylogging, screen viewer and computer control. For more info, you can check the source code, since it is available to the public on github.



