# Ceph Cluster on a Budget

## Vortragstitel

**Ceph Cluster on a Budget**

## Kurzbeschreibung

In diesem Projekt wurde ein Ceph-Cluster aus vorhandener, gebrauchter Hardware aufgebaut. Die Storage-Nodes bestehen aus alten Computern, Servern, Switches und unterschiedlichen Festplatten. Ergänzt wird die lokale Infrastruktur durch Hetzner Cloud Nodes, die als Ceph-Manager, Monitors und VPN-Endpunkte dienen.

Ziel des Setups ist es, mit möglichst geringen Kosten ein funktionsfähiges, verteiltes Storage-System aufzubauen, das sowohl lokal im Keller betrieben wird als auch über eine statische öffentliche IP via WireGuard erreichbar ist.

---

# Architekturübersicht

## Grundidee

Der Cluster besteht aus zwei Hauptbereichen:

1. **Cloud-Komponenten bei Hetzner**
   - Statische öffentliche IP
   - WireGuard VPN
   - Ceph Manager
   - Ceph Monitors
   - externe Erreichbarkeit

2. **Lokale Storage-Infrastruktur**
   - Physische Storage-Nodes im Keller
   - HDDs und wenige SSDs unterschiedlicher Größe
   - lokales Storage-Backbone-Netzwerk
   - Ceph OSDs
   - Monitoring- und Logging-Agenten

---

# Cloud-Infrastruktur

## Hetzner Cloud Nodes

Es werden **3 Hetzner Cloud Nodes** eingesetzt.

| Node | Rolle |
|---|---|
| `CephMaster` | Ceph Manager, Monitor, VPN Server |
| `CephClient1` | Ceph Manager / Monitor / Client |
| `CephClient2` | Ceph Manager / Monitor / Client |

## VPN

Der zentrale VPN-Endpunkt ist:

```text
CephMaster
```
Dieser Node stellt über WireGuard die Verbindung zwischen der externen Welt und dem lokalen Ceph-Cluster her.

Zweck der Hetzner Nodes

Die Hetzner Cloud Nodes werden genutzt für:

statische öffentliche IP-Adresse
WireGuard VPN
Ceph Monitor Services
Ceph Manager Services
Zugriff von außen auf den lokalen Cluster
Trennung zwischen öffentlicher Erreichbarkeit und lokaler Storage-Infrastruktur
Lokale Infrastruktur
Standort

Die Storage-Hardware befindet sich lokal im Keller.

Physische Nodes

Aktuell besteht der lokale Storage-Teil aus:

13 physischen Nodes

Im Monitoring-Screenshot sind aktuell 10 OSD Hosts sichtbar. Der Cluster ist laut Beschreibung inzwischen auf 13 physische Nodes angewachsen.

Betriebssystem

Alle Nodes laufen mit:
Debian 13

Storage-Aufbau
Gesamtkapazität

Die gesamte Cluster-Kapazität beträgt:
89.1 TB

Genutzte Kapazität

Aktuell genutzt:

26.3 TB

Verfügbare Kapazität

Im Screenshot wird eine verfügbare Kapazität von etwa:

70.5 %

angezeigt.

OSDs

Es sind insgesamt:
37 OSDs
im Cluster verteilt.

Festplatten

Die Nodes verwenden unterschiedliche HDDs mit Größen zwischen:


300 GB bis 7 TB

Die Storage-Verteilung ist bewusst heterogen:

manche Nodes haben bis zu 6 OSDs
andere Nodes haben nur 2 OSDs
die Festplattengrößen unterscheiden sich stark
der Cluster besteht aus vorhandener Hardware statt einheitlicher Server-Hardware
SSDs

Es gibt zwei Nodes mit jeweils zusätzlicher SSD:

Diese SSDs sind nicht exklusiv als BlueStore-Device eingerichtet.

Netzwerkdesign
Internetanbindung

Die lokale Internetanbindung ist limitiert auf:

500 MB/s eingehend
500 MB/s ausgehend

Diese Verbindung wird primär für Datenverkehr nach außen genutzt.

Herausforderung

Da alle Storage-Nodes lokal stehen und die externe Verbindung begrenzt ist, wäre es ineffizient, internes Ceph-Rebalancing oder Storage-Traffic über das VPN beziehungsweise über die externe Verbindung laufen zu lassen.

Lokales Backbone-Netzwerk

Deshalb wurde zusätzlich ein lokales Netzwerk aufgebaut.

Dieses Netzwerk dient als internes Storage-Backbone für:

Ceph-internen Traffic
OSD-Kommunikation
Rebalancing
Recovery
Backfill
Replikation
Storage-nahe Kommunikation zwischen lokalen Nodes
IP-Konfiguration

Im lokalen Backbone gibt es keinen DHCP-Server.

Die Netzwerkinterfaces werden deshalb auf jeder Node manuell mit festen IP-Adressen konfiguriert.

Eigenschaften:

statische lokale IPs
keine DHCP-Abhängigkeit
IPs bleiben dauerhaft gleich
einfachere Fehlersuche
stabilere Ceph-Netzwerkkonfiguration
VPN-Traffic

Über das VPN läuft hauptsächlich der externe Datenverkehr.

Ziel ist es, die verfügbare externe Bandbreite möglichst vollständig auszunutzen, ohne Ceph-internen Storage-Traffic unnötig über die Internetanbindung zu führen.

Ceph-Komponenten
Manager

Die Ceph Manager laufen auf den Hetzner Cloud Nodes.

Monitors

Die Ceph Monitors laufen ebenfalls auf den Hetzner Cloud Nodes.

Im Screenshot sichtbar:

Monitors: 5
Mon Session: 86

OSDs

Die OSDs laufen auf den lokalen Storage-Nodes.

Aktueller Stand laut Beschreibung:

37 OSDs
13 physische Storage-Nodes

Beispielhafte Node-Namen aus dem Monitoring

Im Grafana-Screenshot sind unter anderem folgende Hosts sichtbar:

CephMaster
CephClient1
CephClient2
Ceph-Tower2
Ceph-Tower4
Ceph-Tower5
ceph-tower3
ceph-tower7
ceph-tower8
ceph-tower10
huse-storage
munin-herner

Monitoring und Observability
Dienste auf jeder Node

Auf jeder Node laufen folgende Komponenten:

Prometheus

Logs

Die Logs werden gesammelt mit:

Loki

Visualisierung

Die Visualisierung erfolgt über:

Grafana

Alerting

Ein Alertmanager ist vorhanden, aber aktuell noch nicht konfiguriert.

Alertmanager: vorhanden
Konfiguration: noch offen

Monitoring-Screenshot: Cluster-Zustand im Idle

Die Screenshots wurden im Idle-Zustand aufgenommen.

Cluster-Kapazität
| Metrik            |     Wert |
| ----------------- | -------: |
| Cluster Capacity  |  89.1 TB |
| Used Capacity     |  26.3 TB |
| Available         |   70.5 % |
| Number of Objects | 5.60 Mio |
| Bytes Written     | 122.9 MB |
| Bytes Read        | 101.4 MB |

Durchsatz
| Metrik           |      Wert |
| ---------------- | --------: |
| Write Throughput | 40.7 kB/s |
| Read Throughput  |   0.0 B/s |

IOPS
| Metrik     |       Wert |
| ---------- | ---------: |
| Write IOPS | 1.00 ops/s |
| Read IOPS  | 0.80 ops/s |

Ceph Health / Status-nahe Werte
| Metrik                  |  Wert |
| ----------------------- | ----: |
| Difference              | -2.00 |
| Monitor Sessions        |    86 |
| Monitors                |     5 |
| OSDs                    |    37 |
| OSD Hosts im Screenshot |    10 |

Latenzen
| Metrik               |        Wert |
| -------------------- | ----------: |
| Avg Apply Latency    |     8.35 ms |
| Avg Commit Latency   |     8.35 ms |
| Avg Op Write Latency |   0.0857 ms |
| Avg Op Read Latency  | 0.000183 ms |


Monitoring-Screenshot: Host- und Ressourcenwerte
Ressourcenübersicht

| Metrik         |     Wert |
| -------------- | -------: |
| OSD Hosts      |       10 |
| AVG CPU Busy   |   7.45 % |
| AVG RAM Used   |   30.2 % |
| Physical Cores |     1440 |
| AVG Disk Load  |   5.14 % |
| Network Load   | 25.3 MiB |

Beobachtung

Die Screenshots zeigen den Cluster im Idle-Betrieb. Die durchschnittliche CPU-Auslastung und Disk-Load sind niedrig, was darauf hinweist, dass der Cluster zu diesem Zeitpunkt kaum aktive Last verarbeitet.

Gleichzeitig sind im CPU-Graphen periodische Peaks sichtbar, vermutlich verursacht durch Hintergrundprozesse, Monitoring, Ceph-interne Aufgaben oder kurze Aktivitätsphasen einzelner Nodes.

Besonderheiten des Setups
Budget-orientierter Aufbau

Das Setup basiert nicht auf neuer, homogener Enterprise-Hardware, sondern auf vorhandener beziehungsweise alter Hardware:

alte Computer
alte Server
vorhandene Switches
HDDs unterschiedlicher Größe
einzelne SSDs
günstige Cloud-Nodes für Management und VPN
Heterogene Hardware

Der Cluster ist bewusst heterogen aufgebaut:

unterschiedliche Node-Typen
unterschiedliche Anzahl an OSDs pro Node
unterschiedliche HDD-Größen
gemischte Storage-Leistung
einzelne SSDs ohne dedizierte BlueStore-Rolle
Hybrid-Architektur

Das System kombiniert lokale Infrastruktur mit Cloud-Komponenten:

Hetzner Cloud
    ↓ WireGuard VPN
Lokales Keller-Storage
    ↓ internes Backbone-Netzwerk
Ceph OSD Nodes

Trennung von externem und internem Traffic

Ein wichtiger Designpunkt ist die Trennung von:

externem Zugriff über WireGuard
internem Ceph-Storage-Traffic über das lokale Backbone

Dadurch wird verhindert, dass Recovery, Rebalancing oder OSD-Kommunikation unnötig über die begrenzte Internetanbindung laufen.

Technische Kerndaten
| Kategorie               | Wert                                            |
| ----------------------- | ----------------------------------------------- |
| Projekt                 | Ceph Cluster on a Budget                        |
| Sprache des Vortrags    | Deutsch                                         |
| Storage-Technologie     | Ceph                                            |
| Cloud Provider          | Hetzner Cloud                                   |
| VPN                     | WireGuard                                       |
| VPN Server              | CephMaster                                      |
| Cloud Nodes             | 3                                               |
| Lokale physische Nodes  | 13                                              |
| OSDs                    | 37                                              |
| Cluster-Kapazität       | 89.1 TB                                         |
| Genutzte Kapazität      | 26.3 TB                                         |
| HDD-Größen              | 300 GB bis 7 TB                                 |
| SSDs                    | 2 × 300 GB                                      |
| Betriebssystem          | Debian 13                                       |
| Monitoring              | Prometheus, Grafana                             |
| Logging                 | Loki, Promtail                                  |
| Exporter                | ceph-exporter, node-exporter                    |
| Alerting                | Alertmanager vorhanden, noch nicht konfiguriert |
| Internetanbindung lokal | 500 MB/s in und out                             |
| Internes Netzwerk       | statisches lokales Backbone ohne DHCP           |


Mögliche Kernaussage für den Vortrag

Ein Ceph-Cluster muss nicht zwingend aus teurer, homogener Enterprise-Hardware bestehen. Mit vorhandener Hardware, sauberer Netzwerksegmentierung, Cloud-basierten Management-Nodes und gutem Monitoring lässt sich auch mit begrenztem Budget ein funktionierender, verteilter Storage-Cluster aufbauen.

Der wichtigste Architekturpunkt ist dabei die saubere Trennung zwischen öffentlichem Zugriff über VPN und internem Storage-Traffic über ein lokales Backbone-Netzwerk.

Mögliche Themenblöcke für den Vortrag
1. Motivation
Warum Ceph?
Warum ein eigener Cluster?
Warum Budget-Hardware?
Lernprojekt, Storage-Bedarf oder Homelab-Erweiterung
2. Hardware
alte Computer und Server
HDDs unterschiedlicher Größe
einzelne SSDs
Switches
lokale Nodes im Keller
3. Cloud-Anteil
Hetzner Cloud Nodes
statische IP
WireGuard
Ceph Manager und Monitors
4. Netzwerk
lokale Internetlimitierung
VPN-Design
internes Backbone
statische IPs ohne DHCP
Trennung von Client-Traffic und Storage-Traffic
5. Ceph-Setup
OSD-Verteilung
heterogene Festplatten
Monitor- und Manager-Design
Kapazität und Nutzung
Herausforderungen durch unterschiedliche Disk-Größen
6. Monitoring
ceph-exporter
node-exporter
Prometheus
Loki
Promtail
Grafana
Alertmanager als zukünftiger Ausbau
7. Learnings
Ceph funktioniert auch mit heterogener Hardware
Netzwerkdesign ist entscheidend
Monitoring ist Pflicht
alte Hardware bringt unvorhersehbare Performance-Unterschiede
statische IPs erleichtern Debugging
Cloud-Nodes können sinnvoll als stabile externe Einstiegspunkte dienen
8. Ausblick
Alertmanager konfigurieren
bessere Rollenverteilung
dedizierte BlueStore-Devices prüfen
Kapazitätsplanung verbessern
automatisiertes Provisioning
Failover-Tests
Monitoring-Dashboards erweitern