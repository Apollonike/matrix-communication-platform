# Matrix Communication Platform

## Überblick

Dieses Repository dokumentiert die Zielarchitektur, Konzeption und spätere Umsetzung einer selbst betriebenen Kommunikationsplattform auf Basis des Matrix-Protokolls.

Im Mittelpunkt des Projekts stehen nicht nur die technische Bereitstellung einer Messaging-Plattform, sondern vor allem **Datenschutz**, **digitale Souveränität**, **sichere Kommunikation** und die **Integration in bestehende Infrastruktur- und Monitoring-Prozesse**.

Die Plattform soll eine kontrollierbare Alternative zu zentral betriebenen Kommunikationsdiensten schaffen, bei der Kommunikationsdaten, Betriebsdaten und relevante Metadaten nicht unnötig an externe Plattformanbieter abgegeben werden. Gleichzeitig soll sie als sicherer Kanal für Team-Kommunikation, Benachrichtigungen und infrastrukturelle Statusmeldungen genutzt werden.

Die Architektur ist speziell für Umgebungen ausgelegt, in denen:

- die Kernplattform innerhalb einer virtualisierten Infrastruktur auf Proxmox VE betrieben wird
- keine direkt nutzbare öffentliche IPv4-Adresse am Standort vorhanden ist
- die externe Erreichbarkeit über einen vorgeschalteten öffentlichen Linux-Server mit WireGuard-Anbindung zur pfSense erfolgt
- **HAProxy auf pfSense** für die gezielte Veröffentlichung interner Dienste genutzt wird
- ein zweiter öffentlicher Server für TURN/STUN-Dienste bereitgestellt werden kann
- die Plattform zunächst auf Messaging, Audioanrufe, kleine Videocalls und Benachrichtigungen aus der Systemlandschaft ausgerichtet ist

Die öffentliche Namensbasis des Projekts ist:

- **hive42.de**

---

## Fachliche Motivation

Die Motivation dieses Projekts geht über den reinen Betrieb eines selbst gehosteten Messengers hinaus.

### Datenschutz und digitale Souveränität

Ein zentraler Beweggrund ist die Reduktion der Abhängigkeit von fremdbetriebenen Kommunikationsplattformen. Durch den Betrieb der zentralen Matrix-Infrastruktur in der eigenen Umgebung verbleiben Benutzerverwaltung, Kommunikationsflüsse, Verwaltungsdaten und relevante Metadaten im eigenen technischen Verantwortungsbereich.

Das Projekt verfolgt damit insbesondere folgende Ziele:

- Reduktion der Preisgabe von Kommunikationsmetadaten an externe Drittanbieter
- stärkere Kontrolle über Benutzerkonten, Räume und Kommunikationsflüsse
- Nutzung einer offenen, dokumentierten und föderationsfähigen Protokollbasis
- Aufbau einer nachvollziehbaren und revisionsfähigen Kommunikationsinfrastruktur

### Sichere Kommunikation

Die Plattform soll einen sicheren Kommunikationskanal für kleine Arbeitsgruppen und technische Umgebungen bereitstellen. Dabei spielt insbesondere verschlüsselte Kommunikation eine zentrale Rolle.

Wesentliche Zielsetzungen sind:

- vertrauliche Kommunikation auf Basis moderner Verschlüsselungsmechanismen
- kontrollierbare Identitäten und Raumstrukturen
- klare Trennung zwischen öffentlicher Erreichbarkeit und interner Plattformlogik
- minimierte Abhängigkeit von proprietären Messaging-Plattformen

### Integration in Infrastruktur und Monitoring

Ein weiterer wesentlicher Projektaspekt ist die Nutzung der Plattform als **Benachrichtigungs- und Alarmierungskanal** für technische Systeme.

Die Matrix-Plattform soll nicht nur für direkte Benutzerkommunikation dienen, sondern auch Meldungen aus der Systemlandschaft empfangen können, zum Beispiel aus:

- Monitoring-Systemen wie **Checkmk**
- Firewall- und Netzwerkinfrastrukturen wie **pfSense**
- weiteren technischen Diensten, die Status- oder Alarmmeldungen erzeugen

Damit entsteht eine Kommunikationsplattform, die nicht nur den klassischen Chat-Anwendungsfall abdeckt, sondern auch als sicherer Transportweg für infrastrukturelle Ereignisse genutzt werden kann.

### Zielbild des Projekts

Das Projekt verbindet damit mehrere Disziplinen in einer gemeinsamen Plattform:

- sichere Team-Kommunikation
- Self-Hosting und Plattformbetrieb
- Datenschutz und Souveränität
- Monitoring-Integration
- technische Alarmierung
- modular erweiterbare Infrastrukturarchitektur

---

## Projektziel

Ziel dieses Projekts ist die Planung und der Aufbau einer selbst betriebenen Matrix-basierten Kommunikationsplattform mit folgenden Eigenschaften:

- kontrollierte Veröffentlichung ins Internet
- Betrieb der zentralen Plattformdienste in der eigenen Infrastruktur
- saubere Trennung zwischen Messaging-, Relay- und zukünftiger Konferenzlogik
- nachvollziehbare und dokumentierbare Netzarchitektur
- optionale Föderation
- modulare Erweiterbarkeit für spätere Ausbaustufen
- sicherer Kommunikationskanal für Benutzer und Systemmeldungen

Die Plattform soll in der ersten Betriebsstufe insbesondere folgende Funktionen bereitstellen:

- kontrollierte Team-Kommunikation
- Matrix-Räume für Arbeitsgruppen oder Projekte
- Datei- und Bildaustausch
- Audioanrufe
- kleine Videocalls mit wenigen Teilnehmern
- Übermittlung von Benachrichtigungen und Ereignismeldungen aus der Systemlandschaft

Große Videokonferenzen mit vielen gleichzeitigen Teilnehmern sind in der ersten Ausbaustufe bewusst nicht das primäre Ziel.

---

## Architekturprinzipien

Die Zielarchitektur folgt sechs zentralen Grundprinzipien:

### 1. Die zentrale Plattform bleibt in der eigenen Infrastruktur

Benutzerdaten, Datenbank und die zentrale Matrix-Serverkomponente verbleiben innerhalb der eigenen virtualisierten Umgebung.

### 2. Die öffentliche Eintrittskante bleibt minimal

Der bereits vorhandene öffentliche Linux-Server übernimmt ausschließlich die Rolle als Internet-Einstiegspunkt und WireGuard-Endpunkt. Zusätzliche Plattformdienste werden dort bewusst nicht betrieben.

### 3. HAProxy auf pfSense übernimmt die kontrollierte Veröffentlichung

Die externe Veröffentlichung interner Dienste erfolgt über **HAProxy auf pfSense**. Dabei wird Port **443** nicht exklusiv durch die Matrix-Plattform belegt, sondern bleibt als gemeinsamer HTTPS-Einstiegspunkt für mehrere Dienste verfügbar.

Die Zuordnung zu den jeweiligen internen Backends erfolgt DNS- bzw. hostnamenbasiert, zum Beispiel über Subdomains wie:

- matrix.hive42.de
- weitere HTTPS-Dienste unter zusätzlichen Hostnamen

Dadurch können mehrere Web- und Infrastrukturdienste parallel über dieselbe öffentliche HTTPS-Portstruktur veröffentlicht werden, ohne dass einzelne Anwendungen Port **443** exklusiv beanspruchen.

### 4. TURN wird separat betrieben

Audio-/Video-Relay für WebRTC-basierte Verbindungen wird auf einem zweiten, direkt öffentlich erreichbaren Server bereitgestellt.

### 5. Echtzeit-Medienverteilung wird erst bei Bedarf erweitert

Eine SFU ist nicht Teil der ersten Zielstufe und wird erst dann eingeführt, wenn Gruppen-Videocalls in größerem Umfang tatsächlich erforderlich werden.

### 6. Die Architektur bleibt erweiterbar

Die Plattform soll ohne grundlegende Neustrukturierung um zusätzliche Komponenten wie SFU, stärkere öffentliche Systeme oder erweiterte Föderationsszenarien ergänzt werden können.

---

## Ziel-Szenario

Die Plattform ist für folgende Einsatzprofile gedacht:

- kleine Arbeitsgruppen
- technische Projektteams
- Vereinsumgebungen
- interne Test- und Demonstrationsumgebungen
- professionelle Self-Hosting- und Infrastruktur-Szenarien

Der geplante Betriebsrahmen umfasst:

- eine bewusst begrenzte Benutzerzahl
- Fokus auf Messaging, sichere Kommunikation und Benachrichtigung
- ergänzend Audioanrufe
- kleine Videocalls mit wenigen Teilnehmern
- kontrollierte Veröffentlichung und nachvollziehbare Sicherheitsgrenzen
- Anbindung technischer Systeme zur Übermittlung von Status- und Alarmmeldungen

---

## Zielarchitektur

### Aktuelle Zielstufe

Die vorgesehene erste Zielstufe umfasst:

- Matrix-Serverkomponente auf einer Debian-VM in Proxmox VE
- PostgreSQL auf derselben VM
- externe Erreichbarkeit über einen bestehenden öffentlichen Linux-Server
- Weiterleitung definierter Verbindungen durch einen WireGuard-Tunnel in die pfSense
- **HAProxy auf pfSense** als zentrale Reverse-Proxy- und Veröffentlichungskomponente
- separater Coturn-Server auf einem zweiten öffentlichen Linux-Server
- zunächst keine SFU

Diese Architektur schafft eine saubere Grundlage für:

- Matrix-Messaging
- Raumstruktur für Teams und Projekte
- Datei- und Bildaustausch
- Audioanrufe
- kleine Videocalls
- Übermittlung technischer Benachrichtigungen
- spätere modulare Erweiterungen

---

## Logische Topologie

**Primärer Veröffentlichungsweg**

flowchart TB
    Internet1[Internet] --> Public1[Public Server 1<br/>IPv4/IPv6<br/>WireGuard endpoint<br/>Port forwarding only]
    Public1 --> WG[WireGuard Tunnel]
    WG --> pfSense[pfSense<br/>HAProxy / Firewall / Routing]
    pfSense --> DMZ[DMZ / Server Segment]
    DMZ --> Proxmox[Proxmox VE]
    Proxmox --> Debian[Debian VM<br/>Matrix server component + PostgreSQL]

    Internet2[Internet] --> Public2[Public Server 2<br/>Coturn / STUN / TURN]

    Internet3[Internet] --> SFU[Additional or upgraded public servers<br/>SFU for larger group video calls]

---

## Komponenten und Rollen

### Öffentlicher Server 1

Dieser Server ist bereits vorhanden und bleibt bewusst minimal.

**Aufgabe**

- öffentliche IPv4-/IPv6-Erreichbarkeit
- WireGuard-Endpunkt
- definierte Portweiterleitung in Richtung pfSense
- keine Matrix-Anwendungsdienste
- keine Datenbank
- kein TURN
- keine SFU

**Begründung**

Der Server stellt ausschließlich die öffentliche Eintrittskante bereit. Dadurch bleibt der exponierte Internet-Host übersichtlich, wartungsarm und klar abgegrenzt.

### pfSense

pfSense bildet die zentrale Übergabe- und Sicherheitskomponente zwischen öffentlicher Eintrittskante und interner Infrastruktur.

**Aufgabe**

- Annahme des über den Tunnel weitergeleiteten Datenverkehrs
- Firewall- und Freigabelogik
- Routing in die interne Infrastruktur
- Betrieb von **HAProxy** als zentrale Reverse-Proxy- und Veröffentlichungskomponente

**Begründung**

pfSense bleibt die zentrale Kontrollinstanz für eingehende Verbindungen und Sicherheitsrichtlinien. Durch die Integration von HAProxy kann die Veröffentlichung interner Dienste konsistent, nachvollziehbar und kontrolliert umgesetzt werden.

### HAProxy auf pfSense

HAProxy ist Teil der aktiven Veröffentlichungsarchitektur dieses Projekts.

**Aufgabe**

- zentrale HTTPS-Veröffentlichung über Port 443
- Hostname- bzw. SNI-basierte Weiterleitung an interne Backends
- Abbildung öffentlicher DNS-Namen auf interne Dienste
- gemeinsame Nutzung von Port 443 durch mehrere Dienste
- klare Trennung zwischen externer Adresse und internem Service-Backend

**Begründung**

Die Matrix-Plattform soll Port 443 nicht exklusiv belegen. Stattdessen wird HAProxy als zentrale Veröffentlichungsschicht genutzt, sodass HTTPS-Dienste unterschiedlicher Art parallel unter verschiedenen Hostnamen auf Basis derselben öffentlichen Adresse bereitgestellt werden können.

### DMZ / Server-Segment

Die Matrix-Serverkomponente soll nicht im normalen internen Client- oder Administrationsnetz betrieben werden.

**Aufgabe**

- separates Segment für exponierte oder öffentlich erreichbare Dienste
- logische Trennung von Kernsystemen und Internet-nahen Anwendungen
- Begrenzung möglicher Seitwärtsbewegungen bei einer Kompromittierung

**Begründung**

Ein professionelles Sicherheitsdesign geht davon aus, dass ein öffentlich erreichbarer Dienst grundsätzlich angreifbar ist. Die Segmentierung reduziert den möglichen Schadensradius und verhindert, dass eine Kompromittierung der Matrix-VM automatisch Zugriff auf interne Kernsysteme ermöglicht.

### Proxmox VE

Proxmox VE stellt die Virtualisierungsplattform für die zentrale Kommunikationsinfrastruktur bereit.

**Aufgabe**

- Hosting der Debian-VM
- Integration in Snapshot- und Backup-Konzepte
- Grundlage für spätere Skalierung oder Migration

### Debian-VM

Die Debian-VM ist der zentrale Anwendungshost der Plattform.

**Enthaltene Dienste**

- Matrix-Serverkomponente
- PostgreSQL

**Aufgabe**

- Benutzer- und Raumverwaltung
- Nachrichtenverarbeitung
- Speicherung von Konfigurations- und Zustandsdaten
- Medienbereitstellung
- Client-Synchronisation
- optionale Föderation
- Aufnahme und Verteilung technischer Benachrichtigungen

**Begründung**

Für kleine bis mittlere Umgebungen bietet diese Bündelung eine gute Balance aus Übersichtlichkeit, Wartbarkeit und technischer Sauberkeit.

### Öffentlicher Server 2

Dieser zweite öffentliche Server ist für TURN/STUN vorgesehen.

**Enthaltener Dienst**

- Coturn

**Aufgabe**

- Bereitstellung von STUN
- Bereitstellung von TURN-Relay-Funktionalität
- Unterstützung für Audio- und Videocalls bei schwierigen NAT-/Firewall-Szenarien
- direkt erreichbarer öffentlicher Medien-Relay-Endpunkt

**Begründung**

TURN ist auf einem direkt öffentlich erreichbaren System architektonisch deutlich sauberer platziert als hinter mehreren NAT-, Tunnel- und Firewall-Ebenen.

### Optionale spätere SFU

Eine SFU ist nicht Bestandteil der ersten Zielstufe.

**Relevanz**

Eine SFU wird erst sinnvoll, wenn:

- Gruppen-Videocalls regelmäßig genutzt werden
- mehr gleichzeitige Teilnehmer unterstützt werden sollen
- die Skalierung von Echtzeit-Medien zum echten Betriebsfaktor wird

**Mögliche Bereitstellung**

- auf einem aufgerüsteten öffentlichen Server
- auf einem zusätzlichen dedizierten System
- als eigene Ausbaustufe mit höherer Rechenleistung

**Begründung**

Die SFU ist eine eigenständige Medienverteil-Komponente und sollte bewusst als spätere Erweiterung betrachtet werden, nicht als Pflichtbestandteil des initialen Plattformaufbaus.

---

## Sicherheitsarchitektur

Die externe Erreichbarkeit der Matrix-Plattform wird bewusst nicht mit einer direkten Vertrauensstellung gegenüber dem internen Netzwerk gleichgesetzt. Die Architektur folgt dem Grundsatz, dass ein öffentlich erreichbarer Dienst grundsätzlich als potenziell angreifbar betrachtet werden muss. Ziel ist daher nicht nur die sichere Bereitstellung des Dienstes, sondern vor allem die Begrenzung des Schadens im Fall einer Kompromittierung.

### Sicherheitsprinzip

Die Sicherheitsarchitektur orientiert sich an folgenden Grundsätzen:

- keine implizite Vertrauensannahme aufgrund des Netzstandorts
- Segmentierung öffentlich erreichbarer Dienste
- minimale und explizit definierte Netzwerkfreigaben
- kontrollierte Veröffentlichung über zentrale Proxy- und Firewall-Komponenten
- Begrenzung möglicher Seitwärtsbewegungen im internen Netz
- gezielte statt offener Benutzerregistrierung

### Netzwerksegmentierung

Die Matrix-Serverkomponente sollte nicht im normalen internen Client- oder Administrationsnetz betrieben werden. Stattdessen wird ein separates Server- oder DMZ-Segment empfohlen, in dem ausschließlich öffentlich erreichbare oder besonders exponierte Dienste betrieben werden.

Dadurch gilt:

- eine Kompromittierung der Matrix-VM führt nicht automatisch zu direktem Zugriff auf interne Kernsysteme
- Proxmox-Management, Backup-Systeme, Admin-Clients, Storage-Systeme und weitere sensible Komponenten bleiben logisch getrennt
- Zugriffe aus dem Matrix-Segment in andere interne Netzbereiche werden nur explizit und restriktiv erlaubt

### Kontrollierte Veröffentlichung über HAProxy

Die Veröffentlichung der Plattform erfolgt nicht durch direkte Freigabe der Matrix-VM ins Internet, sondern kontrolliert über HAProxy auf pfSense.

Dabei übernimmt HAProxy insbesondere:

- zentrale HTTPS-Veröffentlichung
- Hostname- bzw. SNI-basierte Weiterleitung
- kontrollierte Abbildung externer DNS-Namen auf interne Backends
- gemeinsame Nutzung von Port 443 für mehrere HTTPS-Dienste
- zentrale Protokollierung und nachvollziehbare Veröffentlichungspfade

Die Matrix-Plattform belegt Port 443 dabei nicht exklusiv, sondern wird als einer von mehreren Diensten kontrolliert über die Proxy-Schicht veröffentlicht.

### Restriktive Firewall-Politik

Die Firewall-Regeln sollten nach dem Default-Deny-Prinzip aufgebaut werden. Erlaubt wird nur der tatsächlich notwendige Verkehr.

Für die Matrix-Plattform bedeutet das insbesondere:

- eingehender Verkehr nur über die definierte Veröffentlichungskette
- Zugriff auf das Matrix-Backend nur über die vorgesehenen Proxy- oder Routing-Pfade
- keine pauschalen Freigaben vom Matrix-Segment in interne Vertrauenszonen
- Administrationszugriffe nur aus definierten Management-Netzen oder über VPN
- ausgehende Verbindungen der Matrix-VM ebenfalls nur so weit wie betrieblich erforderlich

### Begrenzung der Seitwärtsbewegung

Ein professionelles Design geht davon aus, dass ein Dienst im schlimmsten Fall kompromittiert werden kann. Entscheidend ist daher, welche weiteren Systeme von dort aus erreichbar sind.

Die Matrix-VM sollte deshalb nicht frei auf interne Infrastruktur zugreifen können. Insbesondere sollte kein unkontrollierter Zugriff auf folgende Bereiche bestehen:

- Hypervisor-Management
- Backup-Infrastruktur
- Storage-Systeme
- Administrationssysteme
- weitere interne Kernsysteme

Damit wird der mögliche Schadensradius im Fall einer Kompromittierung deutlich reduziert.

### Sichere Benutzerverwaltung

Die Plattform ist nicht für offene Selbstregistrierung vorgesehen. Benutzerkonten sollen bewusst und kontrolliert angelegt werden. Falls Registrierungsprozesse später erweitert werden, sollten diese ebenfalls kontrolliert, nachvollziehbar und möglichst token- oder administrativ gesteuert erfolgen.

### Administrationsschnittstellen

Administrationsfunktionen und administrative Endpunkte der Matrix-Plattform dürfen nicht unnötig öffentlich exponiert werden. Administrative Zugriffe sollten nur über definierte und abgesicherte Managementpfade erfolgen.

### Sicherheitsziel dieser Architektur

Die Sicherheitsarchitektur verfolgt nicht den unrealistischen Anspruch, einen öffentlich erreichbaren Dienst absolut unangreifbar zu machen. Ziel ist vielmehr ein professioneller Aufbau, bei dem:

- die Angriffsfläche reduziert wird
- Freigaben nachvollziehbar bleiben
- exponierte Dienste logisch getrennt betrieben werden
- ein erfolgreicher Angriff nicht automatisch zur Kompromittierung des internen Netzwerks führt

Damit wird die Matrix Communication Platform nicht nur funktional, sondern auch sicherheitsarchitektonisch als professionell geplante Infrastrukturkomponente positioniert.

---

## Funktionale Schichten

### Messaging-Schicht

Abgedeckt durch:

- Matrix-Serverkomponente
- PostgreSQL

**Aufgaben**

- Benutzerverwaltung
- Raumverwaltung
- Nachrichtenübermittlung
- Event-Speicherung
- Medien-Metadaten
- Synchronisation der Clients
- optionale Föderationslogik

### Veröffentlichungs-Schicht

Abgedeckt durch:

- öffentlicher Server 1
- pfSense
- HAProxy

**Aufgaben**

- kontrollierte Veröffentlichung interner Dienste
- externe Erreichbarkeit über hive42.de
- zentrale Proxy- und Routing-Logik
- Trennung von öffentlichem Zugriff und internem Backend
- gemeinsame Nutzung von Port 443 für mehrere HTTPS-Dienste

### Medien-Relay-Schicht

Abgedeckt durch:

- Coturn

**Aufgaben**

- STUN-Unterstützung
- TURN-Relay für problematische Netzpfade
- Verbesserung der Erreichbarkeit und Stabilität bei Audio- und Videocalls

### Benachrichtigungs-Schicht

Abgedeckt durch:

- Monitoring- und Infrastruktursysteme
- Schnittstellen oder Benachrichtigungsmechanismen in Richtung Matrix

**Aufgaben**

- Übermittlung technischer Statusmeldungen
- Transport von Alarmierungen und Ereignissen
- Integration von Infrastruktur-Benachrichtigungen in bestehende Kommunikationsräume

### Zukünftige Konferenz-Schicht

Abgedeckt durch:

- optionale SFU

**Aufgaben**

- effiziente Verteilung von Medienströmen in Gruppencalls
- bessere Skalierung bei mehreren aktiven Video-Teilnehmern
- geringere Last auf Endgeräten im Vergleich zu einfachen Mesh-Ansätzen

---

## Warum diese Architektur gewählt wurde

### 1. Betriebshoheit über die zentrale Plattform

Die Kernkomponenten verbleiben in der eigenen Infrastruktur und unter eigener Kontrolle.

### 2. Nutzung bestehender Netzarchitektur

Die vorhandene Kombination aus öffentlichem Linux-Server, WireGuard, pfSense und HAProxy wird sinnvoll weiterverwendet.

### 3. Klare Trennung technischer Rollen

Öffentliche Eintrittskante, Proxy-Schicht, Plattformlogik, Medien-Relay und Benachrichtigungsintegration sind voneinander getrennt.

### 4. Saubere Erweiterbarkeit

Die Architektur erlaubt spätere Ausbaustufen ohne grundlegenden Neuentwurf.

### 5. Datenschutz und sichere Kommunikation als Leitmotiv

Das Projekt adressiert nicht nur technische Betriebsfragen, sondern verfolgt bewusst das Ziel, Kommunikation und technische Benachrichtigungen in einer kontrollierbaren, verschlüsselbaren und unabhängig betreibbaren Plattform zusammenzuführen.

### 6. Sicherheit als eigener Architekturbaustein

Die Plattform wird nicht nur funktional, sondern auch mit Blick auf Angriffsfläche, Segmentierung, Freigabelogik und Schadensbegrenzung geplant. Damit wird sie als professionell entworfene Infrastrukturkomponente und nicht als bloß veröffentlichter Einzelservice behandelt.

---

## Milestones

### Milestone 1 – Architektur und Zielbild definieren

- fachliche Zielsetzung festlegen
- Datenschutz, sichere Kommunikation und Benachrichtigung als Kernziele definieren
- Rollen von Matrix-Serverkomponente, Coturn, HAProxy und optionaler SFU festlegen
- Namens- und Veröffentlichungsstrategie auf Basis von hive42.de definieren

### Milestone 2 – Sicherheits- und Veröffentlichungsarchitektur festlegen

- DMZ-/Server-Segment für exponierte Dienste definieren
- DNS-Konzept für hive42.de festlegen
- HAProxy auf pfSense in die Zielarchitektur einbinden
- Routing über öffentlichen Server 1, WireGuard und pfSense dokumentieren
- externe Hostnamen den internen Diensten zuordnen
- gemeinsame Nutzung von Port 443 für mehrere HTTPS-Dienste sicherstellen
- restriktive Firewall- und Freigabelogik definieren

### Milestone 3 – Zentrale Matrix-Plattform bereitstellen

- Debian-VM auf Proxmox VE bereitstellen
- Matrix-Serverkomponente und PostgreSQL integrieren
- interne Erreichbarkeit und Basisbetrieb herstellen
- Veröffentlichung über HAProxy vorbereiten oder aktivieren
- kontrollierte Benutzeranlage und Registrierungsstrategie festlegen

### Milestone 4 – TURN für Echtzeitkommunikation ergänzen

- öffentlichen Server 2 für Coturn vorsehen
- STUN/TURN in die Kommunikationsarchitektur einbinden
- Grundlage für Audioanrufe und kleine Videocalls schaffen

### Milestone 5 – Benachrichtigungsintegration vorbereiten

- Matrix als Kanal für Status- und Alarmmeldungen vorsehen
- mögliche Quellen wie Checkmk und pfSense konzeptionell einbinden
- Struktur für technische Räume oder Alarmierungsräume definieren

### Milestone 6 – Erweiterbarkeit für spätere Konferenzfunktionen sichern

- SFU als optionale spätere Ausbaustufe dokumentieren
- Architektur so auslegen, dass größere Echtzeit-Medienkomponenten später ausgelagert werden können

---

## DNS- und Namenskonzept

Die öffentliche Basisdomain des Projekts ist:

- hive42.de

Die Veröffentlichung erfolgt hostnamenbasiert über HAProxy auf pfSense. Dadurch bleibt Port 443 als gemeinsamer HTTPS-Einstiegspunkt für mehrere Dienste nutzbar.

Beispielhafte Zuordnung:

- matrix.hive42.de → Matrix-Plattform
- call.hive42.de oder sfu.hive42.de → spätere SFU
- weitere Subdomains → weitere interne HTTPS-Dienste

Dieses Modell verhindert, dass die Matrix-Plattform den HTTPS-Port global exklusiv belegt, und ermöglicht eine saubere parallele Veröffentlichung mehrerer Dienste unter derselben öffentlichen Domainbasis.

---

## Empfohlene Sicherheitsausrichtung

Für die erste Ausbaustufe wird empfohlen:

- zunächst kontrollierter Betrieb
- Benutzerkonten nur gezielt anlegen
- offene Selbstregistrierung deaktiviert lassen
- Föderation erst aktivieren, wenn die Kernplattform stabil läuft
- alle externen Freigaben dokumentieren
- Konfiguration, Datenbank und Medienbestand regelmäßig sichern
- Benachrichtigungsquellen gezielt und nachvollziehbar anbinden
- administrative Zugriffe nur über definierte Managementpfade zulassen

---

## Weiterführende Dokumentation

Ergänzende technische Details zu Architektur, Netzdesign, Sicherheitsbetrachtung und späteren Ausbaustufen werden in wenigen, bewusst zusammengefassten Dokumenten im Verzeichnis docs beschrieben.

---

## Fazit

Diese Architektur beschreibt einen sauberen, modularen und professionell nachvollziehbaren Ansatz für den Aufbau einer selbst betriebenen Matrix-basierten Kommunikationsplattform in einer virtualisierten Infrastruktur.

Sie ist besonders geeignet, wenn:

- die zentrale Plattform in der eigenen Umgebung betrieben werden soll
- die externe Erreichbarkeit über einen bestehenden WireGuard-gestützten Internet-Einstieg erfolgt
- HAProxy auf pfSense als kontrollierte Veröffentlichungsschicht genutzt wird
- TURN auf einem separaten öffentlichen Host bereitgestellt werden soll
- Audioanrufe und kleine Videocalls früh unterstützt werden sollen
- technische Benachrichtigungen sicher und kontrolliert über dieselbe Plattform transportiert werden sollen
- größere Konferenzfunktionen erst später relevant werden
- öffentlich erreichbare Dienste bewusst segmentiert und restriktiv eingebunden werden sollen

Kurz zusammengefasst:

- die zentrale Matrix-Plattform bleibt in der eigenen Infrastruktur
- die Veröffentlichung erfolgt kontrolliert über pfSense und HAProxy
- Port 443 bleibt als gemeinsamer HTTPS-Einstiegspunkt für mehrere Dienste nutzbar
- der öffentliche Weiterleitungsserver bleibt bewusst minimal
- Coturn erhält einen separaten öffentlichen Host
- Benutzerkommunikation und Systembenachrichtigungen werden auf einer gemeinsamen Plattform zusammengeführt
- die Matrix-VM wird sicherheitsorientiert segmentiert betrieben
- eine SFU wird erst bei tatsächlichem Bedarf ergänzt

Damit entsteht eine robuste, klar dokumentierbare und professionell erweiterbare Kommunikationsplattform für kleine Arbeitsgruppen, technische Projektumgebungen und datenschutzorientierte Infrastruktur-Szenarien.
