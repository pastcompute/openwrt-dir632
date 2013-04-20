This patch adds support for the D-link DIR 632 router (hardware version A1)

Signed-off-by: Andrew McDonnell <bugs@andrewmcdonnell.net>
---
Notes:

This patch is 420kb.  Maybe I am doing something wrong :-|  So iaw the patch submission guidelines I have uploaded it to github for the time being instead of inline here.

This router has 8mb flash, 32m RAM, 8-port switch, WLAN, USB2

This is probably a bit "beta", I have the following bugs I havent managed to
solve yet:

0. To get any network at all I had to port the ag7240 device driver from
DD-WRT.  This brings over more than a dozen files.  I think this is needed
because switchs dont seem to connect properly when using the ar71xx_ag7240
driver already present in OpenWRT. I converted it to a platform device so the
MAC addresses could be passed through from the init function.

1. The switch isn't "switching" - computers plugged into the switch can talk
to the router but not each other.  Hopefully just a boot configuration issue.

2. Wifi needs to be turned on as follows before it will work:

uci set wireless.radio0.disabled=0 ; wifi up

3. Kernel - the wlan LED doesnt work yet

4. Kernel - the Power/Status LED only works in Orange (should be able to be
Green as well)

5. Kernel - the WAN LED only works in Green (should be able to be in Orange as
well)

6. Kernel - I was getting a 'warn_slowpath_common' stacktrace once in the
kernel when the first packet is processed in the router, but it doesnt seem to
affect performance

7. MAC addresses - I am not sure what the policy should be - the factory sets
them all to the stickered MAC address, but DD-WRT reads a different MAC
address from the ART partition.  Currently I set the stickered MAC as the
default for the WLAN, and use the ART for default eth0, and the ART 6th
octet+1 for default eth1.

8. The LEDS dont seem to be doing the right thing in user space - at one point
I had power/status flashing at boot but it doesn't currently

 target/linux/ar71xx/base-files/etc/diag.sh         |    3 +
 .../ar71xx/base-files/etc/uci-defaults/01_leds     |    6 +
 .../ar71xx/base-files/etc/uci-defaults/02_network  |    8 +
 target/linux/ar71xx/base-files/lib/ar71xx.sh       |    3 +
 .../ar71xx/base-files/lib/upgrade/platform.sh      |    1 +
 target/linux/ar71xx/config-3.8                     |   26 +-
 .../ar71xx/files/arch/mips/ath79/mach-dir-632-a1.c |  403 +++
 .../mips/include/asm/mach-ath79/ag71xx_platform.h  |    6 +
 .../drivers/net/ethernet/atheros/ag7240/Kconfig    |  124 +
 .../drivers/net/ethernet/atheros/ag7240/Makefile   |   14 +
 .../drivers/net/ethernet/atheros/ag7240/ag7240.c   | 2584 ++++++++++++++++++++
 .../drivers/net/ethernet/atheros/ag7240/ag7240.h   |  706 ++++++
 .../net/ethernet/atheros/ag7240/ag7240_phy.h       |  169 ++
 .../net/ethernet/atheros/ag7240/ag7240_trc.h       |   99 +
 .../drivers/net/ethernet/atheros/ag7240/ar7240.h   |  984 ++++++++
 .../net/ethernet/atheros/ag7240/ar7240_s26_phy.c   | 1431 +++++++++++
 .../ethernet/atheros/ag7240/ar7240_s26_phy.c.new   | 1386 +++++++++++
 .../net/ethernet/atheros/ag7240/ar7240_s26_phy.h   |  224 ++
 .../drivers/net/ethernet/atheros/ag7240/ar8216.c   |  843 +++++++
 .../drivers/net/ethernet/atheros/ag7240/ar8216.h   |  187 ++
 .../net/ethernet/atheros/ag7240/athrf1_phy.c       |  385 +++
 .../net/ethernet/atheros/ag7240/athrf1_phy.h       |   26 +
 .../net/ethernet/atheros/ag7240/athrs16_phy.c      |  896 +++++++
 .../net/ethernet/atheros/ag7240/athrs16_phy.h      |  245 ++
 .../net/ethernet/atheros/ag7240/athrs_ioctl.h      |  135 +
 .../net/ethernet/atheros/ag7240/athrs_qos.c        |  196 ++
 .../net/ethernet/atheros/ag7240/athrs_qos.h        |  289 +++
 .../drivers/net/ethernet/atheros/ag7240/eth_diag.h |   65 +
 .../net/ethernet/atheros/ag7240/python_vlan_igmp.c |  686 ++++++
 .../net/ethernet/atheros/ag7240/rtl8309g_phy.c     |  245 ++
 .../net/ethernet/atheros/ag7240/rtl8309g_phy.h     |   35 +
 .../net/ethernet/atheros/ag7240/tools/Makefile     |   47 +
 .../net/ethernet/atheros/ag7240/tools/Makefile.inc |   84 +
 .../net/ethernet/atheros/ag7240/tools/ethreg.c     |  141 ++
 target/linux/ar71xx/generic/profiles/d-link.mk     |   11 +
 target/linux/ar71xx/image/Makefile                 |   12 +
 .../617-MIPS-ath79-add-DIR-632-A1-support.patch    |   39 +
 .../620-net-add-ag7240-driver-for-DIR-632-A1.patch |   19 +
 38 files changed, 12760 insertions(+), 3 deletions(-)



