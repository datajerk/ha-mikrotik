# ha-mikrotik (Tested stable)
High availability code for Mikrotik routers

# Status: April 30th 2017
This has been tested stable across 6 different pairs of CCR1009s for over a year, there have been multiple adminstrative failover events and a few cases of hardware failovers. The primary RouterOS code is v6.38.1 and there are instances of v6.39 which seem to work without issue.

# Warning
Please do not test this on production routers. This should be tested in a lab setup with complete out of band serial access.
This was developed on the CCR1009-8g-1s-1s+ and is in use in our production environment. Proceed at your own risk, the code can potentionally wipe out 
all of your configuration and files on your device.

Extensive documentation is still needed. This is being delivered as a proof of concept.
You will need to do a bit of code reading and testing to figure out how it works.

# Issues
The #1 issue is a race condition during the startup of the secondary after it gets an updated configuration. It needs to quickly disable all of the interfaces
so that it doesn't end up taking traffic (MACs are cloned) from the active router. If you use spanning tree on your switches, it is likely that this
will happen fast enough and the Layer2/3 won't have time to come up and cause issues. Test this very carefully, you will get very strange results if your ports
start forwarding instantly from your upstream switch.

# Concept
Using a dedicated interface, VRRP, scripts, and backups, we can make a pair of Mikrotik routers highly available.
Configuration and files are actively synchronized to the standby and the standby remains ready to takeover when the VRRP heartbeat fails.

# Hardware originally developed for
Pair of CCR1009-8g-1s-1s+
RouterOS v6.33.5
Routerboard firmware 3.27
Bootstrapped from complete erased routers and then config built up once HA installed.

# Installing
1. Source a pair of matching routers, ideally CCR1009-8g-1s-1s+.
2. Install RouterOS v6.33.5 and make sure the Routerboard firmware is up date.
3. Ensure you have serial connections to both devices.
4. Reset both routers using the command:
`/file remove [find]; /system reset-configuration keep-users=no no-defaults=yes skip-backup=yes`
5. Connect an ethernet cable between ether8 and ether8.
6. On one router, configure a basic network interface on ether1 with an IP address of your choosing. Just enough to be able to copy a file.
7. Upload HA_init.rsc and import it:
`/import HA_init.rsc`
8. Install HA (note to replace the fields of macA, macB, and password. I suggest a sha1 hex hash for the password.
`$HAInstall interface="ether8" macA="[MAC_OF_A_ETHER8]" macB="[MAC_OF_B_ETHER_8]" password="[A RANDOM PASSWORD OF YOUR CHOOSING]"`
9. Follow the instructions given by $HAInstall to bootstrap the secondary. I use the MAC telnet that is suggested at the top but any other method is sufficient.
10. Once router B is bootstrapped, it will reboot itself into a basic networking mode. It needs to be pushed the current configuration.
`$HASyncStandby`
11. B will now reboot and when it returns, it should be in standby mode. A should be the active router. You can now reboot A and B will takeover.
