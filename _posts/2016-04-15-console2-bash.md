---
layout: post
title: Setup the Windows 10 Insider Preview Bash shell in Console2
byline: How I made a simple problem hard
---

I'm a bit of a bash nut - always have been since back in uni days playing with C++ and cygwin through to more recent times using the shell prompts for the Git (before GitHub for Windows!)

However, I also _love_ the Console2 application which allows a single application to host multiple "shells" - cmd.exe, git, powershell, VS201*, etc.  So naturally once I downloaded the latest Windows 10 insider preview (Anniversay build) I wanted to integrate the bash shell.

But it wasnt as straight forward as I thought.

## Stupidity Reigns....

First step was to find where the application was installed - easy check the start menu

>  %windir%\system32\bash.exe

Second step was to configure a new tab in Console2 - however it didnt work!

![Console2 Error](/images/2016-04-15-console2-error.png "Console2 Error")

So..instead of using my brain and working from first principles through what should be a simple problem - I jumped (in typical fashion) to a silly solution involving symbolic links.  

To be honest, given the root cause of this problem I'm suprised that making a symbolic link actually worked - thats a challenge for another day.  Regardless, I'm not going to detail the zany workaround however I will highlight a few things:

1. The new Bash shell is a 64-bit application
1. The 32-bit version of Command cannot see 64-bit binaries

You see the problem right?  I was trying to launch the shell using the 32-bit version of cmd.exe which can't see the 64-bit bash binary.  

Honestly, I wasn't even aware that a 32 bit command console would not display 64-bit binaries however it obviously makes sense when you think about it.

![32 bit Cmd.exe](/images/2016-04-15-win32-64-cmd.png "32 bit Cmd.exe")

##Wrap up

Sometimes when things dont work, we forget the simple stuff and charge off down silly paths forgetting where we started.  Honestly, if I didnt already have a console tab setup to load the Git bash shell (which obviously is a 32-bit application), I may not have run into the problem.

> Shell: C:\windows\System32\cmd.exe /c "c:\windows\system32\bash.exe"

> Icon: %localappdata%\lxss\bash.ico

![Console2 Bash Working](/images/2016-04-15-console2-working.png "Console2 Working")