# Step by Step guide to setup NVIDIA Jetson TX2 

This is a step by step guide to set up Nvidia Jetson TX2 with a virtualbox VM in your local machine. There are a couple of things that are required from your side as well. 

### Jetson TX2 box 

- GPU Board
- micro-USB cable 

These two are required to setup. 

### Your side

- wireless keyboard (to connect to Jetson box)
- wireless mouse (to connect to Jetson box)  
You can buy wireless keyboard and mouse combo. I bought <a href="https://www.amazon.com/gp/product/B079JLY5M5/ref=oh_aui_detailpage_o00_s00?ie=UTF8&psc=1" target="_blank">this one</a> from Amazon. 
- Monitor (for HDMI display from Jetson box)
- Ethernet cable (to connect Jetson box to the same router that your local laptop connects to) 

## Background

There are a few things that you need to be familiar with. 

1. Host (Physical device: Ubuntu 16.04) 
2. Target (Physical device: Jetson TX2 GPU processor) 
3. Jetpack (installer software) 

The target Jetson TX2 has no OS installed. So we're going to install Ubuntu 16.04 in Jetson TX2 via Host. 

<p align="center">
<img src="img/flowchart.png" width="800"></p>
<p align="center">Figure. Jetson TX2 and local Host setup</p>

## Creating Host  
 
The host that will flash OS (see below) to Jetson box. In older version of Nvidia TX1, I saw people tested with Ubuntu 14.0. But for TX2, I only tested with Ubuntu 16.04 and it works fine. Host OS is Ubuntu that will download Jetpack (see below) and do all the installation to Jetson TX2. I'm interchangeably using the word Jetson box and Jetson TX2. They're all the same. It's credit size GPU motherboard with all essential ports for ethernet, USB, wi-fi, memory, etc. 

If you already have a local machine or laptop that have Ubuntu, you don't need to go an extra mile setting up the virtual box. If you don't have one like me, you might want to install virtualbox 6.0 in your local laptop instead of buying a new laptop with Ubuntu OS.  

- Download virtualbox and virtual box extension pack (all supported platform) <a href="https://www.virtualbox.org/wiki/Downloads">Here</a>. 
- Install virtualbox 
- Go to Virtualbox --> Preferences... --> Extensions --> Add "New package" button 

#### Note
- **extension pack** = Double clicking extension pack will just open the virtualbox but won't install automatically.  
- **flash OS** means installing OS into the Jetson TX2 (target) 
- **Jetpack** is the software you'd download and install in host and use it to do all the flashing and installing CUDA, tensorflow into the Jetson TX2. 

**Virtualbox**  

- click "New" 
- Name = <Your_Choice>
- Type = Linux  
- Version = Ubuntu 64 
- Memory = 1024 
- VDI (Virtualbox Disk Image) 
- Dynamically allocated 
- File allocation and size = 50 GB (You'd need at least 10 GB minimum) 

Once the VM is created, 

- select your created VM 
- go to VirtualBox Settings 
- `System` --> `Processors` --> 2CPU 
- `Network` --> `Enabled Network Adapter` --> Attached to `Bridged Adapter` 
- `Port` --> `USB` --> `Enabled USB Controller` --> `USB 3.0 (xHCI) Controller` 

#### Note 
Virtualbox extension package allows virtualbox to set up USB as an exclusive USB port. Remember that the micro-USB you're connecting from the Jetson box to your local machine (USB), without the functionality that extension package provides, the **host** VM (virtual machine) that you created in virtualbox will not recognize the USB connection. 

## Install Ubuntu 64 OS into Host 

For now, all you've been doing is creating an image that will have Ubuntu 64 bit with all pre-defined memory and file allowance. But you still don't have the actual Ubuntu OS image that will install Ubuntu into your virtualbox pre-defined VM. 

- Download Ubuntu 64 bit <a href="http://releases.ubuntu.com/16.04/">Here</a> --> `ubuntu-16.04.5-desktop-amd64.iso` 1.5G   

After download is done, 
- select your created Ubuntu VM on the left panel on the virtualbox 
- click **Start** 
- choose the `ubuntu-16.04.5-desktop-amd64.iso` image you just downloaded from the dropdown selection. 
- When the screen asks, click `Install Ubuntu` 

This will install Ubuntu 64 in your VM. The screen display will be cropped out and sometimes you can't see the `next` button. I go by tab and press either space button or return/enter button and it works fine. To make it normal display, you can only do so after completely installing the OS. 

- open terminal in your Host Ubuntu VM 
```
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install dkms
sudo apt-get install build-essential
```
Go to menu bar `Devices` --> `Insert Guest Additions CD Image...`. Follow the instructions in a new window in Host Ubuntu. There are several approaches to make it work. This is what I did and it automatically readjusts the screen size after the installation. Or you can go to `View` --> `Virtual Screen 1` --> `Scale to 200% (autoscaled output)`. 

Or some do by the following command. Don't know if it works

```
sudo apt-get install virtualbox-guest-additions-iso
```
Ref = <a href=https://askubuntu.com/questions/240745/how-do-i-get-a-larger-screen-resolution-in-virtualbox-on-mac-os-x>Source</a>

## Install Jetpack 3.3 

Here I need to explain what's going on behind and some of the issues I got. Downloading and installing Jetpack requires the internet connection. Since our Host VM is using the `Bridged Adapter`, and when the Jetson Box is in `FORCE RECOVERY` mode (Which we will go in details later), it cannot establish the internet connection with `Bridge Adapter`. So instead of first connecting the Jetson TX2 with micro-USB and making the `FORCE RECOVERY` mode, I first downloaded and installed Jetpack in Host without connecting to the Jetson TX2. 

- Open the firefox browser in Host VM 
- Go to <a href="https://developer.nvidia.com/embedded/jetpack">Nvidia developer download site</a> 
- Register the account to download the Jetpack 3.3 

After the download is done, go to Host VM terminal and change to the directory where the Jetpack 3.3 was downloaded. 
```
cd Downloads
chmod +x Jetpack-L4T-3.3-linux-x64_b39.run
./Jetpack-L4T-3.3-linux-x64_b39.run 
``` 
This will launch the Jetpack 3.3 Installer. 

<p align="center">
<img src="img/jetpack.png" width="600"></p>
<p align="center">Figure. Running Jetpack 3.3</p>

- I chose `No, disable usage collection` 
- select Jetson TX2

<p align="center">
<img src="img/jetsontx2.png" width="600"></p>
<p align="center">Figure. Jetson TX2</p>

<p align="center">
<img src="img/jetpack_manager.png" width="600"></p>
<p align="center">Figure. Jetpack component manager</p>

Look carefully what components Jetpack has and it becomes much clearer when it comes to troubleshooting. Jetpack components manager has two major components: (1) `Host - Ubuntu` and (2) `Target - Jetson TX2/TX2i`. Under these components, there are several sub-components that Jetpack will manager to install. You can either customize and select which components to install or just do **full** installation. In the figure above, you see `no action` under `Host - Ubuntu` because I have already installed those components. 

The reason why I mentioned those components is when you have a internet connection issue, you can switch over to `NAT` from `Bridged Adapter` in Network settings in Virtualbox so that Jetpack can download all those components. 

- Click "Next" for full installation 
- If the components are not downloaded properly and error occurs, switch to `NAT` in network setting. You don't need to shut down the Host Ubuntu. Just go to the bottom bar of your **Host** screen, where you'd see two computers icon (probably 4th icon) --> Network setting --> Attached to `NAT`. 
- Once the Jetpack downloaded all the components, it will install all components into the Host. 
- Once it's done, and freeze at the `Flash OS image to Target`, you can now establish the micro-USB connection between the Host and the Jetson TX2 (target). 
- Power off the Jetson (unplug the AC adapter) 
- Plug micro-USB between Jetson and the Host (Your local machine that has the virtualbox installed and Ubuntu VM is running) 
- Power on the Jetson (plug the AC adapter) 
- Press and release the power button 
- Press the `FORCE RECOVERY` button and hold 
- Press and release the `RESET` button while holding the `FORCE RECOVERY` button 
- Wait 2 seconds until finally release the `FORCE RECOVERY` button 

You wouldn't see anything in your Target HDMI display. 

- Go to Host VM terminal
- `lsusb` to check `Nvidia Corp` is listed in USB drive 
- run `./Jetpack-L4T-3.3-linux-x64_b39.run` 
- In components manager selection, from `Target - Jetson TX2/TX2i`, Jetpack will flash OS (Ubuntu 16.04 OS into Jetson TX2 and all the components listed under the major components. 
- If it runs well, it'd be done within a few minutes. 

- If the installation freezes at `Determining the IP address`, you can stop the installation by `Ctrl+C` in Host VM. 
- Reboot the Jetson (Press the `RESET` button) 
Since it now has the OS already installed, you'd see Ubuntu 16.04 version is up and running. 
- Check the IP address of the Jetson (Target) at the far upper right corner, wi-fi connection 
- eth0 and IP address would be `10.0.0.7`. 

#### Note
Some says it's close to `198.168.xx.xx`. In my IP list, it was under `l4tbr0` and it didn't work. 

- Once the IP address is established, Jetpack from Host will install all sub-components (CUDA, TensorRT, etc) into the Jetson.
- You're now finished with a complete installation. 












