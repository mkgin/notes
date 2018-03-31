# Enable IPv6 Privacy Extensions


```
net.ipv6.conf.all.use_tempaddr = 2
net.ipv6.conf.default.use_tempaddr = 2
...
net.ipv6.conf.ifaceX.use_tempaddr = 2
net.ipv6.conf.ifaceY.use_tempaddr = 2
```
to ```/etc/sysctl.d/55-ipv6.conf```
(where ifaceX,Y ... are netowork card names)

Reload with ```sysctl --system```