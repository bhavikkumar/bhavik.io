---
layout: post
title:  "Bash on Ubuntu on Windows"
date:   2016-08-06 12:00:00 +1200
---
I like to keep my work machine and personal machine completely independent of each other. The only issue is that at home I use Windows 10 which can cause challenges for the tools which I use to build software. This has been getting better in recent times with the likes of Docker for Windows being released. At times I do still find that there are some tools which just won't play nicely.

With the release of the [Windows 10 anniversary update](https://support.microsoft.com/en-nz/help/12387/windows-10-update-history?ocid=update_setting_client), there is a feature which should resolve most, if not all my issues going forward. The feature is the _Windows Subsystem for Linux_ which should allow us to run Ubuntu on Windows.

### Installing Windows Subsystem for Linux
This was straight forward once you've installed the Windows 10 anniversary update, install the Windows Subsystem for Linux is very easy.

 - Open Control Panel
 - Click Programs
 - Click Turn Windows features on or off
 - Check Windows Subsystem for Linux
 - Click OK

Once the installation is complete and the machine has rebooted the Windows Subsystem for Linux will be installed.

### Problems!
The first thing I did was open up Windows Powershell and type in _bash_ but it did not work.

```
PS C:\Users\bhavi> bash
Unsupported console settings. In order to use this feature the legacy console must be disabled.
PS C:\Users\bhavi>
```
This was a bit confusing at first but the fix is easy.

 - Right click on the title bar
 - Click Properties
 - Click on the Options tab
 - Uncheck Use legacy console (requires relaunch)
 - Click OK
 - Restart Windows Powershell

Lets give _bash_ a go now.

```
PS C:\Users\bhavi> bash
-- Beta feature --
This will install Ubuntu on Windows, distributed by Canonical
and licensed under its terms available here:
https://aka.ms/uowterms

In order to use this feature you must have Developer Mode enabled.
PS C:\Users\bhavi>
```
Ok, so I'm not running Windows in developer mode, another easy fix.

 - Press Windows key
 - Search Developer Mode
 - Press Enter
 - Check Developer Mode
 - Click Yes on the message warning you

![Jenkins AWS S3 profile]({{ site.baseurl }}/assets/windows-10-developer-mode.png)

I got a warning saying some features may not be available until restarting the machine.

### Finally Ubuntu on Windows
Lets just try going back to our powershell window and typing _bash_

```
PS C:\Users\bhavi> bash
-- Beta feature --
This will install Ubuntu on Windows, distributed by Canonical
and licensed under its terms available here:
https://aka.ms/uowterms

In order to use this feature you must have Developer Mode enabled.
PS C:\Users\bhavi> bash
-- Beta feature --
This will install Ubuntu on Windows, distributed by Canonical
and licensed under its terms available here:
https://aka.ms/uowterms

Type "y" to continue: y
Downloading from the Windows Store... 10%
```
It works! We just need to agree to the Ubuntu license and then wait for the download to complete. Then the last step is to setup the user.

```
Extracting filesystem, this will take a few minutes...
Please create a default UNIX user account. The username does not need to match your Windows username.
For more information visit: https://aka.ms/wslusers
Enter new UNIX username: bhavik
Enter new UNIX password:
Retype new UNIX password:
passwd: password updated successfully
Installation successful!
The environment will start momentarily...
Documentation is available at:  https://aka.ms/wsldocs
bhavik@BHAVIK:/mnt/c/Users/bhavi$
```
Look at that, we are now in bash shell which you can run any of those wonderful Linux tools and commands.

You can also get to bash, by just pressing the windows key and typing _bash_ and it will come up with Bash on Ubuntu on Windows.

### Curious Cat
I was curious on where things where being run from and installed. What I did was installed git using `apt-get install git`

```
bhavik@BHAVIK:/mnt/c/Users/bhavi$ which git
/usr/bin/git
bhavik@BHAVIK:/mnt/c/Users/bhavi$
```
Ok, so it is in the location where I would expect it to be on a Ubuntu machine, however where is this location on my Windows machine. It turns that the Linux file system resides in `C:\Users\<username>\AppData\Local\lxss`

### Caveat
One thing is your mileage may vary since the subsystem is still in Beta, there is a list of issues which you can find on [GitHub](https://github.com/Microsoft/BashOnWindows/issues).

### Conclusion
The great thing about this is, there is no VMs, it is actually Ubuntu on Windows. I imagine that there will be some overhead as all those sys calls going down in to the Windows Subsystem for Linux which then translates it into something the Windows kernel can understand.
