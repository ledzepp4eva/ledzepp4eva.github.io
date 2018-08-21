There are various reasons why you might not want an ISP to be able to see what you are doing. Such as if something contains sensitive information, or perhaps they have blocked a website and you want access to it.

One popular way to do this is to use a VPN. What this essentially does is creates a secure tunnel to a VPN end point and allows for your connections to appear to originate from there. Now there are a number of third parties out there that for a small fee will let you connect to their VPN service and off you go. But one disadvantage is although they claim they don't log your traffic, being a third party you can't trust them 100%.

If this is a concern you will have to do it yourself. Luckily this isn't as difficult as it first sounds and below is how I have set this up.

First I start with the actual VPN service. To do this I use a VPS provider, which is (link: https://www.linode.com/ text: Linode), but this can work on any provider as long as they can support docker.

Now once this is up and running with (link: https://get.docker.com/ text: docker installed ) and (link: https://docs.docker.com/compose/ text: docker compose) you can now spin up an openVPN server. Documentation to this is very good and extensive, so I won't go into detail here, but the main sources are
- <https://github.com/kylemanna/docker-openvpn>
- <https://github.com/kylemanna/docker-openvpn/blob/master/docs/docker-compose.md>

I use docker-compose for reason that can be read about (link: https://moncur.eu/notes/docker-all-the-things text: here) , but essentially the image allows an easy set up of the openVPN configuration and the set up of client configurations using certificates.

Now that is set up and running we are going to configure our gateway server. The reason I have done this is I run kodi off an (link: https://www.amazon.co.uk/Amazon-Fire-TV-Stick-Streaming-Media-Player/dp/B00KAKUN3E text: amazon fire stick). The reason for this is that it easy to use for apps like bbc iplayer and netflix, that my wife can use, but also it allows me to side load android apk's on to it. One issue however is that that openVPN requires a third party app, so I have elected to instead of trying to install another app on the firestick, which might not be compatible with the remote, and the stick needs rooting to allow vpn support, and I have an (link: http://www.orangepi.org/orangepipc2/ text: orangepi pc2) running ubuntu 16.04 just sitting around.

First I configure the orangepi to connect to the vpn. Luckily this is as easy as
```
apt update
apt install openvpn screen
screen -S openvpn
openvpn --config /path/to/config/generated/by/docker
```
Now we assign this server a static ip, so we always use this IP as our default gateway
```
/etc/network/interfaces.d/eth0:
auto eth0
iface eth0 inet static
address 192.168.0.15
gateway 192.168.0.1
netmask 255.255.255.0
broadcast 192.168.0.255
dns-nameservers 127.0.1.1
```
Please ensure ip addresses are amended for your own network.

Next we need to enable the server to allow it forward network traffic on via the VPN.

    Enable ip_forward, by adding to /etc/sysctl.conf net.ipv4.ip_forward = 1 then run sysctl -p

We now will use iptables to forward the traffic that comes into eth0 out of tun0 (the openVPN interface)
```
iptables -t nat -A POSTROUTING -o tun0 -j MASQUERADE
iptables -A FORWARD -s 192.168.0.0/24 -i eth0 -o tun0 -m conntrack --ctstate NEW -j ACCEPT
iptables -A FORWARD -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
```
Finally we configure the amazon firestick now to use the orangepi. This is sort of hidden, but it is all done under settings > system > network.
If you are already connected to the wireless network you will need to forget the network and re connect again and enter your password, but don't click connect but click advanced. Then fill in the details by following the wizard and when you get to gateway you will put in 192.168.0.15 (or whatever IP you used above). And that is that.
