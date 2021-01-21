## Based on zcu104 sturcture, You can construct the simliar block diagram on zu6eg<br /><br />

## Create the Vivado Hardware Component and Generate XSA<br /><br />
1. Source <Vitis 2020.2_Install_Directory>/settings64.sh, and run the Vivado by typing "vivado" in the console.<br />
2. Create a Vivado project named zu6eg_custom_platform .<br />
   a) Select ***File->Project->New***.<br />
   b) Click ***Next***.<br />
   c) In Project Name dialog set Project name to ```zu6eg_custom_platform```.<br />
   d) Click ***Next***.<br />
   e) Leaving all the setting to default until you goto the Default Part dialog.<br />
   f) Select ***Parts*** tab and then select custom part ***zu6eg-ffvc900-2***<br />
   g) Click ***Next***, and your project summary should like below:<br />
   ![vivado_project_summary.png](/pic_for_readme/project_summary_6eg.png)<br />
   h) Then click ***Finish***<br />
3. Create a block design named system. <br />
   a) Select Create Block Design.<br />
   b) Change the design name to ```system```.<br />
   c) Click ***OK***.<br />
4. Add MPSoC IP and run block automation to configure it.<br />
   a) Right click Diagram view and select ***Add IP***.<br />
   b) Search for ```zynq``` and then double-click the ***Zynq UltraScale+ MPSoC*** from the IP search results.<br />
   c) Set the mpsoc configuration to meet your specification on your board

5. Re-Customizing the Processor IP Block<br />
   a) Double-click the Zynq UltraScale+ MPSoC block in the IP integrator diagram.<br />
   b) Select ***Page Navigator > PS-PL Configuration***.<br />
   c) Expand ***PS-PL Configuration > PS-PL Interfaces*** by clicking the ***>*** symbol.<br />
   d) Expand Master Interface.<br />
   e) Uncheck the AXI HPM0 FPD and AXI HPM1 FPD interfaces.<br />
   f) Click OK.<br />
   g) Confirm that the IP block interfaces were removed from the Zynq UltraScale+ MPSoC symbol in your block design.<br />
   ![hp_removed.png](/pic_for_readme/hp_removed.png)<br />
   h) [optional] If you use custom board, you should do another settings(I/O , Clock, ddr4) to meet with board condition.
  
***Note 1: This is a little different from traditional Vivado design flow. When trying to make AXI interfaces available in Vitis design you should disable these interfaces at Vivado IPI platform and enable them at platform interface properties. The following steps will show you how to do that later***<br><br />

***Note 2: For custom board, you can refer to DB2785 or DB2857***<br><br />

6. Add clock block:<br />
   a) Right click Diagram view and select ***Add IP***.<br />
   b) Search for and add a Clocking Wizard from the IP Search dialog.<br />
   c) Double-click the clk_wiz_0 IP block to open the Re-Customize IP dialog box.<br />
   d) Click the Output Clocks tab.<br />
   e) Enable clk_out1 through clk_out3 in the Output Clock column, rename them as ```clk_100m```, ```clk_300m```, ```clk_600m``` and set the Requested Output Freq as follows:<br />
      - clk_100m to ```100``` MHz.<br />
      - clk_300m to ```300``` MHz.<br />
      - clk_600m to ```600``` MHz.<br />

   f) At the bottom of the dialog box set the ***Reset Type*** to ***Active Low***.<br />
   g) Click ***OK*** to close the dialog.<br />
  The settings should like below:<br />
  ![clock_wizard.png](/pic_for_readme/clock_wizard.png)<br />
***Note: So now we have set up the clock system for our design. This clock wizard use the pl_clk as input clock and geneatate clocks needed for the whole logic design. In this simple design I would like to use 100MHz clock as the axi_lite control bus clock, 300MHz clock as DPU AXI interface clock and 600MHz as DPU core clock. You can just modifiy these clocks as you like and remember we should "tell" Vitis what clock we can use. Let's do that later.(And after creating this example I learn that the Vitis AI DPU can only have 2 clock domains and the axi_lite control bus and DPU AXI interface share same clock. So the 100MHz clock can't be used as axi_lite control bus for DPU now.)***<br><br />

7. Add the Processor System Reset blocks:<br />
   a) Right click Diagram view and select ***Add IP***.<br />
   b) Search for and add a Processor System Reset from the IP Search dialog<br />
   c) Add 2 more Processor System Reset blocks, using the previous step; or select the proc_sys_reset_0 block and Copy (Ctrl-C) and Paste (Ctrl-V) it four times in the block diagram<br />
   d) Rename them as ```proc_sys_reset_100m```, ```proc_sys_reset_300m```, ```proc_sys_reset_600m```<br />
  
8. Connect Clocks and Resets: <br />
   a) Click ***Run Connection Automation***, which will open a dialog that will help connect the proc_sys_reset blocks to the clocking wizard clock outputs.<br />
   b) Enable All Automation on the left side of the Run Connection Automation dialog box.<br />
   c) Select clk_in1 on clk_wiz_0, and set the Clock Source to ***/zynq_ultra_ps_e_0/pl_clk0***.<br />
   d) For each proc_sys_reset instance, select the slowest_sync_clk, and set the Clock Source as follows:<br />
      - proc_sys_reset_100m with /clk_wiz_0/clk_100m<br />
      - proc_sys_reset_300m with /clk_wiz_0/clk_300m<br />
      - proc_sys_reset_600m with /clk_wiz_0/clk_600m<br />

   e) On each proc_sys_reset instance, select the ***ext_reset_in***, set ***Board Part Interface*** to ***Custom*** and set the ***Select Manual Source*** to ***/zynq_ultra_ps_e_0/pl_resetn0***.<br />
   f) Make sure all checkboxes are enabled, and click ***OK*** to close the dialog and create the connections.<br />
   g) Connect all the ***dcm_locked*** signals on each proc_sys_reset instance to the locked signal on ***clk_wiz_0***.<br />
   Then the connection should like below:<br />
   ![clk_rst_connection.png](/pic_for_readme/reset_system.png)<br /><br />
***Now we have added clock and reset IPs and configure and connect them. Some would be used in creating the hardware platform and some would be called in Vitis high level design***<br /><br />

9. Add Kernel Interrupt Support<br />
You can provide kernel interrupt support by adding an AXI interrupt controller to the base hardware design.<br />
   a) In the block diagram, double-click the Zynq UltraScale+ MPSoC block.<br />
   b) Select ***PS-PL Configuration > PS-PL interfaces > Master interface***.<br />
   c) Select the ***AXI HPM0 LPD*** check box, keep the ***AXI HPM0 LPD Data width*** settings as default ***32***.<br />
   d) Click ***OK*** to finsih the configuration.<br />
   e) Connect ***maxihpm0_lpd_aclk*** to ***/clk_wiz_0/clk_100m***.<br />
   f) Right click Diagram view and select ***Add IP***, search and add ***AXI Interrupt Controller*** IP.<br />
   g) Double click the AXI Interrupt Controller block, set ***Interrupts type*** to ```Level Interrupt```, set ***Level type*** to ```Active High```, set ***Interrupt Output Connection*** to ```Bus```. Click ***OK*** to save the change.<br />
   ![intc_settings.png](/pic_for_readme/intc_settings.png)<br /><br />
   h) Click the AXI Interrupt Controller block and go to ***Block Properties -> Properties***, configure or make sure the parameters are set as following:
***C_ASYNC_INTR***: ```0xFFFFFFFF```. <br />
   ![async_intr.png](/pic_for_readme/async_intr.png)<br /><br />
   ***When interrupts generated from kernels are clocked by different clock domains, this option is useful to capture the interrupt signals properly. For the platform that has only one clock domain, this step can be skipped.***

   i) Click ***Run Connection Automation***<br />   
   j) Leave the default values for Master interface and Bridge IP.<br />
      - Master interface default is /zynq_ultra_ps_e_0/M_AXI_HPM0_LPD.<br />
      - Bridge IP default is New AXI interconnect.<br />
   k) Expand output interface Interrupt of axi_intc_0 to show the port irq, connect this irq portto zynq_ultra_ps_e_0.pl_ps_irq0<br />
   l) Setup ***PFM_IRQ** property by typing following command in Vivado console:<br />
   ```set_property PFM.IRQ {intr {id 0 range 32}} [get_bd_cells /axi_intc_0]```<br />
***The IPI design connection would like below:***<br />
![ipi_fully_connection.png](/pic_for_readme/ipi_designn.png)<br /><br />
***Note: Now we have finished the IPI design , let's set some platform parameters and generate the XSA***<br /><br /><br />

## Configuring Platform Interface Properties<br /><br />
1. Run the following Tcl command to enable the extensive platform feature<br />
```set_property platform.extensible TRUE [get_project]```
2. Select ***Platform-system->zynq_ultra_ps_e_0->S_AXI_HP0_FPD***, in ***Platform interface Properties*** tab enable the ***Enabled*** option like below:<br />
![enable_s_axi_hp0_fpd.png](/pic_for_readme/enable_s_axi_hp0_fpd.png)<br /><br />
3. Select ***Options*** tab, set ***memport*** to ```S_AXI_HP``` and set ***sptag*** to ```HP0``` like below:
![set_s_axi_hp0_fpd_options.png](/pic_for_readme/set_s_axi_hp0_fpd_options.png)<br /><br />
4. Do the same operations for ***S_AXI_HP1_FPD, S_AXI_HP2_FPD, S_AXI_HP3_FPD, S_AXI_HPC0_FPD, S_AXI_HPC1_FPD*** and set ***sptag*** to ```HP1```, ```HP2```, ```HP3```, ```HPC0```, ```HPC1```. And be noticed that for HPC0/HPC1 ports the ***memport*** is set to ```S_AXI_HPC``` in default, but actually we would use these ports without data coherency function enabled to get a high performance. So please modify it into ```S_AXI_HP``` manually.<br />
![set_s_axi_hpc0_fpd_options.png](/pic_for_readme/set_s_axi_hpc0_fpd_options.png)<br /><br />
5. Enable the M01_AXI ~ M08_AXI ports of ps8_0_axi_periph IP(The axi_interconnect between M_AXI_HPM0_LPD and axi_intc_0), and set these ports with the same ***sptag*** name to ```HPM0_LPD``` and ***memport*** type to ```M_AXI_GP```<br />
6. Enable the ***M_AXI_HPM0_FPD*** and ***M_AXI_HPM1_FPD*** ports, set ***sptag*** name to ```HPM0_FPD```, ```HPM1_FPD``` and ***memport*** to ```M_AXI_GP```.<br />
***Now we enable AXI master/slave interfaces that can be used for Vitis tools on the platform***<br /><br />
7. Enable ***clk_300m***, ***clk_600m***, ***clk_100m*** of clk_wiz_0, set ***id*** of ***clk_300m*** to ```0```, set ***id*** of ***clk_600m*** to ```1```, set ***id*** of ***clk_100m*** to ```2```, enable ***is default*** for ***clk_300m***.<br />

8. Create a ```xsa_gen``` folder inside your Vivado project.<br />
9. Create a file named ```xsa.tcl``` inside the ***xsa_gen*** folder.<br />
10. Copy the following commands into the xsa.tcl file and save the file.<br />
```
# Set the platform design intent properties
set_property platform.design_intent.embedded true [current_project]
set_property platform.design_intent.server_managed false [current_project]
set_property platform.design_intent.external_host false [current_project]
set_property platform.design_intent.datacenter false [current_project]

get_property platform.design_intent.embedded [current_project]
get_property platform.design_intent.server_managed [current_project]
get_property platform.design_intent.external_host [current_project]
get_property platform.design_intent.datacenter [current_project]

# Set the platform default output type property
set_property platform.default_output_type "sd_card" [current_project]

get_property platform.default_output_type [current_project]
```
11. In your Vivado project, use the ***Tcl console*** to ***navigate to the xsa_gen folder***, and run ```source ./xsa.tcl``` command.
![run_xsa_tcl.png](/pic_for_readme/xsa.png)<br /><br />
12. Right-click and select ***Validate Design*** on ***IP integrator diagram*** (For AXI intc critical warning, you can ignore it safely)<br />
13. Create the HDL wrapper:<br />
    a. Right-click ***system.bd*** in the Block Design, Sources view and select Create HDL Wrapper.<br />
    b. Select Let Vivado manage wrapper and ***auto-update***.<br />
    c. Click ***OK***.<br />

14. Right-click ***system.bd*** in the Block Design, Sources view and select ***Generate Output Products***.<br />
15. Type the tcl command in tcl console like:<br />
```write_hw_platform -unified -force -file <your_vivado_project_dir>/xsa_gen/zu6eg_custom_platform.xsa```<br />
If you use ***export Hardware*** function in Vivado GUI it would add ***-fixed*** option which would generate a XSA for traditional embedded platform which can't add DPU acceleration kernel here.
16. Check the ***<your_vivado_project_dir>/xsa_gen*** folder, you should find the ***zu6eg_custom_platform.xsa*** generated there.<br />

***Now we finish the Hardware platform creation flow, then we should go to the Software platform creation***<br /><br />

## Create the PetaLinux Software Component<br /><br />

A Vitis platform requires software components. For Linux, the PetaLinux tools are invoked outside of the Vitis tools by the developer to create the necessary Linux image,Executable and Linkable Format (ELF) files, and sysroot with XRT support. Yocto or third-party Linux development tools can also be used as long as they produce the same Linux output products as PetaLinux. <br />
1. source <petaLinux_tool_install_dir>/settings.sh<br />
2. Create a PetaLinux project named ***zu6eg_custom_plnx*** and configure the hw with the XSA file we created before:<br /> 
```petalinux-create -t project --template zynqMP -n zu6eg_custom_plnx```<br />
***Note:If you have bsp, you can import your bsp directly***<br />
```petalinux-create -t project -s <your-bsp-path> -n zu6eg_custom_plnx```<br />
After creating the petalinux project, you can import XSA file to the petalinux project<br />
```cd zu6eg_custom_plnx```<br />
```petalinux-config --get-hw-description=<your_vivado_design_path>/xsa_gen/```<br />
3. A petalinux-config menu would be launched, select ***DTG Settings->MACHINE_NAME***, modify it to ```zcu104-revc```.<br />
***Note: If you are using a Xilinx development board it is recomended to modify the machine name so that the board configurations would be involved in the DTS auto-generation. Otherwise you would need to configure the associated settings(e.g. the PHY information DTS node) by yourself manually.***<br />
4. Add user packages by appending the CONFIG_x lines below to the <your_petalinux_project_path>/project-spec/meta-user/conf/user-rootfsconfig file.<br />
Packages for base XRT support:<br />
```
CONFIG_xrt
CONFIG_xrt-dev
CONFIG_zocl
CONFIG_dnf
CONFIG_e2fsprogs-resize2fs
CONFIG_parted
CONFIG_mesa-megadriver
CONFIG_opencl-clhpp-dev
CONFIG_opencl-headers-dev
CONFIG_opencv-dev
CONFIG_packagegroup-petalinux-opencv
CONFIG_packagegroup-petalinux-opencv-dev
```
Packages for DPU support:<br />
```
CONFIG_packagegroup-petalinux-vitisai
```
Packages for building Vitis AI applications:<br /> 
```
CONFIG_googletest
CONFIG_googletest-staticdev
CONFIG_json-c-dev
CONFIG_protobuf-dev
CONFIG_protobuf-c
CONFIG_libeigen-dev
CONFIG_libcanberra-gtk3
```
Packages for native compiling on target board:<br /> 
```
CONFIG_packagegroup-petalinux-self-hosted
CONFIG_cmake 
```
Packages mentioned at DPU integration lab for Vivado flow:<br /> 
```
CONFIG_packagegroup-petalinux-x11
CONFIG_packagegroup-petalinux-v4lutils
CONFIG_packagegroup-petalinux-matchbox
```
5. Run ```petalinux-config -c rootfs``` and  then select ***user packages***.On this page, you sould select the hightlight libraries as below , and then save and exit.
![petalinux_rootfs.png](/pic_for_readme/user_package.png)<br /><br />

6. Enable OpenSSH and disable dropbear<br /> 
Dropbear is the default SSH tool in Vitis Base Embedded Platform. If OpenSSH is used to replace Dropbear, it could achieve 4x times faster data transmission speed (tested on 1Gbps Ethernet environment). Since Vitis-AI applications may use remote display feature to show machine learning results, using OpenSSH can improve the display experience.<br /> 
    a) Run ```petalinux-config -c rootfs```.<br /> 
    b) Go to ***Image Features***.<br /> 
    c) Disable ***ssh-server-dropbear*** and enable ***ssh-server-openssh***.<br /> 
    ![ssh_settings.png](/pic_for_readme/openssh.png)<br /><br />
    d) Go to ***Filesystem Packages-> misc->packagegroup-core-ssh-dropbear*** and disable ***packagegroup-core-ssh-dropbear***.<br />
    e) Go to ***Filesystem Packages  -> console  -> network -> openssh*** and enable ***openssh***, ***openssh-sftp-server***, ***openssh-sshd***, ***openssh-scp***.<br />
7. In rootfs config go to ***Image Features*** and enable ***package-management*** and ***debug_tweaks*** option, store the change and exit.<br />
8. CPU IDLE would cause CPU IDLE when JTAG is connected. So it is recommended to disable the selection.<br /> 
    a) Type ```petalinux-config -c kernel```<br /> 
    b) Ensure the following are ***TURNED OFF*** by entering 'n' in the [ ] menu selection for:<br />
       - ***CPU Power Mangement > CPU Idle > CPU idle PM support***<br />
       - ***CPU Power Management > CPU Frequency scaling > CPU Frequency scaling***<br /><br />

9. Update the Device tree to include the zocl driver by appending the text below to the ***project-spec/meta-user/recipes-bsp/device-tree/files/system-user.dtsi*** file. 
```
&amba {
	zyxclmm_drm {
		compatible = "xlnx,zocl";
		status = "okay";
		interrupt-parent = <&axi_intc_0>;
		interrupts = <0  4>, <1  4>, <2  4>, <3  4>,
			     <4  4>, <5  4>, <6  4>, <7  4>,
			     <8  4>, <9  4>, <10 4>, <11 4>,
			     <12 4>, <13 4>, <14 4>, <15 4>,
			     <16 4>, <17 4>, <18 4>, <19 4>,
			     <20 4>, <21 4>, <22 4>, <23 4>,
			     <24 4>, <25 4>, <26 4>, <27 4>,
			     <28 4>, <29 4>, <30 4>, <31 4>;
	};
};

&axi_intc_0 {
      xlnx,kind-of-intr = <0x0>;
      xlnx,num-intr-inputs = <0x20>;
      interrupt-parent = <&gic>;
      interrupts = <0 89 4>;
};

&sdhci1 {
      no-1-8-v;
      disable-wp;
};

```

10. Modify the u-boot settings:<br />
We use initramfs method to make the rootfs files, and then we need to modify the u-boot so that it can load larger image.
Open ***project-spec/meta-user/recipes-bsp/u-boot/files/platform-top.h*** and add:<br />
```
#define CONFIG_SYS_BOOTM_LEN 0x80000000
#undef CONFIG_SYS_BOOTMAPSZ
```
11. After setting all the petalinux configuration, type ```petalinux-build``` to build the project.<br />
12. Create a sysroot self-installer for the target Linux system:<br />
```
cd images/linux
petalinux-build --sdk
```
***Note: We would store all the necessary files for Vitis platform creation flow. Here we name it ```zu6eg_dpu_pkg ```. Then we create a pfm folder inside.***<br />
13. Type ```./sdk.sh``` to install PetaLinux SDK, provide a full pathname to the output directory ***<full_pathname_to_zu6eg_dpu_pkg>/pfm*** (here in this example I use ***/home/zu6eg_dpu_pkg/pfm***) and confirm.<br />
14. We would install Vitis AI library and DNNDK into this rootfs in the future.<br />
15. After the PetaLinux build succeeds, the generated Linux software components are in the ***<your_petalinux_dir>/images/linux directory***. For our example, the ***images/linux*** directory contains the generated image and ELF files listed below. Copy these files to the ***<full_pathname_to_zu6eg_dpu_pkg>/pfm/boot*** directory in preparation for running the Vitis platform creation flow:<br />
```
    - image.ub
    - fsbl.elf (Rename the petalinux generated zynq_fsbl.elf to fsbl.elf)
    - pmufw.elf
    - bl31.elf
    - u-boot.elf
    - boot.scr
```
16. Add a BIF file (linux.bif) to the ***<full_pathname_to_zu6eg_dpu_pkg>/pfm/boot*** directory with the contents shown below. The file names should match the contents of the boot directory. The Vitis tool expands these pathnames relative to the sw directory of the platform at v++ link time or when generating an SD card. However, if the bootgen command is used directly to create a BOOT.BIN file from a BIF file, full pathnames in the BIF are necessary. Bootgen does not expand the names between the <> symbols.<br />
***Note: There is  no bitstream path now,you can ignore here .***<br />
```
/* linux */
 the_ROM_image:
 {
 	[fsbl_config] a53_x64
 	[bootloader] <fsbl_path>/fsbl.elf
 	[pmufw_image] <pmufw_path>/pmufw.elf
 	[destination_device=pl] <bitstream>
 	[destination_cpu=a53-0, exception_level=el-3, trustzone] <bl31_path>/bl31.elf
 	[destination_cpu=a53-0, exception_level=el-2] <u-boot_path>/u-boot.elf
 }
```

***Note: Now we prepare the HW platform and SW platform, next we would create a Vitis Platform.***<br />

## Create the Vitis Platform<br /><br />

1. Source Vitis and XRT settings<br />
```
source <Vitis_Install_Directory>/settings64.sh
source /opt/xilinx/xrt/setup.sh
```
2. Go to the ***zu6eg_dpu_pkg*** folder you created: ```cd <full_pathname_to_zu6eg_dpu_pkg>``` and type ./sdk.sh -d <Install Target Dir> to install PetaLinux SDK. use the -d option to provide a full pathname to the output directory zu6eg_dpu_pkg/pfm (This is an example ) and confirm.<br />
3. Launch Vitis by typing ```vitis``` in the console.<br />
4. Select ***zu6eg_dpu_pkg*** folder as workspace directory.<br />
![vitis_launch.png](/pic_for_readme/workspace_6eg.png)<br /><br />
5. In the Vitis IDE, select ***File > New > Platform Project*** to create a platform project.<br />
6. In the Create New Platform Project dialog box, do the following:<br />
   a) Enter the project name. For this example, type ```zu6eg_custom```.<br />
   b) Leave the checkbox for the default location selected.<br />
   c) Click ***Next***.<br />
7. In the Platform Project dialog box, do the following:<br />
   a) Select ***Create from hardware specification (XSA)***.<br />
   b) Click ***Next***.<br />
8. In the Platform Project Specification dialog box, do the following:<br />
   a) Browse to the XSA file generated by the Vivado. In this case, it is located in ```zu6eg_custom_platform/xsa_gen/zu6eg_custom_platform.xsa```.<br />
   b) Set the operating system to ***linux***.<br />
   c) Set the processor to ***psu_cortexa53***.<br />
   d) Leave the checkmark selected to generate boot components.<br />
   e) Click ***Finish***.<br />
![new_platform_project.png](/pic_for_readme/new_platform_project.png)<br /><br />	
9. In the Platform Settings view, observe the following:<br />
   - The name of the Platform Settings view matches the platform project name of ***zu6eg_custom***<br />
   - A psu_cortexa53 device icon is shown, containing a Linux on psu_cortexa53 domain.<br />
   - A psu_cortexa53 device icon is shown, containing a zynqmp_fsbl BSP.<br />
   - A psu_pmu_0 device icon is shown, containing a zynqmp_pmufw BSP.<br />
10. Click the linux on psu_cortexa53 domain, browse to the locations and select the directory or file needed to complete the dialog box for the following:
```
Linux Build Output:
    Browse to zu6eg_dpu_pkg/pfm/boot and click OK.
    
Bif file:
    Browse to zu6eg_dpu_pkg/pfm/boot/linux.bif file and click OK.

Image:
    Browse to zu6eg_dpu_pkg/pfm/boot and click OK.
```
![vitis_linux_config.png](/pic_for_readme/workspace_setting_6eg.png)<br /><br />
11. Click ***zu6eg_custom*** project in the Vitis Explorer view, click the ***Build*** button to generate the platform.
![build_vitis_platform.png](/pic_for_readme/zu6eg_build_platform.png)<br /><br />
***Note: The generated platform is placed in the export directory. BSP and source files are also provided for re-building the FSBL and PMU if desired and are associated with the platform. The platform is ready to be used for application development.***<br />

## Prepare for the DPU Kernel<br /><br />

1. Download Vitis AI by git command ```git clone https://github.com/Xilinx/Vitis-AI.git```.<br />
2. Navigate to the repository:```cd Vitis-AI```, set the tag to proper tag(here we use ***v1.3.0***) by typing: ```git checkout v1.3.0```.<br />
3. If you don't want to destroy the TRD reference design. Copy ***DPU-TRD*** folder into another directory. For example I would copy that into my ***zu6eg_dpu_pkg*** folder: ```cp -r DPU-TRD /home/zu6eg_dpu_pkg/```<br />
4. Source Vitis tools setting sh file: ```source <vitis install path>/Vitis/2020.2/settings64.sh```.<br />
5. Source XRT sh file:```source /opt/xilinx/xrt/setup.sh```.<br />
6. Export SDX_PLATFORM with the directory of the custom platform xpfm file which you created before. Here in my project it would be: ```export SDX_PLATFORM=/home/zu6eg_dpu_pkg/zu6eg_custom/export/zu6eg_custom/zu6eg_custom.xpfm```. Remember now this custom platform name is ***zu6eg_custom***.<br />
7. Navigate to the copy of the ***DPU-TRD*** folder, then go to the ***./prj/Vitis*** folder.<br />
There are 2 files can be used to modify the DPU settings: The ***config_file/prj_config*** file is for DPU connection in Vitis project and the dpu_conf.vh is for other DPU configurations. Here we would modify the prj_config so that 2 DPU cores are enabled. And then we modify dpu_conf.vh as [DPU-TRD readme](https://github.com/Xilinx/Vitis-AI/blob/v1.3/dsa/DPU-TRD/README.md) suggested.<br />
8. Modify the ***config_file/prj_config*** like below:<br />
```

[clock]

id=0:DPUCZDX8G_1.aclk
id=1:DPUCZDX8G_1.ap_clk_2
id=0:DPUCZDX8G_2.aclk
id=1:DPUCZDX8G_2.ap_clk_2
id=0:sfm_xrt_top_1.aclk

[connectivity]

sp=DPUCZDX8G_1.M_AXI_GP0:HPC0
sp=DPUCZDX8G_1.M_AXI_HP0:HP0
sp=DPUCZDX8G_1.M_AXI_HP2:HP1
sp=DPUCZDX8G_2.M_AXI_GP0:HPC0
sp=DPUCZDX8G_2.M_AXI_HP0:HP2
sp=DPUCZDX8G_2.M_AXI_HP2:HP3
sp=sfm_xrt_top_1.M_AXI:HPC1

[advanced]
misc=:solution_name=link
#param=compiler.addOutputTypes=sd_card

#param=compiler.skipTimingCheckAndFrequencyScaling=1

[vivado]
#prop=run.impl_1.strategy=Performance_ExploreWithRemap
prop=run.impl_1.strategy=Performance_Explore
#param=place.runPartPlacer=0

```
9. Keep original dpu_conf.vh setting, when your part is zu6eg<br />
```
`define URAM_DISABLE 
`define RAM_USAGE_LOW
```

10. Generate the XO file by typing: <br />
***Make sure the path is <zu6eg_dpu_pkg directory>/DPU-TRD/prj/Vitis***.<br />
```make binary_container_1/dpu.xo DEVICE=zu6eg_custom```<br />
If you enable softmax function in  DPU ip, you should also generate softmax.xo:<br />
```make binary_container_1/softmax.xo DEVICE=zu6eg_custom```<br />
11. Verify if the XO file is generated here: <br />
***<zu6eg_dpu_pkg directory>/DPU-TRD/prj/Vitis/binary_container_1/dpu.xo***.<br />
If you have generated softmax.xo,  and  you can verify it as below path.<br />
***<zu6eg_dpu_pkg directory>/DPU-TRD/prj/Vitis/binary_container_1/softmax.xo***.<br />

## Create and Build a Vitis application
1. Open Vitis workspace you were using before.<br />
2. Select ***File -> New -> Application Project***.<br />
3. Click ***next***<br />
4. Select ***zu6eg_custom*** as platform, click ***next***.<br />
5. Name the project ```hello_dpu```, click ***next***.<br />
5. Set Domain to ***linux on psu_cortexa53***, set ***Sys_root path*** to ```<full_pathname_to_zu6eg_dpu_pkg>/pfm/sysroots/aarch64-xilinx-linux```(as you created by running ***sdk.sh***), keep the ***Kernel Image*** setting in default and click ***next***.<br />
6. Select ***System accelearation templates -> Empty application*** and click ***finish*** to generate the application.<br />
7. Right click on the ***src*** folder under your ***hello_dpu*** application  in the Expplorer window, and select "Import Sources"
8. Choose from directory ***<zu6eg_dpu_pkg directory>/DPU-TRD/prj/Vitis/binary_container_1/*** as the target location, and import the ***dpu.xo*** file that we just created.<br />
If you have softmax, you should also import the ***softmax.xo*** file that we just created.<br />
9. Import sources again, and add the cpp, header and prj_config files from ***ref_files/src*** folder provided by this Git repository.<br />
10. In the Explorer window double click the hello_dpu.prj file to open it, change the ***Active Build configuration*** from ***Emulation-SW*** to ***Hardware***.<br />
11. Under Hardware Functions in hello_dpu_system_hw_link item, click the lightning bolt logo to ***Add Hardware Function***.<br />
![add_dpu_softmax_202.png](/pic_for_readme/add_dpu_softmax_202.png)<br /><br />
12. Select the "DPUCZDX8G" included as part of the dpu.xo file that we included earlier.<br />
13. Click on binary_container_1 to change the name to dpu.<br />
14. Click on ***DPUCZDX8G*** and change the ***Compute Units*** from ```1``` to ```2``` because we have 2 dpu cores involved.<br />
15. Right click on "dpu", select ***Edit V++ Options***, add ```--config <zu6eg_dpu_pkg path>/hello_dpu_kernels/src/prj_config -s``` as ***V++ Options***, then click ***OK***.<br />
![vitis_v++_config_202.png](/pic_for_readme/vitis_v++_config_202.png)<br /><br />
16. Go back to the ***Explorer*** window, right click on the ***hello_dpu*** project folder select ***C/C++ Building Settings**.<br />
17. In ***Propery for hello_dpu*** dialog box, select ***C/C++ Build->Settings->Tool Settings->GCC Host Linker->Libraries***
, click the green "+" to add the following libraries:
```
opencv_core
opencv_imgcodecs
opencv_highgui
opencv_imgproc
opencv_videoio
n2cube
hineon
```
18. In the same page, Check the ***Library search path*** to makesure the ```${SYSROOT}/usr/lib/``` is added, click ***Apply***<br />
![vitis_lib_settings.png](/pic_for_readme/vitis_lib_settings.png)<br /><br />
19. Then go to ***C/C++ Build->Settings->Tool Settings->GCC Host Compiler->Includes***, remove the HLS include directory and add ```${SYSROOT}/usr/include/``` like below, then click ***Apply and Close*** to save the changes.<br />
![vitis_include_settings.png](/pic_for_readme/vitis_include_settings.png)<br /><br />
***These steps are used to make sure your application can call libs in rootfs directly on Vitis appilcation build***
20. The Vitis AI library and DNNDK are not included in PetaLinux SDK rootfs, now let's install them into the rootfs directory:<br />
***Note:*** We should follow the section ***Setting Up the Host For Edge*** of [Vitis AI library readme file](https://github.com/Xilinx/Vitis-AI/blob/v1.3/demo/Vitis-AI-Library/README.md) to install the Vitis AI library and section ***Setup cross-compiler for Vitis AI DNNDK and make samples*** of [DNNDK readme file](https://github.com/Xilinx/Vitis-AI/blob/v1.3/demo/DNNDK/README.md) to install the DNNDK. If you feel difficult to following the official guide there you can refer to the following ones. ***Please just skip these steps if you already install the libs referring to the readme files***:<br />
    a) Download the [vitis_ai_2020.2-r1.3.0.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=vitis_ai_2020.2-r1.3.0.tar.gz) to a particular directory(here we take ***~/Downloads*** as example) and install it to the roofs folder:<br />
    ```
    cd ~/Downloads # Or some place else you download the vitis_ai_2020.2-r1.3.0.tar.gz file
    tar -xzvf vitis_ai_2020.2-r1.3.0.tar.gz -C <full_pathname_to_zu6eg_dpu_pkg>/pfm/sysroots/aarch64-xilinx-linux
    ```
    b) Download DNNDK runtime package [vitis-ai_v1.3_dnndk.tar.gz ](https://www.xilinx.com/bin/public/openDownload?filename=vitis-ai_v1.3_dnndk.tar.gz) to ***~/Downloads*** and install it into rootfs
    ```
    cd ~/Downloads # Or some place else you download the file
    tar -xzvf vitis-ai_v1.3_dnndk.tar.gz
    cd vitis-ai_v1.3_dnndk
    ./install.sh <full_pathname_to_zu6eg_dpu_pkg>/pfm/sysroots/aarch64-xilinx-linux
    ```
***Now we install both the VAI lib and DNNDK packages into the rootfs set as Vitis sysroot, then we can build application on Vitis.***<br />

21. Right click the ***hello_dpu*** project folder and select ***Build Project***<br />
22. Generate Boot Image<br />
![zu6eg_boot_gen.png](/pic_for_readme/zu6eg_boot_gen.png)<br />
When you load bif, the bitstram path is loaded by tool<br />
![zu6eg_boot_gen.png](/pic_for_readme/zu6eg_boot_gen_bif.png)<br />
## Prepare the Network Deployment File<br />

1-1. Find HWH file from your Vitis application folder***hello_dpu_system_hw_link/Hardware/dpu.build/link/vivado/vpl/prj/prj.gen/sources_1/bd/system/hw_handoff/system.hwh***<br />
Or go to your Vitis application folder use command ```find -name *.hwh``` to search for the file.<br />
1-2. Use command ```find -name arch.json``` to find arch.json and compare the content to get the correct string as below
   ```
DPUCAHX8H_ISA2=>0x20200000000002a,
DPUCAHX8H_ISA2_ELP2=>0x20200000000002e,
DPUCAHX8L_ISA0=>0x30000000000001d,
DPUCVDX8G_ISA0_B16384C64B1=>0x600000076080812,
DPUCVDX8G_ISA0_B8192C32B1=>0x600000076080811,
DPUCVDX8G_ISA0_B8192C32B1_ELP4=>0x600000076040411,
DPUCVDX8G_ISA0_B8192C32B3=>0x600000076080831,
DPUCVDX8G_ISA0_B8192C32B3_DW=>0x6000000f6088831,
DPUCVDX8G_ISA0_B8192C32B3_I4W8B2=>0x600000276080831,
DPUCVDX8G_ISA0_B8192C32B3_I8W4B2=>0x600000376080831,
DPUCVDX8G_ISA0_B8192C32B3_I8W8B2=>0x600000176080831,
DPUCVDX8H_ISA0=>0x5000000000007ee,
DPUCZDI4G_ISA0_B4096_DEMO_SSD=>0x400002003220206,
DPUCZDI4G_ISA0_B8192D8_DEMO_SSD=>0x400002003220207,
DPUCZDX8G_ISA0_B1024_MAX=>0x1000020f7014402,
DPUCZDX8G_ISA0_B1024_MIN=>0x100002022010102,
DPUCZDX8G_ISA0_B1152_MAX=>0x1000020f7012203,
DPUCZDX8G_ISA0_B1152_MIN=>0x100002022010103,
DPUCZDX8G_ISA0_B1600_MAX=>0x1000020f7014404,
DPUCZDX8G_ISA0_B1600_MIN=>0x100002022010104,
DPUCZDX8G_ISA0_B2304_MAX=>0x1000020f7014405,
DPUCZDX8G_ISA0_B2304_MAX_BG2=>0x1000020f6014405,
DPUCZDX8G_ISA0_B2304_MIN=>0x100002022010105,
DPUCZDX8G_ISA0_B3136_MAX=>0x1000020f7014406,
DPUCZDX8G_ISA0_B3136_MAX_BG2=>0x1000020f6014406,
DPUCZDX8G_ISA0_B3136_MIN=>0x100002022010106,
DPUCZDX8G_ISA0_B4096_MAX=>0x1000020f7014407,
DPUCZDX8G_ISA0_B4096_MAX_BG2=>0x1000020f6014407,
DPUCZDX8G_ISA0_B4096_MAX_EM=>0x1000030f7014407,
DPUCZDX8G_ISA0_B4096_MIN=>0x100002022010107,
DPUCZDX8G_ISA0_B512_MAX=>0x1000020f7012200,
DPUCZDX8G_ISA0_B512_MIN=>0x100002022010100,
DPUCZDX8G_ISA0_B800_MAX=>0x1000020f7012201,
DPUCZDX8G_ISA0_B800_MIN=>0x100002022010101}
   ```
In this eample, the correct string is DPUCZDX8G_ISA0_B4096_MAX_BG2. And the You should update Tool-Example/arch.json to replace your own string.
2. Copy the ***ref_files/Tool-Example*** folder provided by this Github repository to your Vitis AI download directory.<br />
3. Copy this HWH file into ***<Vitis-AI-download_directory>/Tool-Example*** folder.<br />
4. Go to ***<Vitis-AI-download_directory>*** folder and launch the docker.(If you don't have docker, you can refer to the [docker_installation.md](https://github.com/hill19850213/vitis_ai_custom_platform/blob/master/docker_installation.md)<br />
```
./docker_run.sh xilinx/vitis-ai-gpu:latest
```
5. Use following command to activate TensorFlow tool conda environment:<br />
```
conda activate vitis-ai-tensorflow
```
6. Go to ***/workspace/Tool-Example*** folder and run ```dlet -f ./system.hwh``` or ```dlet -f ./<your-folder-path>/system.hwh```.<br />
You should get the running log as below:
```
(vitis-ai-tensorflow) hill213@hill213-pc:/workspace/Tool-Example$ dlet -f ./system.hwh 
[DLet]Generate DPU DCF file dpu-11-02-2020-15-15.dcf successfully.
```
The DCF file name should be associated with the time and date you generating this file.<br />
7. Open the ***arch.json*** file and make sure the ***"dcf"*** parameter is set with the name you got on the previous step:<br />
```"dcf"      : "./dpu-11-02-2020-15-15.dcf",```<br />
8. Run command```sh download_model.sh``` to download the Xilinx Model Zoo files for resnet-50.<br />
9. Run command```sh custom_platform_compile.sh```, go to ***tf_resnetv1_50_imagenet_224_224_6.97G/vai_c_output_ZCU102/dpu_resnet50_0.elf*** to get the ***dpu_resnet50_0.elf*** file.<br />
10. Copy that file to the ***src*** folder of Vitis application ***hello_dpu***<br />
11. Right click on the ***hello_dpu*** project folder in Vitis select ***C/C++ Building Settings**.<br />
12. In ***Propery for Hello_DPU*** dialog box, select ***C/C++ Build->Settings->Tool Settings->GCC Host Linker->Miscellaneous->Other objects***, add a new object: ```"${workspace_loc:/${ProjName}/src/dpu_resnet50_0.elf}"```, click ***Apply and Close***.<br />
13. Right click the ***hello_dpu*** project folder and select ***Build Project***<br />
![zu6eg_dpu_elf.png](/pic_for_readme/zu6eg_dpu_elf.png)<br /><br />
***Now you should get an updated hello_dpu with a size of about 20MB(the ConvNet model is involved).***<br />

## Run Application on Board<br />
1. Copy all the files from ***sd_card folder*** inside your Vitis application like ***<hello_dpu_application_directory>/Hardware/sd_card/*** to SD card, copy all the files under ***ref_files/boot_additional_files/*** provided by this Github repository to SD card, set zu6eg board to SD boot mode and boot up the board, connect the board with serial port.<br />
2. Connect SSH:<br />
   a) Run ```ifconfig``` to get the IP address, here we take ```192.168.17.2``` as example.<br />
   b) Using SSH terminal to connect zu6eg board with SSH: ```ssh -x root@192.168.17.2```, or use putty in Windows.<br />
3. Go to the /home/root folder and create a new folder named "Vitis-AI/vitis_ai_library":
```
cd /home/root
mkdir -p Vitis-AI/vitis_ai_library
```
4. Since this is a custom design the Vitis AI library, DNNDK and test images are not installed. We need to install them on board.<br />
I would suggest you to refer to section "Setting Up the Target" of [Vitis AI library readme file](https://github.com/Xilinx/Vitis-AI/blob/v1.3/demo/Vitis-AI-Library/README.md) to install the Vitis AI library and refer to section "Setup Evaluation Board and run Vitis AI DNNDK samples" of [DNNDK example readme file](https://github.com/Xilinx/Vitis-AI/blob/v1.3/demo/DNNDK/README.md) to install DNNDK and test images. If you feel difficult to do that please follow the steps below:<br />
   a) Download the Vitis AI Runtime 1.3.0 package [Vitis AI Runtime 1.3.0](https://www.xilinx.com/bin/public/openDownload?filename=vitis-ai-runtime-1.3.0.tar.gz)<br />
   b) Copy the vitis-ai-runtime-1.3.0.tar.gz from host to board with the following command running on host:<br />
   ```
   cd <path_to_vitis-ai-runtime-1.3.0.tar.gz>
   scp vitis-ai-runtime-1.3.01.tar.gz 192.168.17.2:/home/root
   ```
   c) Untar the packet and install them one by one on target board:<br />
   ```
   cd /home/root
   tar -zxvf vitis-ai-runtime-1.3.0.tar.gz
   cd ./vitis-ai-runtime-1.3.0/aarch64/centos/
   bash setup.sh  
   ```
    d) If you want to update the Vitis AI Model which you want to use
   Download [ AI Model](https://github.com/Xilinx/Vitis-AI/tree/master/models/AI-Model-Zoo/model-list)，

   e) Download the [vitis_ai_library_r1.3.x_images.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=vitis_ai_library_r1.3.0_images.tar.gz) and the [vitis_ai_library_r1.3.x_video.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=vitis_ai_library_r1.3.0_video.tar.gz). Copy them from host to the target using scp with the following command.<br />
   ```
   scp vitis_ai_library_r1.3.x_images.tar.gz root@192.168.17.2:/home/root
   scp vitis_ai_library_r1.3.x_video.tar.gz root@192.168.17.2:/home/root

   cd ~
   tar -xzvf vitis_ai_library_r1.3.x_images.tar.gz -C Vitis-AI/vitis_ai_library
   tar -xzvf vitis_ai_library_r1.3.x_video.tar.gz -C Vitis-AI/vitis_ai_library
   ```    
   f) Download the package [vitis-ai_v1.3_dnndk.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=vitis-ai_v1.3_dnndk.tar.gz) and package [vitis-ai_v1.3_dnndk_sample_img.tar.gz](https://www.xilinx.com/bin/public/openDownload?filename=vitis-ai_v1.3_dnndk_sample_img.tar.gz), copy them to board:<br />
   ```
   scp vitis-ai_v1.3_dnndk.tar.gz root@192.168.17.2:/home/root
   scp vitis-ai_v1.3_dnndk_sample_img.tar.gz root@192.168.17.2:/home/root
   ```
   g) Install DNNDK package like below:<br />
   ```
   cd /home/root
   cp vitis-ai_v1.3_dnndk.tar.gz ~/
   cd ~/
   tar -zxvf vitis-ai_v1.3_dnndk.tar.gz
   cd vitis-ai_v1.3_dnndk/
   ./install.sh
   ```
   h) Go back to ***/home/root*** folder and untar the dnndk example file:<br />
   ```
   cd /home/root
   tar -zxvf vitis-ai_v1.3_dnndk_sample_img.tar.gz
   ```
5.If you use your own system image, you may need to copy dpu.xclbin to /usr/lib first
```
cp /media/sd-mmcblk0p1/dpu.xclbin /usr/lib/
```
6. Go to the vitis_ai_dnndk_samples and run the hello_dpu application:<br />
```
cd /home/root/vitis_ai_dnndk_samples
mkdir test
cd test
cp /media/sd-mmcblk0p1/hello_dpu ./
./hello_dpu
```
***We store the hello_dpu to /card/package/vitis_ai_dnndk_samples/test folder to suit the relative path in my code, you can do that according to your code context. The hello_dpu is generated in Vitis application build and was copied to sd card from previous operation.***<br />
7. You should see the result like below:<br />
![test_result.PNG](/pic_for_readme/test_result.PNG)<br /><br />

***Please refer to UG1144 if you would like to implement a ext4 rootfs.<br />***
## Reference<br />
https://github.com/gewuek/vitis_ai_custom_platform_flow<br />
https://www.xilinx.com/html_docs/xilinx2020_2/vitis_doc/index.html<br />
https://github.com/Xilinx/Vitis-AI<br />
https://github.com/Xilinx/Vitis_Embedded_Platform_Source<br />
https://github.com/Xilinx/Vitis-AI-Tutorials/tree/Vitis-AI-Custom-Platform<br />
***Note: If you would like to try with one click creating VAI platform flow it is recommended to try with the official platform source code for*** [zcu102_dpu](https://github.com/Xilinx/Vitis_Embedded_Platform_Source/tree/master/Xilinx_Official_Platforms/zcu102_base) ***and*** [zcu104_dpu](https://github.com/Xilinx/Vitis_Embedded_Platform_Source/tree/master/Xilinx_Official_Platforms/zcu104_base)***.*** <br /><br /><br />

## More Information about Install and Set Vitis and XRT Environment<br />
https://www.xilinx.com/html_docs/xilinx2020_2/vitis_doc/settingupvitisenvironment.html#zks1565446519267<br />
https://www.xilinx.com/html_docs/xilinx2020_2/vitis_doc/pjr1542153622642.html<br />
https://www.xilinx.com/html_docs/xilinx2020_2/vitis_doc/rbk1547656041291.html<br />




   
   





