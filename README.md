# HPC Cluster Setup

This is a list of things that needs to be done for setting up a cluster to
setup a cluster, running malwares and measure behaviors. This setup uses
[rabbitmq](https://www.rabbitmq.com) to manage the communications between the
master node and the client nodes.

# Configuration of Master Node

# Installation of OS

1. Get Windows from [Microsoft Imagine](https://imagine.microsoft.com).

2. Create [Ubuntu bootable USB disk](https://tutorials.ubuntu.com/tutorial/tutorial-create-a-usb-stick-on-ubuntu#0)

# Configurations on Windows

1. Installation of network driver and other drivers. (This varies according to
the machine configuration.)

2. Set the windows [never
sleep](https://answers.microsoft.com/en-us/windows/forum/windows_7-performance/how-do-i-stop-my-computer-from-going-to-sleep/c24555c0-3d73-e011-8dfc-68b599b31bf5?auth=1)
and [autologin
windows](http://www.online-tech-tips.com/windows-7/configure-auto-login-for-windows-7-domain-or-workgroup-pc).

3. Map the drive to the network drive in _My Computer_ and set [autologin](https://www.techrepublic.com/blog/windows-and-office/manage-network-logon-credentials-in-microsoft-windows/) for network drive.

4. Install [softwares](#softwares) and [benchmarks](#benchmarks)

5. Put startup scripts in the folder C:\Users\<user name>\AppData\Roaming\Microsoft\Windows\Start Menu\Programs\Startup

6. Always be [admin](https://www.technipages.com/windows-enable-disable-user-account-control-uac)

# Configurations on Ubuntu

Ubuntu partition here only achieve the reloading of windows partition, which copy
the windows backup image to the location of windows partition. 

1. Copy the windows partition to an image file. 
```bash
sudo dd if=/dev/sda1 of=windows.img bs=4k status=progress
# sda1 is the windows ntfs partition
```
2. Configure the grub.

In order to achieve alternative booting sequence, we need to replace the
assignment of _default\_entry_ with the following lines in the
_/boot/grub/grub.cfg_ file.

```bash
load_env
if [ "${bob_entry}" -eq "0" ]; then
	set default_entry=5 # 5 refers to the windows entry
	set bob_entry=5
else
	set default_entry=0 # 0 refers to the ubuntu entry
	set bob_entry=0
fi
save_env bob_entry
```
Make sure _bob\_entry_ is in _/boot/grub/grubenv_.

3. Auto-reload the windows partition.
```bash
sudo dd of=/dev/sda1 if=windows.img bs=4k status=progress
# sda1 is the windows partition.
```
In the startup application, add the script to the list.

4. Make sure the ubuntu does not sleep, autologin, and no password to use sudo.
* Not sleeping setup is in the system power configurations.

* [Auto login](https://help.ubuntu.com/community/AutoLogin) explains how to
autologin. While installation of Ubuntu, the configuration provides autologin.

* [No password for
sudo](https://askubuntu.com/questions/192050/how-to-run-sudo-command-with-no-password)
explains no password for sudo setup.

# Softwares


- [x]
[Adobe PDF Reader](https://get.adobe.com/reader/otherversions/)

- [x]
[SPEC](https://www.spec.org/cpu2006/docs/install-guide-windows.html) 
explains how to run SPEC2006 benchmark on a windows machine.

- [x]
[Chrome](https://www.google.com/chrome/browser/desktop/index.html?brand=CHBD&gclid=Cj0KCQjw1PPJBRDqARIsABNl38GnxS-mJGET70epjFJ-S0eiiIhbRn8eCd4jZrqXuhFAE4aiMofOuqEaAohyEALw_wcB)

- [x]
[Visual Studio](https://www.visualstudio.com/)

- [x]
[Anaconda](https://www.continuum.io/downloads) After the installation, do:
```bash
 pip install pika 
```
to install rabbitmq interface on python in windows. 

# Benchmarks
## Opensource Benchmarks
- [ ]
AIM Multiuser Benchmark: composed of a list of tests that could be mixed to
create a ‘load mix’ that would simulate a specific computer function on any
**UNIX-type OS**.  Bonnie++: filesystem and hard drive benchmark

- [ ]
BRL-CAD: cross-platform architecture-agnostic benchmark suite based on
multithreaded ray tracing performance; baselined against a VAX-11/780; and used
since 1984 for evaluating relative CPU performance, compiler differences,
optimization levels, coherency, architecture differences, and operating system
differences. **BRL-CAD only works on 64-bit machines.**

- [ ]
DEISA Benchmark Suite: scientific HPC applications benchmark **European Union
driven scientific reserch benchmark, no body use this**

- [ ]
Dhrystone: integer arithmetic performance, often reported in DMIPS (Dhrystone
millions of instructions per second) **Synthetic computing benchmark**

- [ ]
Fhourstones: an integer benchmark **Required GNU gcc**

- [ ]
HINT: designed to measure overall CPU and memory performance **HINT is designed
for UNIX**

- [ ]
Iometer: I/O subsystem measurement and characterization tool for single and
clustered systems.

- [ ]
IOzone: Filesystem benchmark

- [ ]
LINPACK benchmarks: traditionally used to measure FLOPS

- [ ]
Livermore loops

- [ ]
NAS parallel benchmarks

- [ ]
NBench: synthetic benchmark suite measuring performance of integer arithmetic,
memory operations, and floating-point arithmetic

- [ ]
PAL: a benchmark for realtime physics engines

- [ ]
PerfKitBenchmarker: A set of benchmarks to measure and compare cloud offerings.

- [ ]
POV-Ray: 3D render

- [ ]
Tak (function): a simple benchmark used to test recursion performance

- [ ]
TATP Benchmark: Telecommunication Application Transaction Processing Benchmark

- [ ]
TPoX: An XML transaction processing benchmark for XML databases

- [ ]
VUP (VAX unit of performance), also called VAX MIPS Whetstone: floating-point
arithmetic performance, often reported in millions of Whetstone instructions
per second (MWIPS)


## Windows Specific Benchmarks
- [ ]
BAPCo: MobileMark, SYSmark, WebMark, **895 dolloars per suite. I have reached
them for educational version.**

- [x]
Futuremark: 3DMark, PCMark **We have a free license, and they works.** [Command
Line
Reference](http://s3.amazonaws.com/download-aws.futuremark.com/PCMark8-CommandLineGuide.pdf)
Futuremark includes the tests for office and adobe. Adobe does not have
educational version.


- [ ]
Whetstone **Designed for ARM**

- [ ]
Worldbench (discontinued) **No proper download**

- [x]
PiFast **Works** [pifast](http://numbers.computation.free.fr/Constants/PiProgram/pifast.html)

- [ ]
SuperPrime **Not working**

- [x]
Super PI Windows System Assessment Tool, included with Windows Vista and later
releases, providing an index for consumers to rate their systems easily **works**

- [x]
[phoronix](https://github.com/phoronix-test-suite/phoronix-test-suite) is
tunned for 64 bit Windows. But some suites of Phoronix can also run under 32
bit Windows. The way to run phoronix is download 
[PHP](http://windows.php.net/download/) and use the _php.exe_ to run
phoronix-test-suite.bat.

- [ ]
[selenium](http://selenium-python.readthedocs.io/)
python web  browsing

# VTunes Measurement

Since we are using VTunes on 32-bit windows machine, the measurements can only
be done through command line. The command line reference of VTunes is [this
link](https://software.intel.com/en-us/node/544220).

Collect data:
```batch
amplxe-cl.exe -collect hotspots -result-dir r001hs -quiet application_you_want_to_run.exe
```
Read data:
```batch
amplxe-cl.exe -report hotspots -r r001hs 
```
Relog command can read vtune data base and reformat the data.
We use [hardware event-based
sampling](https://software.intel.com/en-us/node/609450) for sampling with a
sampling interval of 1ms.

# Remote Connection

We use
[rdesktop](http://www.linuxquestions.org/questions/linux-general-1/how-to-install-rdesktop-on-ubuntu-622949/)
to connect to windows machines.

We use
[ssh-pass](https://www.cyberciti.biz/faq/noninteractive-shell-script-ssh-password-provider/)
ssh to connect to linux machines.
