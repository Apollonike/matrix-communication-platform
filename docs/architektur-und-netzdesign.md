# Architektur und Netzdesign

## Überblick

Dieses Dokument beschreibt den aktuell umgesetzten Infrastrukturstand der Matrix Communication Platform auf Netzwerk- und Plattformebene.

Der Fokus liegt auf der sauberen Trennung öffentlich erreichbarer Dienste vom restlichen internen Netzwerk, der kontrollierten Einbindung über pfSense sowie der Vorbereitung einer sicheren und professionell dokumentierbaren Betriebsumgebung.

---

## Ziel der Netzarchitektur

Die Plattform wird nicht im normalen internen Client-Netz betrieben, sondern in einem separaten Netzwerksegment für öffentlich erreichbare Dienste.

Ziele dieser Trennung sind:

- Begrenzung der Angriffsfläche
- Trennung von Public Services und internem Standard-LAN
- kontrollierte Veröffentlichung über pfSense und HAProxy
- Vorbereitung für weitere externe Dienste wie Webserver und Mailserver
- nachvollziehbare Sicherheitsarchitektur für ein professionelles Infrastruktur-Setup

---

## Netzwerksegmentierung

### PublicServices VLAN

Für öffentlich erreichbare Dienste wurde ein eigenes VLAN eingerichtet:

- **VLAN-ID:** `100`
- **Bezeichnung:** `PublicServices`

Dieses Segment ist für Systeme vorgesehen, die später direkt oder indirekt aus dem Internet erreichbar sein sollen, darunter:

- Matrix-Plattform
- Webserver
- Mailserver

---

## pfSense-Konfiguration

Auf pfSense wurde VLAN 100 auf dem internen LAN-Uplink erstellt und einem eigenen Interface zugewiesen.

### Interface

- **Interface-Name:** `PublicServices`
- **Parent-Interface:** `igc1`
- **VLAN-Interface:** `igc1.100`

### IPv4-Konfiguration

- **IPv4-Adresse:** `10.100.0.1/24`

### DHCP

Für das Interface `PublicServices` wurde ein DHCP-Bereich eingerichtet.

- **Subnetz:** `10.100.0.0/24`
- **DHCP-Range:** `10.100.0.100 - 10.100.0.150`

Der DHCP-Dienst auf pfSense wurde erfolgreich verifiziert.

---

## Proxmox-Konfiguration

Die Matrix-VM wird auf Proxmox VE betrieben.

### VM

- **VM-Name:** `matrix`
- **Gastbetriebssystem:** Debian 13
- **Netzwerkinterface in der VM:** `ens18`

### Virtuelle Netzwerkkarte

Die VM-Netzwerkkarte ist auf der Proxmox-Seite wie folgt konfiguriert:

- **Bridge:** `vmbr0`
- **VLAN-Tag:** `100`

### Wichtiger Punkt: VLAN-aware Bridge

Damit VLAN-getaggter Verkehr korrekt zur VM durchgereicht werden kann, musste die verwendete Linux-Bridge auf Proxmox VLAN-aware konfiguriert werden.

Erforderliche Bridge-Eigenschaft:

    bridge-vlan-aware yes

Fehlte diese Option, erhielt die VM trotz korrektem VLAN-Tag keine DHCP-Lease und fiel auf APIPA/Link-Local-Adressierung zurück.

Nach Aktivierung von VLAN-awareness und einem späteren Neustart des Proxmox-Hosts wurde die Verbindung korrekt hergestellt.

---

## Switch-Pfad

Der VLAN-Pfad zwischen pfSense und Proxmox verläuft über zwei Switches:

- ein vorgelagerter Kupfer-Switch
- ein zusätzlicher GF-/Core-Switch vor dem Proxmox-Host

Für die erfolgreiche Ende-zu-Ende-Verbindung von VLAN 100 war entscheidend, dass VLAN 100 auf allen beteiligten Ports entlang des Pfads korrekt mitgeführt wird.

### Wichtige Erkenntnis

Ein VLAN funktioniert nur dann stabil, wenn es auf **jedem** beteiligten Link korrekt konfiguriert ist.

Insbesondere relevant waren:

- Port pfSense → Switch
- Uplink zwischen den Switches
- Port Switch → Proxmox

---

## GF-Switch-Konfiguration

Auf dem GF-/Core-Switch wurde VLAN 100 explizit angelegt und den beteiligten Trunk-Ports zugewiesen.

### VLAN 100

- VLAN 100 wurde als eigenes VLAN erstellt
- relevante Ports wurden als **Tagged** für VLAN 100 konfiguriert

### Port-Settings

Für die verwendeten Uplink-/Trunk-Ports wurden folgende Parameter verwendet:

- **Mode:** `Trunk`
- **PVID:** `1`
- **Accept Frame Type:** `All`
- **Ingress Filtering:** `Enabled`
- **TPID:** `0x8100`

Diese Konfiguration erlaubt:

- untagged Standardverkehr im nativen VLAN
- zusätzlich getaggten Verkehr für VLAN 100

---

## Matrix-VM Netzwerkstand

Die Matrix-VM wurde zunächst testweise per DHCP in VLAN 100 eingebunden.

Nach erfolgreicher Fehleranalyse und Stabilisierung wurde die Konnektivität mit statischer Adressierung verifiziert.

### Statische Testkonfiguration

    Adresse: 10.100.0.10/24
    Gateway: 10.100.0.1
    DNS:     10.100.0.1, 1.1.1.1

Diese Konfiguration wurde erfolgreich getestet.

### Ergebnis

- Layer-2-Konnektivität zum pfSense-Gateway funktioniert
- ARP-Auflösung funktioniert
- ICMP zum Gateway funktioniert
- DNS-Auflösung funktioniert

Anschließend wurde DHCP erneut getestet und funktionierte ebenfalls erfolgreich.

### Aktueller erfolgreicher DHCP-Zustand

Die VM erhielt per DHCP eine Adresse aus dem konfigurierten Bereich, zum Beispiel:

- `10.100.0.103/24`

sowie:

- Gateway über `10.100.0.1`
- DNS-Auflösung funktionsfähig

---

## DNS in Debian

In der Debian-VM musste zusätzlich sichergestellt werden, dass die Resolver-Konfiguration korrekt gesetzt ist.

Für den Betrieb wurde als Resolver verwendet:

- `10.100.0.1`
- `1.1.1.1`

Erst nach korrekter Resolver-Konfiguration funktionierte die Namensauflösung erwartungsgemäß.

---

## Fehleranalyse und Troubleshooting

Im Rahmen der Inbetriebnahme wurden mehrere Fehlerursachen systematisch eingegrenzt.

### 1. Fehlende VLAN-awareness auf Proxmox-Bridge

Symptom:

- VM erhielt keine DHCP-Lease im VLAN 100
- stattdessen APIPA-Adresse (`169.254.x.x`)

Ursache:

- verwendete Bridge war nicht VLAN-aware

Lösung:

- `bridge-vlan-aware yes`
- anschließend Netzwerk neu laden bzw. Proxmox später sauber neu starten

### 2. VLAN 100 auf dem GF-Switch zunächst nicht vorhanden

Symptom:

- VLAN 100 funktionierte nicht bis zur VM
- andere VLANs für WLAN liefen bereits

Ursache:

- VLAN 100 war auf dem GF-Switch zunächst noch nicht angelegt

Lösung:

- VLAN 100 erstellen
- Trunk-Mitgliedschaft für die beteiligten Ports setzen

### 3. Asymmetrisches ARP-/DHCP-Verhalten

Zwischenzeitlich zeigte sich ein Zustand, in dem:

- DHCP Discover pfSense erreichte
- DHCP Offer von pfSense gesendet wurde
- ARP Replies von pfSense sichtbar waren
- der Gast diese Antworten jedoch nicht sauber verarbeitete

Die Konfiguration wirkte logisch korrekt, der Zustand war jedoch nicht konsistent.

Lösung:

- saubere Reinitialisierung
- Neustart des Proxmox-Hosts

Danach funktionierte die Kommunikation korrekt.

### 4. DNS-Auflösung im Gast

Symptom:

- IP-Konnektivität vorhanden
- Namensauflösung funktionierte nicht

Lösung:

- Resolver in Debian korrekt setzen

---

## Aktueller Stand

Die Netzwerkbasis für die Matrix-Plattform ist bis hierhin erfolgreich hergestellt.

### Erfolgreich umgesetzt

- separates VLAN für Public Services eingerichtet
- pfSense-Interface `PublicServices` konfiguriert
- DHCP-Bereich für VLAN 100 eingerichtet
- Proxmox-Bridge VLAN-aware konfiguriert
- Matrix-VM mit VLAN-Tag 100 angebunden
- Switch-Pfad bis Proxmox erfolgreich hergestellt
- statische und dynamische Adressierung erfolgreich getestet
- Gateway- und DNS-Erreichbarkeit bestätigt

### Noch offen

- restriktive Firewall-Regeln für VLAN 100
- interne DNS-Namensauflösung für die Zielsysteme
- HAProxy-Veröffentlichung für Matrix
- Installation von PostgreSQL und Synapse
- spätere Einbindung von Coturn
- spätere Einbindung weiterer Public-Services-Systeme wie Web und Mail

---

## Nächste Schritte

Die logischen nächsten Arbeitspunkte sind:

1. Firewall-Regelwerk für `PublicServices` definieren
2. feste Adress- und Namensstruktur für Matrix, Web und Mail festlegen
3. Debian-Basishärtung abschließen
4. Matrix-Serverkomponente installieren
5. Veröffentlichung über HAProxy vorbereiten

---

## Zusammenfassung

Mit VLAN 100 wurde eine eigene Public-Services-Zone aufgebaut, in der die Matrix-Plattform sicherheitsorientiert und getrennt vom Standard-LAN betrieben werden kann.

Die technische Inbetriebnahme zeigte, dass gerade bei VLAN-, Switch- und Hypervisor-Übergängen nicht nur die Zielkonfiguration entscheidend ist, sondern auch die saubere Reinitialisierung der beteiligten Komponenten. Durch systematische Prüfung von DHCP, ARP, Routing und Resolver-Konfiguration konnte die Verbindung bis zur Debian-VM erfolgreich hergestellt werden.

Damit ist die Infrastrukturgrundlage für die weitere Umsetzung der Matrix Communication Platform geschaffen.