---
marp: true
theme: rose-pine
class: invert
paginate: true
html: true
footer: 'Ceph Cluster on a Budget'
---

<!-- _class: lead invert -->

# Ceph Cluster on a Budget

### Verteilter Storage aus gebrauchter Hardware

---

## Motivation
<br>


- Speicher Bedarf wächst — kaufen oder selbst bauen?
  - RAM und Speicher sind teuer dieser Tage
- Ausfallsicherheit - wie redundant muss die Lösung sein?
- Alte Hardware liegt rum — warum wegwerfen?


<br>

> Ziel: Speicher Cluster zu möglichst geringen Kosten

---

## Was ist Ceph?

- Ceph verwaltet Speicher von verschiedenen Geräten in einem Cluster und stellt sie nach außen als einheitlichen Speicher bereit, während alle Hardwarekomponenten austauschbar bleiben.

<br>

- Unterstützt Block-, Datei- und Object-Storage gleichzeitig

- Orchestriert sich weitgehend selbst: 
  Rebalancing, Recovery und Replikation laufen automatisch im Hintergrund.

---



![w:700](images/Ceph-aufbau.png)

---

## Womit haben wir gebastelt?

- Alte Desktop-PCs und Server - zwischen 2007 - 2020
  - (noch mit DDR2 RAM)
  - Betriebssystem: Debian 13
- HDDs: 300 GB bis 7 TB bunt gemischt
- 2 × SSD (300 GB)
- Switch


<br>

> Aus allem zusammen gepuzzelt was das Lager noch zu bieten hatte.

---

## Die Zahlen

| Metrik              | Wert            |
| ------------------- | --------------: |
| Physische Nodes     | 13              |
| OSDs                | 37              |
| Cluster-Kapazität   | 89.1 TB         |
| Genutzte Kapazität  | 26.3 TB (29 %)  |
| HDD-Größen          | 300 GB – 7 TB   |
| Betriebssystem      | Debian 13       |

---

## 13 Jahre Hardware — ein Cluster

<div style="display:grid;grid-template-columns:1fr 1fr;gap:1rem;font-size:0.78em;margin-top:0.5rem;">
  <div>

| Node | CPU | Jahr |
|---|---|---|
| ceph-tower10 | i7-970 (Gulftown) | **2010** |
| Ceph-Tower4/5 | i5-3570 (Ivy Bridge) | **2012** |
| CephTower2 | i7-4770 (Haswell) | **2013** |
| tower-storage | i5-6600 (Skylake) | **2015** |
| ceph-tower7 | Xeon E-2236 | **2018** |
| ceph-tower8 | Xeon E-2288G | **2020** |

  </div>
  <div>

**Kurioseste Funde:**

🦕 Älteste Platte: Hitachi 500 GB (~2008)
mit **50.239 Betriebsstunden** — noch aktiv

🖥️ ceph-tower8: brandneuer Xeon E-2288G,
betreibt aber eine **WD Green von 2009** als OSD

💾 ceph-tower5 bootet heute noch von einer
**Samsung HD250HJ — Baujahr ~2007**

  </div>
</div>

> Ceph interessiert sich nicht für das Alter der Hardware — solange die Disk antwortet, wird sie genutzt.

---

## Architektur: Zwei Teile — ein Cluster

<div style="display:grid;grid-template-columns:1fr 1fr;gap:2rem;margin-top:1.5rem;">
  <div style="background:rgba(59,130,246,0.25);border:2px solid #3b82f6;border-radius:12px;padding:1.2rem;">
    <div style="font-size:1.1em;font-weight:bold;margin-bottom:0.5rem;">☁️ Hetzner Cloud</div>
    <div style="font-size:0.85em;line-height:1.8;">
      Ceph Manager<br>
      Ceph Monitor (×5)<br>
      WireGuard-Server<br>
      Statische öffentliche IPs
    </div>
  </div>
  <div style="background:rgba(22,163,74,0.25);border:2px solid #16a34a;border-radius:12px;padding:1.2rem;">
    <div style="font-size:1.1em;font-weight:bold;margin-bottom:0.5rem;">🏠 Lokaler Keller</div>
    <div style="font-size:0.85em;line-height:1.8;">
      13 physische Nodes<br>
      37 Ceph OSDs<br>
      Backbone-Switch<br>
      Statische lokale IPs
    </div>
  </div>
</div>

---

## Hetzner Cloud — Warum?

| Node | Rolle |
| --- | --- |
| `CephMaster` | Manager · Monitor · WireGuard · statische IP |
| `CephClient1` | Manager · Monitor · statische IP · Failover |
| `CephClient2` | Manager · Monitor · statische IP · Failover |

- Statische öffentliche IP — ohne eigenen Anschluss zu exponieren
- Manager & Monitors laufen unabhängig vom Keller-Stromnetz
- CephClient1/2: Failover für Manager/Monitor, **kein VPN-Takeover**

---

## Netzwerk: Das Problem

Lokale Internetanbindung: **500 Mbit/s**

Ceph-interner Traffic belastet die Leitung:

- OSD-Replikation
- Rebalancing
- Recovery & Backfill

<br>

❌ Alles über WireGuard = Leitung sofort gesättigt

---

## Netzwerk: Die Lösung

<div style="display:grid;grid-template-rows:auto auto auto;gap:0.6rem;margin-top:1rem;text-align:center;">
  <div style="background:rgba(59,130,246,0.25);border:2px solid #3b82f6;border-radius:10px;padding:0.8rem;">
    <strong>☁️ Hetzner Cloud</strong> &nbsp;·&nbsp; CephMaster · CephClient1 · CephClient2
  </div>
  <div style="font-size:0.9em;opacity:0.8;padding:0.2rem;">
    ↕&nbsp; <em>WireGuard VPN — externer Client-Traffic (500 Mbit/s ISP-Leitung)</em>
  </div>
  <div style="background:rgba(22,163,74,0.25);border:2px solid #16a34a;border-radius:10px;padding:0.8rem;">
    <strong>🏠 Lokaler Keller</strong> &nbsp;·&nbsp; 13 Nodes · 37 OSDs
    <div style="margin-top:0.5rem;font-size:0.85em;opacity:0.8;">
      ↕&nbsp; <em>Backbone-Switch — Ceph-interner Storage-Traffic</em>
    </div>
    <div style="font-size:0.8em;opacity:0.6;margin-top:0.2rem;">statische IPs · kein DHCP</div>
  </div>
</div>

---

## Ceph-Setup

| Komponente | Anzahl | Standort |
| --- | ---: | --- |
| Manager | 3 | Hetzner Cloud |
| Monitor | 5 | Hetzner Cloud |
| OSDs | 37 | Lokaler Keller |
| OSD Hosts | 13 | Lokaler Keller |

**Heterogene OSD-Verteilung:**
- 2 bis 6 OSDs pro Node
- Festplatten: 300 GB bis 7 TB — bewusst gemischt

---

## Monitoring-Stack

| Tool | Aufgabe |
| --- | --- |
| `ceph-exporter` | Ceph-Cluster-Metriken |
| `node-exporter` | CPU · RAM · Disk · Network |
| Prometheus | Metriken-Sammlung |
| Promtail + Loki | Log-Aggregation |
| Grafana | Visualisierung |
| Alertmanager | ⚠️ Installiert — noch nicht konfiguriert |

---

<!-- _footer: '' -->

## Dashboard — Cluster im Idle

![bg right:65% contain](images/Cluster-stats-idle.png)

**Cluster-Zustand:**
- 89.1 TB Kapazität
- Write: 40.7 kB/s
- 5 Monitors aktiv
- 37 OSDs online
- Latenz: ~8 ms

---

<!-- _footer: '' -->

## Dashboard — Host-Ressourcen

![bg right:65% contain](images/Host-stats-idle.png)

**Im Idle:**
- Ø CPU: 7.45 %
- Ø RAM: 30.2 %
- Ø Disk Load: 5.14 %
- 1440 physische Cores
- Periodische CPU-Peaks durch Ceph-Hintergrundprozesse

---

## Learnings

✅ Ceph läuft auf **heterogener Hardware** — kein Blocker

✅ **Netzwerktrennung ist der kritische Architekturpunkt**

✅ Cloud-Nodes als stabile externe Einstiegspunkte — günstig & effektiv

✅ Statische IPs vereinfachen Debugging erheblich

⚠️ Alte Hardware = unvorhersehbare Performance-Unterschiede

⚠️ Monitoring **von Anfang an** einrichten — nicht nachträglich

---

<!-- _class: lead invert -->

## Ausblick

Alertmanager konfigurieren · Dedizierte BlueStore-Devices
Automatisiertes Provisioning (Cephadm / Ansible)
Failover-Tests · Kapazitätsplanung

<br>

# Danke! Fragen?

`github.com/PhilippTheSurfer`
