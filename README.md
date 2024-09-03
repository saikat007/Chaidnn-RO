# Chaidnn-RO

There are two main folders in this repo. "[DONTCHANGE]_chai-old_wo-out" is the base Chaidnn accelerator codebase (which has already been built) and the second "AT_Test_chai-old_wo-out_RO_NAND" is the custom accelertor setup with Ring Oscillators. All contents can be found here: https://drive.google.com/drive/folders/1uuu-OK5iEBhrmApz7dMvd3Yv02PrcMAM?usp=sharing

# Setup

Generally follow setup instructions from here: https://github.com/Xilinx/CHaiDNN. I am highlighting a few steps below:

1) I have tested the setup on Ubuntu 18.04 and SDSoC v2018.2. The SDSoC environment should be availaible for download from the main Xilinx downloads page. Any other versions did not work for me.
2) Once installed, follow the instructions to [Build from Source](https://github.com/Xilinx/CHaiDNN?tab=readme-ov-file#run-inference). First generate the custom platform following the instructions here: https://github.com/Xilinx/CHaiDNN/blob/master/docs/CUSTOM_PLATFORM_GEN.md. The instructions there are for ZCU102, so make sure you replace everything with the board you are using (in this case ZCU104). The global variables I used are as follows:
```
set board_name "zcu104"
set part_name "xczu7ev-ffvc1156-2-e"
set board_part_name "xilinx.com:zcu104:part0:1.1"
set proj_path "."
```
3) Once the clock wizard is setup, follow the instructions from step 2 and 3 in the main CHaiDNN repo (linked above). I used BASH, the command I ran to set the SDx tool is:
```
source ~/Desktop/SDx/2018.2/settings64.sh
```
4) To build the hardware, goto the ```<main-folder>/design/build``` folder in terminal. Edit the Makefile with the correct Paths. From the terminal, run: ```make POOL_ENABLE=0 DECONV_ENABLE=0 -j4``` to start building the hardware. Note that you will need the correct licenses for the SDSoC tool, which should also be availaible once you get the software. Depending on your system this might take sometime to finish.
5) Once finished, run ```make copy``` and copy the ```libxlnxdnn.so``` file to ```<main-folder>/SD_Card/lib```. Now build the software from ```<main-folder>/software/scripts```. Change any directory listed in the Makefile to appropriate locatons.
6) The software can be build using the same command: ```make POOL_ENABLE=0 DECONV_ENABLE=0 -j4``` from the terminal.
7) Now copy the contents from the ```SD_Card``` folder to an actual SD Card. Also copy the ```models``` folder to that same SD Card.
8) Insert this SD Card in the FPGA, power on the board and connect it to a host PC using a USB/UART Cable. You should be able to issue commands here using a serial controller. Enter the following for the UART Serial port:
```
Baud rate: 115200
Data: 8 bit
Parity: none
Stop: 1 bit
Flow control: none
```
9) Once the boot sequence is finished enter the following in the serial terminal:
```
export OPENBLAS_NUM_THREADS=2
export LD_LIBRARY_PATH=lib/:opencv/arm64/lib/:protobuf/arm64/lib:cblas/arm64/lib
```

10) The model inference can now be run using: ```./vgg.elf Xilinx 8 images/camel.jpg images/goldfish.JPEG```. This accelerator runs inference on the ImageNet dataset and the results will also be saved in the ```models/<network>/out``` folder, with the Class ID and the confidence value. You can cross-refernce the ImageNet Class IDs form here: https://gist.github.com/yrevar/942d3a0ac09ec9e5eb3a
11) Note that the "Do not Change" folder already has the required contents needed in the SD_Card folder, so you do not have to build that again. Only the one with ROs needs to be built. CHaiDNN does a HW/SW partitioning for performance (https://github.com/Xilinx/CHaiDNN/blob/master/docs/HW_SW_PARTITIONING.md). So, mostly the convolution layers are executed in the FPGA fabric based on calls made by the SW component. In this sample project, the ROs will be triggerred whenever these convolution layers are executed (in case of VGG used here, there'll be 13 Convolution layers) . You might need to change the frequency of the circuit and the supply voltage of the board to generate the fault effects.
12) For changing the supply voltage, insert the Infineon USB005 voltage controller in a host system, and connect the I2C component to the FPGA board. You will need to install the PowerIRCenter in your host machine to control the supply voltage. Follow the guide here: https://www.infineon.com/dgdl/an-0035.pdf?fileId=5546d462533600a40153559088d40f2d. Based on the ZCU104 board power system: https://docs.amd.com/v/u/en-US/ug1267-zcu104-eval-bd, you will only need to play around with the I2C address: 0x13 power controller. As fas as I know the PowerIRCenter only works in Windows 10. I have attached the relevant installers for this. From what I have seen the FPGA board will crash if the supply voltage is set below 0.648V.
