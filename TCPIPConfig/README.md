# TCP/IP-configuratie

## Basis

Verplicht:

- **IP-adres**
- **subnetmasker**

Optioneel:

- **standaard-gateway**

Vanaf dan heb je Internet-connectiviteit: `ping 8.8.8.8` werkt!

Soms wordt hiet ook nog het adres van een DNS-server onder verstaan maar strict genomen is dit niet

DNS-servers van Google:

- `8.8.8.8`
- `8.8.4.4`

> `ping 8.8.8.8` is een veelgebruikte test om op een client internet-connectiviteit te testen.

## Klasses van IP-adressen
![](IPAdressClasses.png)

Dit zijn enkele voorbeelden van 
- Class A-adressen:
	- `1.0.0.1/8`
	- `125.240.200.8/8`
	- ..
	- Gereserveerd voor privaat gebruik: `10.0.0.0/8`
	- Gereserveerd voor local loopback: `127.0.0.0/8` (verkeer wordt nooit op een netwerkkabel gezet)
- Class B-adressen:
	- `128.0.0.1/16`
	- `191.255.205.3/16`
	- Gereserveerd voor privaat gebruik: `172.16.0.0/12` - `172.31.255.255/12` (opgelet: `/12`!)
- Class C-adressen
	- `192.0.4.5/24`
	- `223.4.67.2/24`
	- Gereserveerd voor privaat gebruik: `192.168.0.0/24`

> De classes zijn eigenlijk grotendeels verouderd omdat door het grote tekort aan **IPv4**-adressen overgeschakeld is naar **VLSM** en **CIDR** waarbij men zich niet meer strict houdt aan de subnetmaskergrenzen `/8`, `/16` en `/24`.
> Zie (ter info) http://vlsmcalc.net/

> **IPv6**-adressen hebben hun eigen regels maar zijn voorbereid op de verdere uitbreiding van het Internet.

## APIPA

**Automatic Private IP Addressing**

Wanneer een host staat ingesteld op automatisch een IP-adres verkrijgen maar geen DHCP-server gevonden wordt en de host zichzelf dan maar een IP-adres geeft.

Hiervoor is een gans IP-netwerk gereserveerd: `169.254.0.0/16` (Class B)

## DHCP

**Dynamic Host Configuration Protocol**

De DHCP-server deelt TCP/IP-configuraties uit aan de DHCP-clients die voor een bepaalde tijd (**lease time**) geldig zijn.

Naast de noodzakelijke informatie (IP-adres en subnetmasker), kunnen extra **options** worden meegegeven. Elk option heeft een eigen nummer:
- DHCP option 1: subnet mask
- DHCP option 2: hostname
- DHCP option 3: default gateway
- DHCP option 6: lijst van DNS servers
- DHCP option 42: lijst van NTP servers
- DHCP option 51: lease time
- ...

Berichtenuitwisseling:
- Bij nieuwe IP-leases: **DORA**
	- Discover
	- Offer
	- Request
	- Acknowledge
- Bij vernieuwing/verlenging van een bestaande (nog niet afgelopen) IP-lease: **RA**
	- Request
	- Acknowledge


## Subnet-berekeningen

Analyse van `192.168.0.1/24`:

```
Address:   192.168.0.1           11000000.10101000.00000000 .00000001  
Netmask:   255.255.255.0 = 24    11111111.11111111.11111111 .00000000  
Wildcard:  0.0.0.255             00000000.00000000.00000000 .11111111  
=>
Network:   192.168.0.0/24        11000000.10101000.00000000 .00000000 (Class C)  
Broadcast: 192.168.0.255         11000000.10101000.00000000 .11111111  
HostMin:   192.168.0.1           11000000.10101000.00000000 .00000001  
HostMax:   192.168.0.254         11000000.10101000.00000000 .11111110  
Hosts/Net: 254                   (Private Internet)

```

*Gegenereerd met https://jodies.de/ipcalc*

Elk netwerk (b.v. `192.168.0.0/24`) heef altijd 1 broadcast-adres (b.v. `192.168.0.255`). We spreken over het aantal **net-bits** (b.v. 24) en het aantal **host-bits** (b.v. 8). Dat laatste bepaalt hoeveel hosts er maximaal in het netwerk passen maar er zijn 2 adressen bezet, nl.
- het netwerkadres
- het broadcastadres

Praktisch:
-  `Get-NetIPConfiguration | Format-Table`
-  `Get-NetIPConfiguration | ft`
-  `Get-NetIPAddress | ft`
-  ...

Opmerking: IPv6 spreekt over **PrefixLength** i.p.v. **subnetmasker**. In Powershell-commando's wordt deze term ook voor IPv4 gebruikt.

## Gedrag van de TCP/IP-stack van een OS

Een host kijk steeds naar zijn eigen TCP/IP-configuratie om te bepalen of het een pakket zal verzenden en zoja, op welke interface het dit pakket zal verzenden.

Hiervoor worden o.a. gebruikt:

- de subnetmaskers
- de routeringstabellen

> Voorbeeld: Stel een host heeft slechts 1 netwerkadapter en heeft zelf IP `192.168.1.1/24`. Deze host pingt naar `192.168.0.1`. Het OS beslist dan het pakket **niet** rechtstreeks te sturen want de bestemming zit op een ander IP-netwerk. Vervolgens kijkt het OS naar de route-tabellen: in de meeste gevallen komt dit neer op het pakket doorsturen naar de **default gateway**. Indien deze niet in ingesteld, wordt het pakket **niet** verstuurd. Mocht het wel verstuurd worden, is het de taak van de router (de default gateway) om alsnog te trachten het pakket af te leveren aan de bestemming.
> Als een host meerdere netwerkadapters heeft, is het natuurlijk mogelijk dat de bestemmingeling via een andere netwerkadapter toch de bestemming kan bereiken.

## Praktisch

Overziten van Powershell-commando's i.v.m. netwerken: 

- `Get-Command -Module NetAdapter`
- `Get-Command -Module NetTCPIP`


# Netwerkprofielen in client-OS
Netwerkprofielen zijn (op Windows) een client-beveiliging. Mogelijke profielen zijn o.a.:
- **Private**: wanneer je in een vertrouwde omgeving bent, er worden meer connecties van buiten af toegelaten
- **Public**: wanneer je in een publieke omgeving bent (zoals een publiek WiFi-netwerk), er worden minder connecties toegelaten, de **firewall** is stricter ingesteld

Praktisch:
- `Get-Command -Module NetConnection`
- `Get-NetConnectionProfile`
- `Set-NetConnectionProfile`
- `firewall.cpl`
- `WF.msc`

Netwerkprofielen kunnen *ge-fine-tuned* worden met de **Windows Firewall**.
