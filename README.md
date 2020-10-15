# ethernet over usb
https://developer.ridgerun.com/wiki/index.php/How_to_use_USB_device_networking

# HPC Cluster Setup

This is a list of things that needs to be done for setting up a cluster to
setup a cluster, running malwares and measure behaviors. This setup uses
[rabbitmq](https://www.rabbitmq.com) to manage the communications between the
master node and the client nodes.

# Private Network
This is the tutorial on network setup. [tutorial](https://wiki.debian.org/DHCP_Server)

Here is an example of how to configure.

/etc/network/interfaces:
```
auto lo
iface lo inet loopback

auto enp1s0
iface enp1s0 inet static
    address 192.168.1.102
    #netmask 255.255.255.0
    #broadcast 192.168.1.255
    #gateway 192.168.1.1
    #dns-nameservers 8.8.8.8
```

/etc/dhcp/dhcpd.conf:
```
# dhcpd.conf
#
# Sample configuration file for ISC dhcpd
#
# Attention: If /etc/ltsp/dhcpd.conf exists, that will be used as
# configuration file instead of this file.
#

# option definitions common to all supported networks...
# option domain-name "analog.com";
# option domain-name-servers ns1.example.org, ns2.example.org;
# option domain-name-servers 8.8.8.8 192.168.1.102;

default-lease-time 600;
max-lease-time 7200;

# The ddns-updates-style parameter controls whether or not the server will
# attempt to do a DNS update when a lease is confirmed. We default to the
# behavior of the version 2 packages ('none', since DHCP v2 didn't
# have support for DDNS.)
# ddns-update-style none;

# If this DHCP server is the official DHCP server for the local
# network, the authoritative directive should be uncommented.
authoritative;

# Use this to send dhcp log messages to a different log file (you also
# have to hack syslog.conf to complete the redirection).
#log-facility local7;

# No service will be given on this subnet, but declaring it helps the
# DHCP server to understand the network topology.

#subnet 10.152.187.0 netmask 255.255.255.0 {
#}

subnet 192.168.1.0 netmask 255.255.255.0 {
  # Specify the default gateway address
  option routers 192.168.1.102;
  # Specify the subnet-mask
  option subnet-mask 255.255.255.0;
  # Specify the range of leased IP addresses
  range 192.168.1.100 192.168.1.200;

  option domain-name-servers 8.8.8.8;

  host rbb1 {
      hardware ethernet 00:02:b5:01:c5:be;
      fixed-address 192.168.1.100;
  }

  # host lab {
  #     hardware ethernet d0:94:66:ee:16:b0;
  #     fixed-address 192.168.1.102;
  # }

  host rbb4 {
      hardware ethernet 00:02:b5:01:c6:3a;
      fixed-address 192.168.1.101;
  }
  host rbb2 {
      hardware ethernet 00:02:b5:01:c6:44;
      fixed-address 192.168.1.103;
  }
  host rbb5 {
      hardware ethernet 00:02:b5:01:c6:3c;
      fixed-address 192.168.1.104;
  }
  host rbb305 {
      hardware ethernet 54:10:ec:93:12:92;
      fixed-address 192.168.1.106;
  }
  host rbb306 {
      hardware ethernet 54:10:ec:93:27:e2;
      fixed-address 192.168.1.107;
  }
host pm3 {
      hardware ethernet 20:B0:F7:04:6B:50;
      fixed-address 192.168.1.108;
  }
}

# This is a very basic subnet declaration.

#subnet 10.254.239.0 netmask 255.255.255.224 {
#  range 10.254.239.10 10.254.239.20;
#  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
#}

# This declaration allows BOOTP clients to get dynamic addresses,
# which we don't really recommend.

#subnet 10.254.239.32 netmask 255.255.255.224 {
#  range dynamic-bootp 10.254.239.40 10.254.239.60;
#  option broadcast-address 10.254.239.31;
#  option routers rtr-239-32-1.example.org;
#}

# A slightly different configuration for an internal subnet.
#subnet 10.5.5.0 netmask 255.255.255.224 {
#  range 10.5.5.26 10.5.5.30;
#  option domain-name-servers ns1.internal.example.org;
#  option domain-name "internal.example.org";
#  option subnet-mask 255.255.255.224;
#  option routers 10.5.5.1;
#  option broadcast-address 10.5.5.31;
#  default-lease-time 600;
#  max-lease-time 7200;
#}

# Hosts which require special configuration options can be listed in
# host statements.   If no address is specified, the address will be
# allocated dynamically (if possible), but the host-specific information
# will still come from the host declaration.

#host passacaglia {
#  hardware ethernet 0:0:c0:5d:bd:95;
#  filename "vmunix.passacaglia";
#  server-name "toccata.example.com";
#}

# Fixed IP addresses can also be specified for hosts.   These addresses
# should not also be listed as being available for dynamic assignment.
# Hosts for which fixed IP addresses have been specified can boot using
# BOOTP or DHCP.   Hosts for which no fixed address is specified can only
# be booted with DHCP, unless there is an address range on the subnet
# to which a BOOTP client is connected which has the dynamic-bootp flag
# set.
#host fantasia {
#  hardware ethernet 08:00:07:26:c0:a5;
#  fixed-address fantasia.example.com;
#}

# You can declare a class of clients and then do address allocation
# based on that.   The example below shows a case where all clients
# in a certain class get addresses on the 10.17.224/24 subnet, and all
# other clients get addresses on the 10.0.29/24 subnet.

#class "foo" {
#  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
#}

#shared-network 224-29 {
#  subnet 10.17.224.0 netmask 255.255.255.0 {
#    option routers rtr-224.example.org;
#  }
#  subnet 10.0.29.0 netmask 255.255.255.0 {
#    option routers rtr-29.example.org;
#  }
#  pool {
#    allow members of "foo";
#    range 10.17.224.10 10.17.224.250;
#  }
#  pool {
#    deny members of "foo";
#    range 10.0.29.10 10.0.29.230;
#  }
#}
```

network address translation:
```
#!/bin/bash
#sudo echo "1" > /proc/sys/net/ipv4/ip_forward
sudo echo "1" > /proc/sys/net/ipv4/ip_forward

sudo iptables --flush
sudo iptables --table nat --flush

sudo iptables --table nat --append POSTROUTING --out-interface wlp2s0 -j MASQUERADE
sudo iptables --append FORWARD --in-interface enp1s0 -j ACCEPT
```

at last, enable dnsmasq


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

7. [Turn off](https://pubs.vmware.com/view-50/index.jsp?topic=%2Fcom.vmware.view.administration.doc%2FGUID-D7921206-FCA5-41FA-9AF5-F481B86463D6.html) windows 7 updates

8. [Show](https://support.microsoft.com/en-us/help/14201/windows-show-hidden-files) all the files

9. [Use AOMEI](http://www.disk-partition.com/) to change partition shrinking size

	1. [Delete](https://support.microsoft.com/en-us/help/920730/how-to-disable-and-re-enable-hibernation-on-a-computer-that-is-running) hiberfil

	2. [Delete](http://www.techentice.com/delete-pagefile-sys-in-windows-7/) the pagefile.sys

	3. [Shrink](https://www.youtube.com/watch?v=cHKS2qG3cNg) Disk beyond the point where any unmovable files are located.

10. Add [credentials](https://www.howtogeek.com/106906/how-to-add-credentials-to-the-windows-credential-manager-vault/) to the windows credential manager vault

11. [Set](https://askubuntu.com/questions/330781/how-to-create-launcher-for-application) up start up scripts

12. path environmental variable
```bat
%SystemRoot%\system32;%SystemRoot%;%SystemRoot%\System32\Wbem;%SYSTEMROOT%\System32\WindowsPowerShell\v1.0\;C:\Program Files\AMD\ATI.ACE\Core-Static;C:\Users\foo\Downloads\php-5.6.30-nts-Win32-VC11-x86;C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Program Files\AMD\ATI.ACE\Core-Static;C:\php;C:\Program Files\Futuremark\PCMark 8\bin;C:\ProgramData\Anaconda2;C:\Program Files\IntelSWTools\VTune Amplifier XE 2017\bin32;C:\ProgramData\Anaconda2;C:\Users\foo\AppData\Local\Continuum\Anaconda2;C:\Users\foo\Anaconda2;C:\Users\foo\Anaconda2\Script
```

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

Licence path:
```batch
C:\Program Files\Common Files\Intel\Licenses
```

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


# boot automatic script

# how to reset transceiver on shutdown/reboot

1. add a script for the system init.d script
```
$ cat /etc/init.d/reset_transceiver
#!/bin/bash
### BEGIN INIT INFO
# Provides: resettransceiver
# Required-Start:
# Required-Stop:
# Default-Start: 2 3 4 5
# Default-Stop: 0 1 6
# Short-Description: restart the transceiver when poweroff and reboot
# Description: restart the transceiver when poweroff and reboot
### END INIT INFO
/home/ranging/ranging_daemon/src/build/reset_transceiver
```

2. add a script for the system service
```
$ cat /etc/systemd/system/reset_transceiver.service
[Unit]
Description=runs only upon shutdown
#Conflicts=reboot.target
After=network.target

[Service]
Type=oneshot
ExecStart=/bin/true
ExecStop=/etc/init.d/reset_transceiver
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
3. Enter the following command to enable the system service
```
sudo chmod +x /etc/init.d/reset_transceiver
sudo systemctl daemon-reload
sudo systemctl enable reset_transceiver --now
```
If everything is successful, you should see the following:
```
Synchronizing state of reset_transceiver.service with SysV init with /lib/systemd/systemd-sysv-install...
Executing /lib/systemd/systemd-sysv-install enable reset_transceiver
```
And you should be able to verify that by entering the commands below.
```
$ ls /etc/rc*.d/*reset_transceiver
/etc/rc0.d/K01reset_transceiver@  /etc/rc2.d/S01reset_transceiver@  /etc/rc4.d/S01reset_transceiver@  /etc/rc6.d/K01reset_transceiver@
/etc/rc1.d/K01reset_transceiver@  /etc/rc3.d/S01reset_transceiver@  /etc/rc5.d/S01reset_transceiver@
```

4. Reboot to test it out!
```
sudo reboot
```
