# Installing Gentoo Linux on a T2 Mac

## Hardware Requirements

* Gentoo Minimal Installation ISO
    * Requirements for this ISO: USB keyboard, mouse, ethernet adapter/Wi-Fi adapter
    * If you don't meet the requirements, then you are required to use another distro's ISO. A recommended distro to use is Ubuntu, more specifically, [mbp-ubuntu](https://wiki.t2linux.org/distributions/ubuntu/installation/#download-the-latest-safe-release).
        * Make sure to follow [this guide](https://wiki.gentoo.org/wiki/Installation_alternatives#Installation_from_non-Gentoo_LiveCDs) along with this guide and the Gentoo Handbook starting at step 5
* USB-C to USB-A adapter
* USB Flash Drive

## Install Procedure

1. Partition your SSD

    1. Open the Bootcamp installer and follow along until it requests for you to input a Windows ISO. This should clear space for a Linux partition because of removed APFS snapshots.
    2. Now open up Disk Utility. Make a partition with any format. The amount of space you allocate for this Linux partition will be final, so choose wisely.

2. Download a Gentoo ISO and flash it to your USB Flash Drive via [Balena Etcher](https://www.balena.io/etcher/) (you can also use dd)
3. Ensure that Secure Boot is disabled

    1. Follow the instructions from [this article](https://support.apple.com/en-us/HT208198)
    2. Once in the Startup Security Utility, set secure boot to no security and enable external boot.

4. Boot into the Live USB enviornment

    1. Plug in the USB Flash Drive into your computer
    2. Startup while holding the Option key, this will put you in the macOS Startup Manager
    3. Select the orange EFI Boot option and press enter to boot into it. (If you're using a Ubuntu Live environment, then make sure to select the orange EFI Boot option all the way to the right)

5. Follow the [Gentoo Handbook](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Disks)

    1. Instead of making a new EFI partition for Gentoo, you should instead mount `/dev/nvme0n1p1` to `/mnt/gentoo/boot/efi`
    2. Make sure that you have deleted the partition you made ahead of time, or else you'll lose important data

6. Continue through the Handbook until chapter "Configuring the kernel"

    1. In order to have the Linux Kernel support your device the most, you'll have to build from a modified Linux Kernel source tree.
    2. Git clone [the t2linux kernel source tree](https://github.com/t2linux/kernel) to `/usr/src`
        * Make sure to git checkout the version you want. For example, if you want kernel version v5.16.17, you would checkout tag `t2-v5.16.17`.
    3. Run these commands to symlink the kernel to `/usr/src/linux`:

    ```bash
    eselect kernel list # there should be one option that shows up
    eselect kernel set 1
    ls -l /usr/src/linux # check if the symlink succeded
    ```

    1. Run these commands to grab apple-bce and apple-ibridge:

    ```bash
    git clone https://github.com/t2linux/apple-bce-drv /usr/src/apple-bce
    git clone https://github.com/t2linux/apple-ib-drv /usr/src/apple-ibridge
    for i in apple-bce apple-ibridge; do 
    mkdir /usr/src/linux/drivers/staging/$i 
    cp -r /usr/src/$i/* /usr/src/linux/drivers/staging/$i/
    done
    ```

    1. Run these commands in order to include apple-bce and apple-ibridge into the kernel source tree:

    ```bash
    git clone https://github.com/Redecorating/mbp-16.1-linux-wifi /linux-patches
    cd /usr/src/linux
    for i in /linux-patches/*.patch; do
    echo $i
    patch -Np1 < $i
    done
    ```

    1. Git clone the `https://anongit.gentoo.org/git/proj/linux-patches.git` repo to a folder called `/gentoo-patches`. This repo includes Gentoo patches for the kernel source tree usually included with the `gentoo-sources` Portage package.
    2. Apply them with a modified version of the commands used above.
    3. Copy the default config from the patches repo to the kernel source tree. If you do this, please make sure to set any filesystem drivers you want to use (like for ext4) to be built-in instead of being a module with `make menuconfig`
    4. Build with `make all`. If you want to speed up the build process, add `-j$(nproc)`

7. Before finishing, if you want to connect to the internet later, now is a good time to install `NetworkManager` and optionally `iwd`.
8. Continue through the Handbook until chapter "Configuring the bootloader"

    1. Install grub by using `emerge --ask --verbose sys-boot/grub`
    2. Edit the config file `/etc/default/grub` with your favorite text editor of choice (i.e. vim or nano)
    3. On the line with `GRUB_CMDLINE_LINUX`, append the following parameters: `intel_iommu=on iommu=pt pcie_ports=compat`
    4. Run `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB --no-nvram --removable` to install Grub.
    5. Run `grub-mkconfig -o /boot/grub/grub.cfg` to make the config files for Grub

9. You're done! You should now be able to boot into Gentoo via the macOS Startup Manager
10. If you confirmed that Gentoo does bootup no problem, then you can boot into macOS and follow the [Wi-Fi guide](https://wiki.t2linux.org/guides/wifi/)
