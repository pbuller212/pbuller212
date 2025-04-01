I recently bought a Nucleo ARM Development board. It has been a long time since I did this type of work, especially outside of Arduino style hardware. This seemed like a way to challenge myself and learn something. 

I use Windows with WSL Linux, Ubuntu 22.04 to be clear. I tried the STM Workbench and the VS Code extensions, but was not able to get these to work. I am not sure if there is some kind of conflict with this hybrid setup under WSL, but I did not go too far into it and instead I am using a more command line driven approach after finding this [set of instructions.](https://github.com/davisjp1822/stm32_nucleo_linux)

Here is what I had to do:
1. On WSL, install the required dependancies:
```
sudo apt install gcc-arm-none-eabi gdb-arm-none-eabi binutils-arm-none-eabi openocd screen
```
2. Download and unzip en.stm32cubef4-v1-28-0.zip from [here.](http://www.st.com/content/st_com/en/products/embedded-software/mcus-embedded-software/stm32-embedded-software/stm32cube-embedded-software/stm32cubef4.html)
```
unzip en.stm32cubef4-v1-28-0.zip
cd STM32Cube_FW_F4_V1.28.0/
```
3. Install [usbipd](https://github.com/dorssel/usbipd-win/releases) on Windows. This is required so that the Ubuntu container can see the serial port.
3. Configuring the Windows so that the Ubuntu container can see the device. Here, you need to open up Powershell with Administrator privileges and have your board plugged into your computer.    
    a. Find the bus ID by running ```usbipd list```. You can then see your device and its bus ID. For me, it was 2-2.
    ```
    PS C:\WINDOWS\system32> usbipd list
    Connected:
    BUSID  VID:PID    DEVICE                                                        STATE
    2-1    062a:4102  USB Input Device                                              Not shared
    2-2    0483:374b  ST-Link Debug, USB Mass Storage Device, STMicroelectronic...  Not shared
    2-3    2a94:5609  USB Input Device                                              Not shared
    2-4    046d:c534  USB Input Device                                              Not shared
    2-5    5986:2130  Integrated Camera                                             Not shared
    2-10   8087:0026  Intel(R) Wireless Bluetooth(R)                                Not shared
    ```
    b. Bind the Bus ID by running ```usbipd bind --busid 2-2```, where the 2-2 is what you gatherd from the step above. You should then be able to see that the device is now Shared.
    ```
    PS C:\WINDOWS\system32> usbipd bind --busid 2-2
    PS C:\WINDOWS\system32> usbipd list
    Connected:
    BUSID  VID:PID    DEVICE                                                        STATE
    2-1    062a:4102  USB Input Device                                              Not shared
    2-2    0483:374b  ST-Link Debug, USB Mass Storage Device, STMicroelectronic...  Shared
    2-3    2a94:5609  USB Input Device                                              Not shared
    2-4    046d:c534  USB Input Device                                              Not shared
    2-5    5986:2130  Integrated Camera                                             Not shared
    2-10   8087:0026  Intel(R) Wireless Bluetooth(R)                                Not shared
    ```
    c. Attach the Bus ID to wsl by running ```usbipd attach --wsl --busid 2-2```. Your container should now be able to see the board when plugged into your computer.
    ```
    PS C:\WINDOWS\system32> usbipd attach --wsl --busid 2-2
    usbipd: info: Using WSL distribution 'Ubuntu' to attach; the device will be available in all WSL 2 distributions.
    usbipd: info: Detected networking mode 'nat'.
    usbipd: info: Using IP address 172.29.144.1 to reach the host.
    ```
3. Finding the serial port the board is using. After plugging in the device, type in ```dmesg``` at your linux prompt and the output will be something like:
```
[121172.078858] vhci_hcd vhci_hcd.0: devid(131074) speed(2) speed_str(full-speed)
[121172.079828] vhci_hcd vhci_hcd.0: Device attached
[121172.373269] vhci_hcd: vhci_device speed not set
[121172.454866] usb 1-1: new full-speed USB device number 3 using vhci_hcd
[121172.536286] vhci_hcd: vhci_device speed not set
[121172.607612] usb 1-1: SetAddress Request (3) to port 0
[121172.647966] usb 1-1: New USB device found, idVendor=0483, idProduct=374b, bcdDevice= 1.00
[121172.647973] usb 1-1: New USB device strings: Mfr=1, Product=2, SerialNumber=3
[121172.647974] usb 1-1: Product: STM32 STLink
[121172.647976] usb 1-1: Manufacturer: STMicroelectronics
[121172.647977] usb 1-1: SerialNumber: 066CFF484971754867024738
[121172.661115] cdc_acm 1-1:1.2: ttyACM0: USB ACM device
```
What your are looking for is the ttyACM0, or something similar.