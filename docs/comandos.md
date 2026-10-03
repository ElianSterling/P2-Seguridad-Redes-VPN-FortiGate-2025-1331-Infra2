# Comandos — Infraestructura 2

## Proxmox / cliente

~~~bash
pct config 101
pct exec 101 -- ip neigh
pct exec 101 -- ping -c 4 172.16.60.2
pct exec 101 -- nc -nvz -w 5 172.16.60.2 22
~~~

## FortiGate

~~~text
get system arp
get router info routing-table details 172.16.50.10
get router info routing-table details 172.16.60.2
show router static

diagnose hardware deviceinfo nic port2
diagnose hardware deviceinfo nic port3

get vpn ipsec tunnel summary
diagnose vpn ike gateway list
diagnose vpn tunnel list

show vpn ipsec phase1-interface
show vpn ipsec phase2-interface
~~~

## Sniffer IKE

~~~text
diagnose sniffer packet port2 'host 192.0.2.6 and udp port 500' 4 0 a
~~~

## Debug IKE

~~~text
diagnose debug reset
diagnose debug console timestamp enable
diagnose debug application ike -1
diagnose debug enable

# generar tráfico

diagnose debug disable
~~~

## Cisco R3

~~~text
show ip interface brief
show run interface GigabitEthernet0/1
show run | section crypto
show access-lists
show crypto map
show crypto isakmp sa
show crypto ipsec sa
~~~

## Generación de tráfico

~~~text
ping 172.16.50.10 source 172.16.60.1
~~~

Las PSK se sustituyen por <REDACTED> en cualquier configuración publicada.
