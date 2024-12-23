 Hello! Today I'm going to share with you the way I solved GreenHorn HTB machine.
 Firstly, let's edit our /etc/hosts file and add IP address of this machine

<img src="images/1.png" />


 Now, let's use nmap scanner to look for some open ports.

<img src="images/2.png" />

 As we can see, we 22,80 and 3000 are opened, let's now look at the web page of this machine:
 
<img src="images/3.png" />

 This seems to be some kind of web-development plaform for beginners. Let's go to the admin panel, which we can find at the bottom of the page

<img src="images/4.png" />

 Here we can see, that web-page is powered by pluck framework which has 4.7.18 version. But before we'll drive deep into seaching common vulnerabilities, let's visit 3000 port.


<img src="images/5.png" />

 If we look close on this page, we'll see that we can go straight into repository of this app.

<img src="images/6.png" />

Firstly I thought, that it's nothing interesting on this page. but then i found interesting file located at src/branch/main/data/settings/pass.php

<img src="images/7.png" />

 This file seems to be password hash, let's try to crack it using [CrackStation](https://crackstation.net/)

<img src="images/8.png" />

Cool! We've got our first password. What's next? Let's search common vulnerabilities of plugin-4.7.18. I've found  [this repository](https://github.com/Rai2en/CVE-2023-50564_Pluck-v4.7.18_PoC) on GitHub, which is PoC of CVE-2023-50564. CVE-2023-50564 is a vulnerability that allows unauthorized file uploads in Pluck CMS version 4.7.18. This exploit leverages a flaw in the module installation function to upload a ZIP file containing a PHP shell, thereby enabling remote command execution. Let's get our hands dirty! As it said in manual, let's edit file poc.py and add there target urls.
 

<img src="images/9.png" />

Now let's try to upload malicious zip archive on web page. It seems that we can do that through uploading new "module", as it shown in picture bellow

<img src="images/10.png" />

It's time to run poc.py and finally get RCE. But before let's run netcat with listening mode on 4444 port.

<img src="images/11.png" />

 And we get RCE :)

<img src="images/12.png" />

 From my point of view it always will be good pratise to stabilize our reverse shell. For this reason let's execute command bellow
```
python3 -c 'import pty;pty.spawn("/bin/bash")'
```
<img src="images/13.png" />

Now we've got fully interactive shell. Let's explore file system.

<img src="images/14.png" />

 Apperantly we can't get into users home directory from 'www-data'. Let's try to escalate priveleges by logging into junior account with password we found before.

<img src="images/15.png" />

Cool! Now we can get user flag :)
