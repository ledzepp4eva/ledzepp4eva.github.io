I have a habit of re-installing my laptop or staff virtual machines all the time. This is generally triggered by a new release, be it debian 9.1 or RedHat 7.4. So this time I have been looking at FreeBSD 11.1. I am lucky enough to have access to an openstack enviornment to play, which recently had IPv6 enabled. So as I make sure for all VMs that each use IPv6, this is just because we are only given a limited number of IPv4 addresses (as you can imagine), and I don't want to waste IPv4 addresses, when I have full IPv6 support at home and at work.

Looking online it looks pretty simple as you are taken to some official documenation that is located here [https://www.freebsd.org/doc/handbook/network-ipv6.html|en] and section 30.9.2. Configuring IPv6, so this is what I do:

///
ifconfig_vtnet0_ipv6="inet6 accept_rtadv"
rtsold_enable="YES"
#ipv6_activate_all_interfaces="YES"
ifconfig_vtnet0_ipv6="inet6 2a02:22d0:9:6::a prefixlen 128"
ipv6_defaultrouter="fe80::f816:3eff:feb1:ed64"
///

I reboot the server, but I still don't have any connectivety over ipv6, which is strange as I know this IP information is correct, as I have the exact same configuration on a different server (non freebsd), but with the last 16bits changed. So this is strange. 

By running ifconfig I can see the IP is assigned, so lets check the default route
///
Internet6:
Destination                       Gateway                       Flags     Netif Expire
::/96                             ::1                           UGRS        lo0
default                           fe80::f816:3eff:feb1:ed64     UGS         lo0
::1                               link#2                        UH          lo0
::ffff:0.0.0.0/96                 ::1                           UGRS        lo0
2a02:22d0:9:6::a                  link#1                        UHS         lo0
fe80::/10                         ::1                           UGRS        lo0
fe80::%vtnet0/64                  link#1                        U        vtnet0
fe80::f816:3eff:feb9:ea5e%vtnet0  link#1                        UHS         lo0
fe80::%lo0/64                     link#2                        U           lo0
fe80::1%lo0                       link#2                        UHS         lo0
ff02::/16                         ::1                           UGRS        lo0
///

and the line that stands out to me is
///
default                           fe80::f816:3eff:feb1:ed64     UGS         lo0
///
This looks like the default gw is going out the lo0 interface, which is strange. Luckily there is a quick fix for this that took me an awful lot of searching for in the /etc/rc.conf file

///
ipv6_defaultrouter="fe80::f816:3eff:feb1:ed64%vtnet0"
///

and now when I reboot IPv6 is working.
