# QNAP 301w Non-Disassembly Flashing Method

#### Special thanks to [@coolsnowwolf](https://github.com/coolsnowwolf), [@asushugo](https://github.com/asushugo), and all the awesome developers for adapting the 301w router!

#### This tutorial has been tested and verified only on lean's qsdk closed-source firmware and [lede open-source firmware](https://github.com/coolsnowwolf/lede). Please test independently for derivative firmwares.

#### [QNAP QHora-301W Product Introduction](./301w_Specs.md)  
#### [QNAP QHora-301W OpenWRT Official Introduction](https://openwrt.org/inbox/toh/qnap/301w) 
#### [QNAP QHora-301W Third-party Web UI & Original Dual-Partition Uboot Support](./uboot.md)  

### I. Enable SSH Service

While the router is powered on and the system is running normally, press and hold the WPS button on the back of the router until you hear the **second** "beep" (approximately 12 seconds), then release.

---

### II. SSH Connection to Router Backend

Please note the following:

- The default SSH port enabled by the router is **22200**.

- The username is `admin`, and the password is your router's web login password.

The ssh command is `ssh admin@192.168.100.1 -p 22200` (If the port is not open, repeat Step 1).

Alternatively, use PuTTY to connect.

![putty](pic/putty.jpg)

---

### III. Switch Boot Partition to Second Partition

`sudo fw_setenv current_entry 1` **The password requested here is the same one you used to log into SSH; this will not be mentioned again.**

`sudo reboot` Restart

---

### IV. Check Current Router Boot Partition

*Follow Step II again to reopen SSH.*

`sudo fw_printenv -n current_entry`

Check the partition to ensure it outputs `1`. If not, repeat Step III.

---

### V. Use WinSCP to upload QSDK's kernel.bin and rootfs.bin to /tmp 

The kernel and rootfs files can be obtained by extracting the sysupgrade format firmware.

---

### VI. Flash QSDK to the First Partition using dd

*Note: It is strongly recommended to use the dd command (search online for methods) to back up important partitions, such as mtd flash partitions and mmc flash partitions, so that you can [restore the official firmware](./recovery_oem.md) in the future.*

```sh
sudo dd if=/tmp/kernel.bin of=/dev/mmcblk0p1
sudo dd if=/tmp/rootfs.bin of=/dev/mmcblk0p4
sudo fw_setenv current_entry 0
sudo fw_setenv boot_0 good
sudo reboot
```
Reference screenshot:

![putty](pic/flash_qsdk.jpg)

After executing the above commands, QSDK will be flashed and will boot from the first partition. Note that since the 10G PHY firmware has not been flashed yet, you must flash the firmware.

---

### VII. Flash 10G PHY Firmware

1. Transfer `AQR_ethphyfw_5.6.7.mbn` to the QSDK `/tmp` directory using the scp command or WinSCP tool.
2. Access the router backend via PuTTY or other SSH tools and erase the data in the original ethfw partition mtd10: `mtd erase /dev/mtd10`
3. Flash the fw file: `mtd -n write /tmp/AQR_ethphyfw_5.6.7.mbn /dev/mtd10`
4. Modify the `bootcmd` environment variable: `fw_setenv bootcmd "aq_load_fw 0; aq_load_fw 8; bootipq"`
5. Run `fw_printenv` to check if there is a record: `bootcmd=aq_load_fw 0; aq_load_fw 8; bootipq`. If it exists, it is correct.
6. Reboot
