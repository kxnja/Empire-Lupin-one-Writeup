# Set up

# What we need

1. A laptop or desktop with atleast 8 GB of RAM (The more the better)
2. A stable internet connection to download the necessary files

# Steps

1. Install VirtualBox (https://www.virtualbox.org/) and the kali Linux .ova file for VirtualBox (https://www.kali.org/get-kali/#kali-virtual-machines).
2. Install the Empire : lupin one file from vulnhub (https://www.vulnhub.com/entry/empire-lupinone,750/).
3. Run the VirtualBox file first to set up the virtual machine.
4. Run the kali Linux.ova file second and name it as you please(do not change other settings if not needed).
5. Extract the zipped file and run the Empire_lupinone.ova file that will then open on virtual box.
6. Right click on the kali Linux machine (DO NOT POWER IT ON FIRST) then open settings and go to the network setting and change it to the internal network from NAT (CHANGE THE NETWORK NAME TO INTNET to avoid difficulties later, I'll explain how).Do the same for the Lupin one application. (Change it from bridged adapter to internal network and MUST have the same network name as the kali machine that is INTNET)This is just to make sure they are on the same network.
7. If you do not have a DHCP server already set up you can set it up by ( DHCP server is essential since it assigns IP addresses to your devices (your kali linux machine and lupin one) : 
	a. Open command prompt by typing cmd on the search box on the task bar (on windows though it is fairly similar on mac or linux)
    b. Type in ```cd /Program Files/Oracle/VirtualBox```
   and click enter (I recommend copying and pasting to avoid errors)
    c. type in   ```vboxmanage dhcpserver add --network=intnet --server-ip=10.38.1.1 --lower-ip=10.38.1.110 --upper-ip=10.38.1.120 --netmask=255.255.255.0  --enable```      (I'll explain ,I know it's a lot)
  8. Final step is now going back to start your kali and Lupin one VM (cheers and happy hacking ethically of course )

# Explanation

So essentially what we have done is that we have used the vboxmanage tool to create a dhcp server for our network (that is why I suggested we use the same name to avoid misconfiguring ) we gave the server the IP - 10.38.1.1 and gave it a range of 10.38.1.110 to 10.38.1.120 to assign to connected devices.
    The netmask basically only defines what part is the network and what part is the host (you). We then finish by enabling it . 
