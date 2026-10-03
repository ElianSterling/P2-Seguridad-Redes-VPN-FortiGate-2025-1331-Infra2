# Infraestructura 2 — VPN IPsec FortiGate ↔ Cisco R3

**Video:** https://youtu.be/zmOdv6nX4SU

## 1. Objetivo

Implementar un túnel IPsec entre FGT-02 y Cisco R3 para permitir que la red 172.16.50.0/25 alcance la red 172.16.60.0/28.

## 2. Topología

![Topología](../diagrams/topologia-infraestructura-2.svg)

## 3. Direccionamiento

| Equipo / interfaz | Dirección | Función |
|---|---|---|
| USER-01 | 172.16.50.10/25 | Cliente |
| FGT-02 port3 | 172.16.50.1/25 | LAN usuarios |
| FGT-02 port2 | 198.51.100.2/30 | WAN |
| Cisco R3 Gi0/1 | 192.0.2.6/30 | WAN |
| Cisco R3 Gi0/0 | 172.16.60.1/28 | LAN servidor |
| WEB-01 | 172.16.60.2/28 | Servidor |

## 4. Proxmox / interfaces

| Interfaz | Función | Bridge |
|---|---|---|
| FGT-02 port1 | WAN/administración | vmbr0 |
| FGT-02 port2 | WAN hacia R3 | vmbr5 |
| FGT-02 port3 | LAN usuarios | vmbr6 |

USER-01 quedó en vmbr6 con 172.16.50.10/25 y gateway 172.16.50.1.

## 5. Phase 1

~~~text
config vpn ipsec phase1-interface
    edit "VPN-FGT02-R3"
        set interface "port2"
        set peertype any
        set net-device disable
        set proposal des-sha1
        set dhgrp 14
        set remote-gw 192.0.2.6
        set psksecret <REDACTED>
    next
end
~~~

Parámetros: IKEv1, PSK, DES/SHA-1, DH14 y peer 192.0.2.6.

## 6. Phase 2

~~~text
config vpn ipsec phase2-interface
    edit "VPN-FGT02-R3"
        set phase1name "VPN-FGT02-R3"
        set proposal des-sha1
        set dhgrp 14
        set src-subnet 172.16.50.0 255.255.255.128
        set dst-subnet 172.16.60.0 255.255.255.240
    next
end
~~~

Selectores:

~~~text
LOCAL : 172.16.50.0/25
REMOTE: 172.16.60.0/28
~~~

## 7. Cisco R3

El crypto map se aplicó a GigabitEthernet0/1.

Elementos documentados:

- Peer: 198.51.100.2.
- Tráfico interesante: 172.16.60.0/28 ↔ 172.16.50.0/25.
- ESP DES con SHA-HMAC.
- PFS/DH14.
- PSK configurada pero no publicada.

## 8. Negociación y estado final

Durante el troubleshooting se observaron retransmisiones y posteriormente la autenticación PSK correcta. La negociación llegó a IKE SA operativo y Quick Mode creó el IPsec SA con los selectores correctos.

Estado final de FGT-02:

~~~text
'VPN-FGT02-R3' 192.0.2.6:0
selectors(total,up): 1/1
~~~

En R3 se observó la SA en estado QM_IDLE y tráfico IPsec contabilizado.

## 9. Prueba de conectividad

USER-01 alcanzó 172.16.60.2 mediante ICMP:

~~~text
4 packets transmitted
4 received
0% packet loss
~~~

La prueba final específica de SSH a 172.16.60.2:22 terminó en timeout y se decidió no continuar modificando ni probando SSH. Por ello, SSH no se presenta como servicio validado de Infraestructura 2.

## 10. Troubleshooting

La investigación se realizó por capas: L2/ARP, routing, llegada al port3, forward policy, IKE Phase 1, Quick Mode/Phase 2, IPsec SA y contadores de tráfico.

## 11. Resultado

La Infraestructura 2 demuestra interoperabilidad IPsec entre FortiGate y Cisco IOS, incluyendo IKEv1, PSK, Phase 2, selectores, routing y cifrado de tráfico entre las redes privadas.
