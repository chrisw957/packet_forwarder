
# Fork of Lora network Packet forwarder

This is a fork of https://github.com/Lora-net/packet_forwarder

This version sends received lora packets to udp port 1680 and port 1681.

1681 is where nitrodash will listen and show the packets on the UL Packets screen.

This fork is intended to be used with the fork of the lora_gateway library at https://github.com/chrisw957/lora_gateway.

## Install lora_gateway fork
<pre>
cd ~
git clone https://github.com/chrisw957/lora_gateway.git
cd lora_gateway
make
<pre>

## Install packet_forwarder fork
<pre>
cd ~
git clone https://github.com/chrisw957/packet_forwarder.git
cd packet_forwarder
make
</pre>

## Install nitrodash
Follow the readme here:
https://github.com/chrisw957/nitrodash/blob/master/README.md

## Running packet_forwarder
First, make sure you have gpio access as shown below.

Then, run reset_lgw.sh script:
<pre>
cd ~/lora_gateway
./reset_lgw.sh start
</pre>

Then, launch the packet forwarder:
<pre>
cd ~/packet_fowarder/lora_pkt_fwd
./lora_pkt_fwd
</pre>
You should see packets on nitrodash's UL Packets screen, and on the terminal from lora_pkt_fwd:
<pre>
INFO: Received pkt from mote: 20000450 (fcnt=20329)

JSON up: {"rxpk":[{"tmst":10098636,"chan":0,"rfch":0,"freq":902.300000,"stat":1,"modu":"LORA","datr":"SF10BW125","codr":"4/5","lsnr":8.2,"rssi":-34,"size":12,"data":"QFAEACCAaU/SEYst"}]}
</pre>

## GPIO Access
For this all to work, we need to be able to toggle gpio 103.

Initially, the /sys/class/gpio entries will only be accessible by root.

Initially, there is also no "gpio" group. We can create a gpio group:
<pre>
ubuntu@bionic-dev64:~$ sudo addgroup gpio                                       
Adding group `gpio' (GID 1002) ...                                              
Done.
</pre>
Now we need to add the group to the user.  In this case, we will add gpio to groups that the user named "Ubuntu" is in.  For this to take effect, the user needs to logout and log back in.
<pre>
ubuntu@bionic-dev64:~$ sudo usermod -a -G gpio ubuntu  
ubuntu@bionic-dev64:~$ exit  
(log back in...)
ubuntu@bionic-dev64:~$ groups                                                   
ubuntu adm dialout cdrom sudo audio dip video plugdev weston-launch gpio 
</pre>
We can then change the group of /sys/class/gpio/export and unexport to gpio with the chown command:
<pre>
ubuntu@bionic-dev64:~$ sudo chown root:gpio /sys/class/gpio/export              
ubuntu@bionic-dev64:~$ sudo chown root:gpio /sys/class/gpio/unexport            
</pre>
We can also change the permissions with the chmod command:
<pre>
ubuntu@bionic-dev64:~$ sudo chmod 770 /sys/class/gpio/unexport                  
ubuntu@bionic-dev64:~$ sudo chmod 770 /sys/class/gpio/export 
</pre>
We also need a udev rule that runs when a new device is created.  This will make sure the new entries under /sys/class/gpio also get the right group and permissions.  Do this by creating a new file: /etc/udev/rules.d/99-gpio.rules

It should have one line:
<pre>
SUBSYSTEM=="gpio*", PROGRAM="/bin/sh -c 'find -L /sys/class/gpio/ -maxdepth 2 -exec chown root:gpio {} \; -exec chmod 770 {} \; || true'"
</pre>
Now restart udev:
<pre>
$ sudo /etc/init.d/udev restart
</pre>
Now the user ubuntu can export gpio 103...
<pre>
ubuntu@bionic-dev64:~$ echo 103 > /sys/class/gpio/export 
ubuntu@bionic-dev64:~$ ls -alg /sys/class/gpio/gpio103                          
lrwxrwxrwx 1 root 0 Oct 23 18:21 /sys/class/gpio/gpio103 -> ../../devices/platform/30230000.gpio/gpiochip3/gpio/gpio103  
</pre>
And the user ubuntu can access the entries as normal:
<pre>
ubuntu@bionic-dev64:~$ cd /sys/class/gpio/gpio103                               
ubuntu@bionic-dev64:/sys/class/gpio/gpio103$ echo out > ./direction             
ubuntu@bionic-dev64:/sys/class/gpio/gpio103$ echo 1 > ./value                   
ubuntu@bionic-dev64:/sys/class/gpio/gpio103$ echo 0 > ./value                   
ubuntu@bionic-dev64:/sys/class/gpio/gpio103$ 
</pre>

cd ~/lora_gateway
./reset_lgw.sh start
cd ~/packet_forwarder
cd lora_pkt_fwd
./lora_pkt_fwd

