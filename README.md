# openwrt-packages

Add feed to feeds.conf.default: `src-git mrhaav https://github.com/mrhaav/openwrt-packages.git`\
\
\
**leds-apu1**

Kernel module to access the three front LEDs and the push button on the PC Engines APU1.\
LEDs are called apu:1, apu:2 and apu:3. The push buttom is accessible via GPIO and `cat /sys/class/gpio/gpio187/value`. 1 = unpressed, 0 = pressed.\
You can use packages like `kmod-ledtrig-netdev` to trigger the LEDs for network activity.\
\
\
**LoopiaAPI**

Hotplug script for updating Loopia DNS via LoopiaAPI, https://www.loopia.com/api/. \
LoopiaAPI is triggered from `etc/udhcpc.user` and `etc/hotplug.d/iface/90-loopia`. `udhcpc.user` is triggered when the DHCP client is updating the IP-address and `90-loopia` is triggered when an interface is updated with ip address and route. `90-loopia` could be usefull for LTE modem if they doesn´t support DCHP.\
Messages are written to System Log.\
\
You need to configure your Loopia API username and password and "connect" the interface to your domain name.
```
uci set ddns.loopiaapi.username='user@loopiaapi'
uci set ddns.loopiaapi.password='password'
uci set ddns.loopiaapi.wan='your.domain.com'
uci commit ddns
```
\
As long as the DNS has correct information nothing is sent to Loopia. You can force regulary updates, in days, with:
```
uci set ddns.loopiaapi.forced_update='30'
uci commit ddns
```
Packages dependencies:\
`curl`
`libxml2-utils`

libxml2-utils is missing in 19.07. You can download from 21.02 and install manually or compile from here. "Override" 19.07 version with:\
`scripts/feeds uninstall libxml2`\
`scripts/feeds install -p mrhaav libxml2`\
\
\
**luci - luci-proto-umbim**

Just a copy of luci-proto-uqmi.\
\
\
**me909s**

AT command script for Huawei ME909s-120 LTE modem.

You need to configure you wwan interface:\
Network - Interfaces - wwan\
Edit - Firewall Settings\
&nbsp;&nbsp;&nbsp;Create / Assign firewall-zone: Add wwan to correct firewall-zone
	
Add APN setting:
```
uci set network.wwan.apn=internet
uci set network.wwan.pdp_type=IP
uci commit network
```

The script will check connectivity (ping 8.8.8.8) every 600 sec, by default.\
You can change the intervall with:
```
uci set network.wwan.check_timer=xx
uci commit network
```
Reboot router\
\
Packages dependencies:\
`kmod-usb-net-cdc-ether`
`kmod-usb-serial-option`
`comgt`\
\
\
**r8168**

NIC drivers to Realtek RTL8111E with support for customized LEDs. Designed for PC Engines APU1.\
The APU1 board has LED0 (green) and LED1 (amber) connected. Default flashes LED0 for network activity, for all speeds, and LED1 is lit for Link100M.
I have change the drivers so LED0 is lit for Link, all speeds, and flashes for network activity, all speeds, and LED1 is lit for Link1G.\
That equals hex-word 0x004F, according to table:
|      | Activity | Link1G | Link100M | Link10M |
| --- | :---: | :---: | :---: | :---: | 
| LED0 | Bit3 | Bit2 | Bit1 | Bit0 |
| LED1 | Bit7 | Bit6 | Bit5 | Bit4 |
| N/A | Bit11 | Bit10 | Bit9 | Bit8 |
| LED3 | Bit15 | Bit14 | Bit13 | Bit12 |

If you want a different behavior, just change the hex-word in file r8168_n.c under section "Enable Custom LEDs".

Files "Realtek_" are the original files from the driver package, https://www.realtek.com/en/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software
\
\
\
**uqmi**

Control utility for mobile broadband modems. Based on https://git.openwrt.org/project/uqmi.git 2021-11-22.\
This version use APN profiles. The default APN profile is verified before the modem goes online. If the default profile is not correct, the modem is set to Airplane mode on and the profile is corrected. Then the modem is set to Airplane mode off.\
If PDP-type = IPv4v6, dual-stack will be activate. Default APN profile is defined with IPv4 and a secondary profile is defined with IPv6.

PKG_RELEASE:=0.4\
PKG_VERSION:=2021-12-22
- qmi.sh: Some minor cosmetic corrections.

PKG_RELEASE:=0.3\
PKG_VERSION:=2021-12-22
- qmi.sh: Added support for PLMN configuration and PIN code.

PKG_RELEASE:=0.2\
PKG_VERSION:=2021-12-22
- qmi.sh: Support for dual-stack. PDP Type = IPv4v6 will use IPv4 for default connection and IPv6 for secondary connection. PLMN configuration not supported.

- nas: Add support for three digit MNC even if the value is less then 100. MNC = 008 will be a three-digit and 08 will be a two-digit MNC.
- wds: Added command: --create-profile
- nas: Add decoding of lte_system_info_v2.cid and intrafrequency_lte_info_v2.global_cell_id to enodeb_id and cell_id and decoding of wcdma_system_info_v2.cid to rnc_id and cell_id. Change order to mcc-mnc-tac/lac-enodeb_id/rnc_id-cell_id.
- wds: Added command: --get-default-profile-number, --get-profile-settings, --modify-profile
- dms: Added command: --get-device-operating-mode


Compiling:\
If you don´t find uqmi with desciption: `Control utility for mobile broadband modems, mod by mrhaav` in `make menuconfig` you need to "override" official uqmi.\
`scripts/feeds uninstall uqmi`\
`scripts/feeds install -p mrhaav uqmi`
