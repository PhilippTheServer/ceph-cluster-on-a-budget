# Ceph Cluster on a Budget

> **12-Minuten-Konferenzvortrag** · Sprache: Deutsch

Ein funktionierender, verteilter Ceph-Storage-Cluster aus gebrauchter Hardware — kombiniert mit günstigen Cloud-Nodes für Management und externe Erreichbarkeit.

---

## Kernaussage

> Ceph braucht keine teure, homogene Enterprise-Hardware. Mit vorhandenen Maschinen, sauberer Netzwerksegmentierung und Cloud-basierten Management-Nodes entsteht auch mit kleinem Budget ein stabiler, verteilter Storage-Cluster.

---

## Vortragsgliederung

1. [Motivation](#1-motivation)
2. [Hardware](#2-hardware)
3. [Cloud-Anteil](#3-cloud-anteil)
4. [Netzwerkdesign](#4-netzwerkdesign)
5. [Ceph-Setup](#5-ceph-setup)
6. [Monitoring](#6-monitoring)
7. [Learnings](#7-learnings)
8. [Ausblick](#8-ausblick)

---

## 1. Motivation

- Warum Ceph? — verteilter, selbstverwalteter Storage ohne Vendor Lock-in
- Warum eigener Cluster? — Lernprojekt, wachsender Storage-Bedarf, Homelab
- Warum Budget-Hardware? — vorhandene Maschinen nutzen statt neu kaufen

---

## 2. Hardware

### Lokale Storage-Nodes (Keller)

- **13 physische Nodes** — alte Computer und Server
- **37 OSDs** verteilt über alle Nodes
- **Heterogene Festplatten:** 300 GB bis 7 TB (HDD), 2 × 300 GB SSD
- Manche Nodes: bis zu 6 OSDs · andere Nodes: nur 2 OSDs
- SSDs ohne dedizierte BlueStore-Rolle
- Betriebssystem: **Debian 13**

### Cluster-Kapazität

| Metrik            |      Wert |
| ----------------- | --------: |
| Cluster Capacity  |   89.1 TB |
| Used Capacity     |   26.3 TB |
| Available         |    70.5 % |
| Number of Objects | 5,60 Mio. |

---

## 3. Cloud-Anteil

### Hetzner Cloud Nodes

| Node          | Rolle                                    |
| ------------- | ---------------------------------------- |
| `CephMaster`  | Ceph Manager, Monitor, WireGuard-Server  |
| `CephClient1` | Ceph Manager, Monitor, Client            |
| `CephClient2` | Ceph Manager, Monitor, Client            |

### Warum Cloud für Management?

- **Statische öffentliche IP** ohne eigenen Anschluss zu exponieren
- **WireGuard VPN** als sicherer Tunnel zum lokalen Keller
- Ceph Monitor und Manager laufen stabil unabhängig vom lokalen Stromnetz
- Klare Trennung: Cloud = Einstiegspunkt, Keller = Storage

---

## 4. Netzwerkdesign

### Herausforderung

Die lokale Internetanbindung ist auf **500 Mbit/s** begrenzt. Ceph-interner Traffic (Rebalancing, Recovery, Replikation) würde diese Leitung überlasten, wenn er über das VPN läuft.

### Lösung: Zwei getrennte Netzwerke

```
Hetzner Cloud  (statische IP, WireGuard-Endpunkt)
      │
      │  WireGuard VPN  ← externer Client-Traffic
      │
Lokales Keller-Netzwerk
      │
      │  internes Backbone  ← Ceph-Storage-Traffic
      │
OSD Nodes (lokale statische IPs, kein DHCP)
```

| Netzwerk          | Zweck                                             |
| ----------------- | ------------------------------------------------- |
| WireGuard VPN     | Externer Zugriff, Admin-Verbindungen              |
| Lokales Backbone  | OSD-Kommunikation, Rebalancing, Recovery, Backfill |

**Statische IPs** auf jedem Node — kein DHCP, einfacheres Debugging, stabile Ceph-Konfiguration.

---

## 5. Ceph-Setup

### Komponenten-Übersicht

| Komponente | Anzahl | Standort        |
| ---------- | -----: | --------------- |
| Manager    |      3 | Hetzner Cloud   |
| Monitor    |      5 | Hetzner Cloud   |
| OSDs       |     37 | Lokale Nodes    |
| OSD Hosts  |     13 | Keller          |

### Node-Namen (Beispiele aus dem Monitoring)

`CephMaster` · `CephClient1` · `CephClient2` · `Ceph-Tower2` · `Ceph-Tower4` · `Ceph-Tower5` · `ceph-tower3` · `ceph-tower7` · `ceph-tower8` · `ceph-tower10` · `huse-storage` · `munin-herner`

---

## 6. Monitoring

### Stack

| Komponente     | Aufgabe                        |
| -------------- | ------------------------------ |
| `ceph-exporter`| Ceph-Cluster-Metriken          |
| `node-exporter`| Host-Metriken (CPU, RAM, Disk) |
| Prometheus     | Metriken-Sammlung              |
| Loki + Promtail| Log-Aggregation                |
| Grafana        | Visualisierung                 |
| Alertmanager   | Alerting (noch nicht konfiguriert) |

### Dashboard: Cluster-Zustand (Idle)

![Cluster Stats Idle](images/Cluster-stats-idle.png)

| Metrik               |        Wert |
| -------------------- | ----------: |
| Write Throughput     |  40.7 kB/s  |
| Read Throughput      |    0.0 B/s  |
| Write IOPS           | 1.00 ops/s  |
| Read IOPS            | 0.80 ops/s  |
| Avg Apply Latency    |     8.35 ms |
| Avg Commit Latency   |     8.35 ms |
| Monitor Sessions     |          86 |

### Dashboard: Host- und Ressourcenwerte (Idle)

![Host Stats Idle](images/Host-stats-idle.png)

| Metrik         |     Wert |
| -------------- | -------: |
| OSD Hosts      |       10 |
| AVG CPU Busy   |   7.45 % |
| AVG RAM Used   |   30.2 % |
| Physical Cores |     1440 |
| AVG Disk Load  |   5.14 % |
| Network Load   | 25.3 MiB |

> Die Screenshots zeigen den Cluster im Idle-Betrieb. Niedrige CPU- und Disk-Auslastung — periodische CPU-Peaks sind durch Ceph-Hintergrundprozesse und Monitoring bedingt.

---

## 7. Learnings

- **Ceph funktioniert mit heterogener Hardware** — unterschiedliche Disk-Größen und Node-Typen sind kein Blocker
- **Netzwerkdesign ist entscheidend** — Storage-Traffic muss vom externen Traffic getrennt sein
- **Monitoring ist Pflicht** — ohne Sichtbarkeit ist ein verteiltes System schwer zu betreiben
- **Statische IPs vereinfachen Debugging** erheblich
- **Cloud-Nodes als stabile externe Einstiegspunkte** sind eine günstige und zuverlässige Lösung
- Alte Hardware bringt **unvorhersehbare Performance-Unterschiede** — einplanen

---

## 8. Ausblick

- Alertmanager konfigurieren
- Dedizierte BlueStore-Devices prüfen (WAL/DB auf SSD)
- Kapazitätsplanung verbessern
- Automatisiertes Provisioning (Ansible / Cephadm)
- Failover-Tests durchführen
- Monitoring-Dashboards erweitern

---

## Technische Kerndaten

| Kategorie               | Wert                                            |
| ----------------------- | ----------------------------------------------- |
| Storage-Technologie     | Ceph                                            |
| Cloud Provider          | Hetzner Cloud                                   |
| VPN                     | WireGuard                                       |
| Cloud Nodes             | 3 (Manager + Monitor)                           |
| Lokale physische Nodes  | 13                                              |
| OSDs                    | 37                                              |
| Cluster-Kapazität       | 89.1 TB                                         |
| Genutzte Kapazität      | 26.3 TB (≈ 29.5 %)                              |
| HDD-Größen              | 300 GB – 7 TB                                   |
| SSDs                    | 2 × 300 GB                                      |
| Betriebssystem          | Debian 13                                       |
| Monitoring              | Prometheus, Grafana                             |
| Logging                 | Loki, Promtail                                  |
| Internetanbindung lokal | 500 Mbit/s symmetrisch                          |
| Internes Netzwerk       | Statisches Backbone, kein DHCP                  |
