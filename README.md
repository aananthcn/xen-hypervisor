# xen-hypervisor

## Steps to build
 * `git clone https://github.com/aananthcn/xen-hypervisor.git`
 * `cd xen-hypervisor/build`
 * `source ../yocto/poky/oe-init-build-env .`
 * `bitbake xen-image-minimal`

<br>

 ## Steps to flash the image
  * The image to flash is `xen-hypervisor/build/tmp/deploy/images/raspberrypi4-64/xen-image-minimal-raspberrypi4-64.rootfs.wic.bz2`
  * After flashing copy the binary `xen-hypervisor/build/tmp/deploy/images/raspberrypi4-64/xen-raspberrypi4-64` as `xen` into the `\boot` partition of the SD Card (after the step above).

<br>

## Steps to connect the Serial console
 * Check if the following flags/variables are set as below in `config.txt`
```
# Enable UART for serial console
enable_uart=1

# Disable Bluetooth to free UART0 for the console (if needed)
dtoverlay=disable-bt

# Optional: Ensure UART clock consistency
core_freq=250
```
<br>

## Running Zephyr RTOS as Domain-U in XEN Hypervisor
Running Zephyr RTOS on Raspberry Pi 4 requires compiling the Zephyr kernel for ARM architecture. Download the Zephyr SDK from their official site. After extracting it, run the setup shell script which will prompt you to install additional dependency tools such as CMake and Python 3’s pip:

bash
```
echo $ZEPHYR_SDK_INSTALL_DIR
export ZEPHYR_SDK_INSTALL_DIR=~/<zephyr-sdk-directory>
echo 'export ZEPHYR_SDK_INSTALL_DIR=~/zephyr-sdk-0.16.8' >> ~/.bashrc
source ~/.bashrc
mkdir -p ~/zephyrproject
cd ~/zephyrproject
west init
```

Next, update your environment:
```
bash
west update
source zephyr/zephyr-env.sh
west build -b xenvm zephyr/samples/synchronization
```
The built zephyr.bin file will be located in <project_directory>/build/zephyr/.... When building for Raspberry Pi 4, remember that it supports only GIC v2; GIC v3 kernel builds won’t work.

Creating conf file for Zephyr
Create a file called zephyr.conf with this content:

text
```
kernel="zephyr.bin"
name="zephyr"
vcpus=1
memory=16
gic_version="v2"
on_crash="preserve"
```

Copy both zephyr.bin and zephyr.conf files to the root directory of the Xen hypervisor. Then create an instance of Zephyr RTOS VM:

bash
```
xl create -c zephyr.conf
```


# References
 * https://pmsquaresoft.com/running-zephyr-rtos-on-raspberry-pi-4-with-xen/