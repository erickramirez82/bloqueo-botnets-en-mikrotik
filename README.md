# bloqueo-botnets-en-mikrotik
Control y bloqueo de botnets. Bloqueo de escaneo de puertos en Mikrotik

Tienen protocolos preferidos para el ataque, como Telnet, NTP, SSH; a pesar de la preferencias hay una gran lista que según la temporada son más afectados que otros. A continuación daré una pequeña lista de los protocolos que preferentemente suelen atacar.

* FTP (TCP/20,21)
* SSH (TCP/22)
* Telnet (TCP/23,2323)
* DNS (UDP/53)
* TFTP (TCP/69)
* HTTP (TCP/80,8080,8880)
* NTP (UDP/123)
* POP3 (TCP/110,995)
* IMAP (TCP/143,220,993)
* HTTPS (TCP/443)
* RDP (TCP/3389)
* SMTP (TCP/25,465,587)
* Winbox (TCP/8291)
* VNC (TCP/5500,5800,5900)

## Bloqueo Telnet 

```
/ip firewall filter
add action=add-src-to-address-list address-list=drop_telnet address-list-timeout=1w chain=input \
    comment="List Telnet to WAN" dst-port=23,2323 in-interface-list=WAN protocol=tcp  place-before=1
add action=drop chain=input src-address-list=drop_telnet comment="Drop list Telnet to WAN"  place-before=1
```

## Bloqueo SSH

```
/ip firewall filter
add action=add-src-to-address-list address-list=drop_ssh address-list-timeout=1w chain=input \
    comment="List SHH to WAN" dst-port=22 in-interface-list=WAN protocol=tcp  place-before=1
add action=drop chain=input src-address-list=drop_ssh comment="Drop List SHH to WAN"  place-before=1
```


## Bloqueo escaneo de puertos

```
/ip firewall filter
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input comment="Port scanners to list" disabled=no protocol=tcp psd=21,3s,3,1 place-before=1
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input comment="NMAP FIN Stealth scan" place-before=1 disabled=no protocol=tcp tcp-flags=fin,!syn,!rst,!psh,!ack,!urg
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input \
    comment="SYN/FIN scan" disabled=no protocol=tcp tcp-flags=fin,syn place-before=1
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input \
    comment="SYN/RST scan" disabled=no protocol=tcp tcp-flags=syn,rst place-before=1
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input \
    comment="FIN/PSH/URG scan" disabled=no protocol=tcp tcp-flags=fin,psh,urg,!syn,!rst,!ack place-before=1
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input \
    comment="ALL/ALL scan" disabled=no protocol=tcp tcp-flags=fin,syn,rst,psh,ack,urg place-before=1
add action=add-src-to-address-list address-list="port scanners" address-list-timeout=2w chain=input \
    comment="NMAP NULL scan fin los cambias por tcp-flags=!fin" disabled=no protocol=tcp tcp-flags=fin,!syn,!rst,!psh,!ack,!urg place-before=1
add action=drop chain=input comment="dropping port scanners" disabled=no src-address-list="port scanners" place-before=1
```

## Bloqueo y proteccion ataque ddos 

```
/ip firewall address-list
add list=ddos-attackers
add list=ddos-targets

/ip firewall filter
add action=return chain=detect-ddos dst-limit=32,32,src-and-dst-addresses/10s place-before=1 comment="Ataque DDOS"
add action=add-dst-to-address-list address-list=ddos-targets address-list-timeout=10m chain=detect-ddos place-before=1 comment=" deteccion Ataque DDOS"
add action=add-src-to-address-list address-list=ddos-attackers address-list-timeout=10m chain=detect-ddos place-before=1 comment="deteccion Ataque DDOS"
/ip firewall raw
add action=drop chain=prerouting dst-address-list=ddos-targets src-address-list=ddos-attackers place-before=1 comment="bloqueo Ataque DDOS"
```

