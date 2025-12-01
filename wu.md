# Showcase TeamT demo challenge Writeup
We are given a ```.dmp``` file and a ```.txt``` file.
```console
[an2in@an2in chall]$ ls

chall.dmp  Description.txt

[an2in@an2in chall]$ cat Description.txt

Challenge Name: Forensics for newbie

Category: Forensics

Description: Yesterday, my supervisor has sent to my colleague a secret file. After that, I saw his screen was running something with a black background. I think he was did something with that secret file. Can you help me to view that file?

[an2in@an2in chall]$ file chall.dmp

chall.dmp: MS Windows 64bit crash dump, version 15.19045, 4 processors, DumpType (0x1), 1048333 pages
```
The challenge file is a file containing a snapshot of the computer's memory (at the time it is captured). Also, the description said that he saw *a black background* strongly suggest that his supervisor was using ```cmd.exe```.

To solve this challenge, I will use Volatility 3 - the world's most widely used framework for extracting digital artifacts from volatile memory (RAM) samples.

## Some facts to know

Before diving into knowing what command to use. We need to know how some things work.

### How ```cmd.exe``` works

When opening a command prompt, Windows actually launches two programs:

```cmd.exe```: This program is invisible. It takes text commands, calculates the result, and sends text back. It doesn't know how to draw pixels, fonts, or windows.

```conhost.exe```: This is the "Console Window Host." It draws the black box, handles the font, the scrollbar, and keyboard input.

So technically, when I type a command like ```dir```, I am typing it into ```conhost.exe```. Conhost then passes it to ```cmd.exe```.

### The "Up Arrow" Feature
When I opened a cmd, typed a few commands, and then pressed the "Up Arrow" in my keyboard, it brings back the previous commands that I typed (not all but enough).

To make that "Up Arrow" feature work, ```conhost.exe``` must save a list of everything I typed in its own memory. In other words, it keeps a "History Buffer" in the RAM.
## Solve
By utilizing some facts we know, we will use the ```windows.consoles``` plugin, which can looks for Windows console buffers.
```console
(venv) [an2in@an2in chall]$ vol -f chall.dmp windows.consoles
Volatility 3 Framework 2.26.2
Progress:  100.00		PDB scanning finished                                                                                              
PID	Process	ConsoleInfo	Property	Address	Data
[...]
_CONSOLE_INFORMATION.HistoryList.CommandHistory_1_Command_0	0x1fde0d0ab30	cd Desktop
*** 2852	conhost.exe	0x7ff6232e4be0	_CONSOLE_INFORMATION.HistoryList.CommandHistory_1_Command_1	0x1fde0d0ab50	7z a -pKetSoSadisveryfunny -mhe secret.7z secret.png
*** 2852	conhost.exe	0x7ff6232e4be0	_CONSOLE_INFORMATION.HistoryList.CommandHistory_1_Command_2	0x1fde0d0ab70	dir
*** 2852	conhost.exe	0x7ff6232e4be0	_CONSOLE_INFORMATION.HistoryList.CommandHistory_1_Command_3	0x1fde0d0ab90	del secret.png

[...]

(c) Microsoft Corporation. All rights reserved.

C:\Users\ctf-player>cd Desktop

C:\Users\ctf-player\Desktop>7z a -pKetSoSadisveryfunny -mhe secret.7z secret.png

7-Zip 25.01 (x64) : Copyright (c) 1999-2025 Igor Pavlov : 2025-08-03

Scanning the drive:
1 file, 19596 bytes (20 KiB)

Creating archive: secret.7z

Add new data to archive: 1 file, 19596 bytes (20 KiB)


Files read from disk: 1
Archive size: 17074 bytes (17 KiB)
Everything is Ok

C:\Users\ctf-player\Desktop>dir
 Volume in drive C has no label.
 Volume Serial Number is E053-B4C5

 Directory of C:\Users\ctf-player\Desktop

11/22/2025  12:34 AM    <DIR>          .
11/22/2025  12:34 AM    <DIR>          ..
11/22/2025  12:26 AM           530,640 DumpIt.exe
11/22/2025  12:34 AM            17,074 secret.7z
11/21/2025  10:38 PM            19,596 secret.png
               3 File(s)        567,310 bytes
               2 Dir(s)  39,006,892,032 bytes free

C:\Users\ctf-player\Desktop>del secret.png
```
Looking at the result, we can see that Mr.KetSoSad is creating a compressed ```.7z``` file storing some mysterious image named ```secret.png``` with the password ```KetSoSadisveryfunny```.

Now, lets extract that file.
```console
(venv) [an2in@an2in chall]$ vol -f chall.dmp windows.filescan | grep "secret.7z"
0xdd84dba88cd0.0\Users\ctf-player\Desktop\secret.7z
(venv) [an2in@an2in chall]$ vol -f chall.dmp windows.dumpfiles --virtaddr 0xdd84dba88cd0
Volatility 3 Framework 2.26.2
Progress:  100.00		PDB scanning finished                                
Cache	FileObject	FileName	Result

DataSectionObject	0xdd84dba88cd0	secret.7z	file.0xdd84dba88cd0.0xdd84de7e1e30.DataSectionObject.secret.7z.dat
(venv) [an2in@an2in chall]$ mv file.0xdd84dba88cd0.0xdd84de7e1e30.DataSectionObject.secret.7z.dat gotchabi.7z
(venv) [an2in@an2in chall]$ 7z x gotchabi.7z
[...]
Extracting archive: gotchabi.7z
Enter password:KetSoSadisveryfunny
[...]
```
After extracting it, we got our flag.
![Oh yeah bb!](./secret.png)
