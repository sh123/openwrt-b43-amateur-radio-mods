# OpenWRT backfire 10.03 b43 drivers mods

## Introduction
**This modification might be illegal in your country. You can use
router with this modification only if you have amateur radio license. You
cannot use encryption and you need to make sure your callsign is present 
inside packets transmitted**

The goal is to modify OpenWRT Backfire 10.03 to make router work outside of 
ISM standard WiFi channels on 13 cm amateur band (channels 0 and probably
-1, -2 if possible) and try to setup a link between two routers using one of 
Mesh network frameworks.

By using legacy 11 MHz channel width mode and work on channel 0, 2407 MHz 
router will be still within ISM band, it won’t be visible by causal access 
points and at the same it will be within 13 cm amateur band, thus can be used 
legally with directional antennas and higher E.I.R.P. (maximum allowed is mostly 100 mW).

## Userful read
OpenWRT documentation and other useful links:

* WRT54G – <https://wiki.openwrt.org/toh/linksys/wrt54g>
* Build system installation – <https://wiki.openwrt.org/doc/howto/buildroot.exigence>
* Broadband Hamnet – <http://www.broadband-hamnet.org/>
* OLSR mesh – <http://www.olsr.org/>
* Open mesh – <http://www.open-mesh.com>
* Bad flash recovery – <http://www.dd-wrt.com/wiki/index.php/Recover_from_a_Bad_Flash>

## Step-by-step instruction on patching OpenWRT Backfire 10.03

1. In case of problems compiling OpenWRT on new Debian/Ubuntu distributions it is possible to use [debootstrap](https://wiki.debian.org/Debootstrap) to install older Debian chroot environment.
    ```
    cd ~
    sudo su
    mkdir debian_old
    debootstrap oldstable openwrt_debian
    chroot debian_old
    adduser openwrt
    ```

2. Install packages required for OpenWRT build system
    ```
    apt-get update
    apt-get install git-core build-essential libssl-dev libncurses5-dev unzip gawk zlib1g-dev subversion file python bison gawk sudo vim flex python-m2crypto
    ```

3. Fetch OpenWRT backfire 10.03 sources and apply patches
    ```
    cd
    git clone git://git.openwrt.org/10.03/openwrt.git backfire
    git clone https://github.com/danitool/openwrt-legacy-buildroot-fixes
    git clone git://git.kernel.org/pub/scm/linux/kernel/git/sforshee/wireless-regdb.git
    cp openwrt-legacy-buildroot-fixes/* backfire/
    cd backfire
    ./patch-br.sh
    ```

4. Update and install feeds for packages and luci
    ```
    scripts/feeds update packages
    scripts/feeds update luci
    scripts/feeds install -a -p packages
    scripts/feeds install -a -p luci
    ```

5. Configure OpenWRT
    ```
    make menuconfig
    ```
    Target System - Broadcom BCM947xx/953xx
    Target Profile - Broadcom BCM43xx WiFi
    Select additional packages, such as batman-adv, luci, batmand, olsr

6. Build OpenWRT
    ```
    make
    ```

7. Modify regdb regulatory database and regenerate regulatory.bin
    ```
    cd wireless-regdb
    patch < db.txt.patch
    make
    cp cert.pem ../backfire/build_dir/linux-brcm47xx/crds-1.1.1/pubkeys
    ```

8. Patch b43 broadcom drivers
    ```
    cd backfire
    patch < b43.batch
    ```

9. Patch wpad-mini
    ```
    patch < wpad-mini.patch
    ```

10. Patch luci
    ```
    patch < luci.patch
    ```

11. Rebuild OpenWRT with changes
    ```
    make
    ```

12. Flash image to router, boot, set password thus enabling ssh

13. Copy/scp updated regulatory.bin to router’s /usr/lib/crda/

14. Reboot
