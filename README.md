# FPGA_Troubleshoot_Linux(REHL 8.8)
Solutions to problems I encountered while trying to setup Vivado and Vitis to work with the SmartLynq -> JTAG -> PicoZed (xc7z030) + FMC Carrier V2

Notes: Make sure you have an AMD account already, it will ask you to login to download the AMD Unified Installer. If not, you can sign up for one -> https://www.amd.com/en/registration/create-account.html

**Problem: Downloading the AMD Unified Installer**
1. Download the AMD Unified Installer for FPGAs & Adaptive SoCs 2023.1 (Includes Vivado, Vitis and other resources) from https://www.xilinx.com/support/download.html
2. I tried to download the full installer (AMD Unified Installer for FPGAs & Adaptive SoCs 2023.1 SFD (TAR/GZIP - 110.85 GB)) but did not worked, most likely network instability (Internet connection disconnected every now and then).
**Solution**
Download the AMD Unified Installer for FPGAs & Adaptive SoCs 2023.1: Linux Self Extracting Web Installer (BIN - 265.94 MB). Also download the Digests, Signature and Public Key, they will help you to sign and verify the integrity of the installer. Now just follow the instructions provided by AMD -> https://docs.xilinx.com/r/en-US/ug973-vivado-release-notes-install-license/Download-and-Installation

**Problem: Vivado can't connect to the SmartLynq**

**Solution**
Open a terminal and list the USB devices using `lsusb`:
   
`[username@my_computer~]$ lsusb
Bus 004 Device 001: ID xxxx:000X Linux Foundation 3.0 root hub
Bus 003 Device 006: ID xxxx:x0XX Logitech, Inc. M105 Optical Mouse
Bus 003 Device 005: ID xxxx:XX0X Dell Computer Corp. QuietKey Keyboard
Bus 003 Device 015: ID xxxx:00XX Xilinx, Inc.   <----------------
Bus 003 Device 003: ID xxxx:XXX0 Huasheng Electronics 
Bus 003 Device 007: ID xxxx:0XxX Logitech, Inc. 
Bus 003 Device 004: ID xxxx:00XX Intel Corp. AX201 Bluetooth
Bus 003 Device 001: ID xxxx:000X Linux Foundation 2.0 root hub
Bus 002 Device 002: ID xxxx:XXXX Realtek Semiconductor Corp. RTL8153 Gigabit Ethernet Adapter
Bus 002 Device 001: ID xxxx:000X Linux Foundation 3.0 root hub
Bus 001 Device 001: ID xxxx:000X Linux Foundation 2.0 root hub`

If the device is not listed, install the drivers, follow the instructions -> https://docs.xilinx.com/r/en-US/ug973-vivado-release-notes-install-license/Install-Cable-Drivers.
Run the `lsusb` command in the terminal again. If not listed reestart the PC and run the command once more. If still not showing then the physical cable (try changing the cable that connects the SmartLynq to the PC) might be the problem.
...
If the Xilinx, Inc. is listed, then run the `ifconfig -a` command in the terminal and make sure you see a network interface in the 10.0.0.x range, where x = 0 to 254:

```[username@my_computer ~]$ ifconfig -a
enp0s13f0u2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether xx:xx:xx:xx:xx:xx txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
enp0s20f0u7u1: flags=XXXX<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 5732  bytes 1558684 (1.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
enp88s0: flags=XXXX<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet xxx.xxx.xxx.xxx  netmask xxx.xxx.xxx.xxx  broadcast xxx.xxx.xxx.xxx
        inet6 xxxx::xxxx:xxxx:xxxx:xxxx  prefixlen 64  scopeid 0x20<link>
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 35012007  bytes 17836937615 (16.6 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 30229238  bytes 20993543473 (19.5 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device memory 0x6a200000-6a2fffff  
lo: flags=XXXX<UP,LOOPBACK,RUNNING>  mtu 65536
        inet xxx.xxx.xxx.xxx netmask xxx.xxx.xxx.xxx
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 14536321  bytes 5713108928 (5.3 GiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 14536321  bytes 5713108928 (5.3 GiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
virbr0: flags=XXXX<UP,BROADCAST,MULTICAST>  mtu 1500
        inet xxx.xxx.xxx.xxx  netmask 255.255.255.0  broadcast xxx.xxx.xxx.xxx
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
wlo1: flags=XXXX<BROADCAST,MULTICAST>  mtu 1500
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        
Where xx:xx:xx:xx:xx:xx is a valid MAC address and xxx.xxx.xxx.xxx a valid IP Address.
If you do not see any Network Interface with IP Address in the 10.0.0.x range as the example above, disconnect it and run the command `ifconfig -a` again. Notice that one of the network interfaces will not be listed this time, well that is the one you need to modify and set it to the required network range (10.0.0.x). So connect the SmartLynq again and run the command `sudo ifconfig name_of_the_network_interface 10.0.0.1 netmask 255.255.255.0` (you are going to need root privilage to do it):

`[root@my_computer ~]# sudo ifconfig enp0s20f0u7u1 10.0.0.1 netmask 255.255.255.0 <- In this case the name_of_the_network_interface = enp0s20f0u7u1`

Now run the command `ifconfig -a` one more time to verify the change. You should see the IP Address (10.0.0.1) listed as part of the Network Interface (enp0s20f0u7u1 in this example):

```[root@my_computer ~]# ifconfig -a
enp0s13f0u2: flags=XXXX<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
enp0s20f0u7u1: flags=XXXX<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.0.0.1  netmask 255.255.255.0  broadcast 10.0.0.255   <----------------
        ether xx:xx:xx:xx:xx:xx  txqueuelen 1000  (Ethernet)
        RX packets 5766  bytes 1568955 (1.4 MiB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 15  bytes 1521 (1.4 KiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
