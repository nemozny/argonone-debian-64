# How to get Argon One v2 fan control working on Raspberry 4B and Debian 64


## Introduction
For some reason none of the existing Argon One fan control scripts worked on a 64-bit OS. The reasons are beyond my comprehension, but I managed to troubleshoot the problem down to I2C.
This guide is based on the Debian 64-bit OS, because I am familiar with Debian since Woody. But I assume the following might be applicable to Raspberry OS 64-bit as well, since it appears to be also Debian.

Don't ask me any details about Raspberry Pi, I bought my first this week (I am writing this). I had some fun with Arduino and Particle Photon a long time ago, though.  
&nbsp;


#### Install Debian 64-bit
For a reference I have used https://raspi.debian.net/tested-images/  
&nbsp;

### Get I2C working
Insert "sudo" where appropriate in this guide.

&nbsp;

#### Check you modules directory
```
# cd /usr/lib/modules/<your kernel version>/kernel/drivers/i2c
# ls & ls busses/
```
You will need "i2c-dev" and in my case also "i2c-bcm2835".  
&nbsp;

#### Install modprobe and i2c tools
`# apt-get install kmod i2c-tools`

&nbsp;

#### Load modules
```
# /usr/sbin/modprobe i2c-bcm2835
# /usr/sbin/modprobe i2c-dev
```   

&nbsp;

#### Now let's scan this thing
First bus
```
# /usr/sbin/i2cdetect -y 0
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         08 09 0a 0b 0c 0d 0e 0f
10: 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f
20: 20 21 22 23 24 25 26 27 28 29 2a 2b 2c 2d 2e 2f
30: -- -- -- -- -- -- -- -- 38 39 3a 3b 3c 3d 3e 3f
40: 40 41 42 43 44 45 46 47 48 49 4a 4b 4c 4d 4e 4f
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: 60 61 62 63 64 65 66 67 68 69 6a 6b 6c 6d 6e 6f
70: 70 71 72 73 74 75 76 77
```


Second bus
```
# /usr/sbin/i2cdetect -y 1
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         08 09 0a 0b 0c 0d 0e 0f
10: 10 11 12 13 14 15 16 17 18 19 1a 1b 1c 1d 1e 1f
20: 20 21 22 23 24 25 26 27 28 29 2a 2b 2c 2d 2e 2f
30: -- -- -- -- -- -- -- 37 38 39 3a 3b 3c 3d 3e 3f
40: 40 41 42 43 44 45 46 47 48 49 4a 4b 4c 4d 4e 4f
50: 50 -- -- -- -- -- -- -- -- 59 -- -- -- -- -- --
60: 60 61 62 63 64 65 66 67 68 69 6a 6b 6c 6d 6e 6f
70: 70 71 72 73 74 75 76 77
```


Third bus
```
# /usr/sbin/i2cdetect -y 2
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```


Fourth bus
```
# /usr/sbin/i2cdetect -y 3
     0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
00:                         -- -- -- -- -- -- -- --
10: -- -- -- -- -- -- -- -- -- -- 1a -- -- -- -- --
20: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
30: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
40: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
50: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
60: -- -- -- -- -- -- -- -- -- -- -- -- -- -- -- --
70: -- -- -- -- -- -- -- --
```


And a fifth bus, for a good measure
```
# /usr/sbin/i2cdetect -y 4
Error: Could not open file '/dev/i2c-4' or '/dev/i2c/4': No such file or directory
```


And voilá! Bus #3 (-y 3) was detected being Argon One.

Why?

You can refer to this page - https://github.com/Argon40Tech/Argon-ONE-i2c-Codes  
&nbsp;
&nbsp;

#### Test the fan
Now, that we know the real bus ID, we can start controlling the fan.
&nbsp;
```
# /usr/sbin/i2cset -y 3 0x01a 0x00
```
This stops the fan (0%).
&nbsp;
```
# /usr/sbin/i2cset -y 3 0x01a 0x64
```
And this sets 100%.
This command is also useful to verify that the script is doing its job - that it will automatically change the value you have set manually.  
&nbsp;


### The script
THE Script. Doesn't work on 64-bit as it is.  
&nbsp;

#### Download the script
```
# wget https://download.argon40.com/argon1.sh
```

Problem is when you try and run the script  
```
# bash argon1.sh
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
E: Unable to locate package raspi-gpio
********************************************************************
Please also connect device to the internet and restart installation.
********************************************************************
```

Vanilla Debian doesn't have the "raspi-gpio" package as Raspberry OS has. It is called "rpi.gpio-common" instead.  
&nbsp;
```
# apt-cache search rpi.gpio
python3-rpi.gpio - Module to control Raspberry Pi GPIO channels (Python 3)
rpi.gpio-common - Module to control Raspberry Pi GPIO channels (common files)
```

You just need to edit the argon1.sh script and replace "raspi-gpio" with "rpi.gpio-common".  
Ultimately it will install its stuff, which I believe you know what to do with.  
&nbsp;

#### Fix the daemon script
Actual work is being done by Python script called argononed.py.
I recommend you copy the script elsewhere (your home), to conduct experiments and to replace the script to its original location later.  
&nbsp;
```
# cd
# cp /usr/bin/argononed.py ./
# cp argononed.py argononed_orig.py
```

Now that we have safely backed up the script, let's run this thing  
&nbsp;

```
# ./argononed.py
Traceback (most recent call last):
  File "./argononed.py", line 16, in <module>
    GPIO.setup(shutdown_pin, GPIO.IN,  pull_up_down=GPIO.PUD_DOWN)
RuntimeError: Mmap of GPIO registers
```
&nbsp;
This GPIO library has issues with 64-bit system, apparently. Or maybe just with Debian.

&nbsp;

#### Workaround
After some (surprisingly brief) googling I found Patrick and this thread - https://bugzilla.redhat.com/show_bug.cgi?id=1780903

His fix indeed worked for me.  
&nbsp;
```
# cd /boot/firmware/
# nano cmdline.txt
```
&nbsp;
I added the "iomem=relaxed" argument, so that my line looked  
&nbsp;
```
console=tty0 console=ttyS1,115200 root=LABEL=RASPIROOT rw fsck.repair=yes iomem=relaxed net.ifnames=0  rootwait.
```
If you have a better fix, then please let us know.  

&nbsp;

#### Reboot.

&nbsp;

#### Test the script again
```
./argononed.py
Exception in thread Thread-1 (shutdown_check):
Traceback (most recent call last):
  File "/usr/lib/python3.11/threading.py", line 1038, in _bootstrap_inner
    self.run()
  File "/usr/lib/python3.11/threading.py", line 975, in run
    self._target(*self._args, **self._kwargs)
  File "./argononed.py", line 22, in shutdown_check
    GPIO.wait_for_edge(shutdown_pin, GPIO.RISING)
RuntimeError: Error waiting for edge
```
&nbsp;

#### Another workaround
The script actually runs 2 threads, one is reading temperature and controlling the fan and the other is waiting for a system shutdown and probably kills the fan.

The "waiting for edge" exception comes from the "shutdown_check" thread, so I just commented it out.  

&nbsp;

And the end of the script you will then have  

```
try:
  # t1 = Thread(target = shutdown_check)
  t2 = Thread(target = temp_check)
  # t1.start()
  t2.start()
except:
  # t1.stop()
  t2.stop()
  GPIO.cleanup()
```
That's it.  

I could not figure out a better fix. Again, anyone please let us know.  
&nbsp;

#### Replace the actual bus ID
The last thing is to replace the actual bus ID with the one we have i2cdetected during our experiments earlier.
Edit the script and replace SMBus(1) with SMBus(3)
```
if rev == 2 or rev == 3:
    bus = smbus.SMBus(1)
```
&nbsp;
change to  
&nbsp;
```
if rev == 2 or rev == 3:
    bus = smbus.SMBus(3)
```  

My "rev" was 3, I think, so you need to change only this line. "Rev" seems to be unrelated to the "bus" number.  

I will upload my script here later.  

&nbsp;

#### Replace the production script
Now you can run the script one last time and if it is not producing any errors, you can replace it.  

```
# cp argononed.py /usr/bin/argononed.py
# systemctl restart argononed
```
&nbsp;

### Configure Debian
We need to make sure that our two I2C modules are being loaded during boot  
&nbsp;
```
echo "i2c-bcm2835" >> /etc/modules-load.d/modules.conf
echo "i2c-dev" >> /etc/modules-load.d/modules.conf
```
&nbsp;
Plus above is the change to /boot/firmware/cmdline.txt.  
&nbsp;

## DONE!
You can test if the script is working by ie. spinning up the fan to 100% and waiting up to 30 seconds for it to stop again.
```
# /usr/sbin/i2cset -y 3 0x01a 0x64  
```
You might need to run the above i2cset 2x for you to hear the fan.
