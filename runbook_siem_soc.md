# runbook_siem_soc.md
## WinPerfWatch – SIEM / SOC Incident Runbook

---

## 1. Zielgruppe & Zweck

Dieses Runbook richtet sich an:
- SOC (Security Operations Center)
- SIEM-Analysten
- IT-Betrieb / On-Call

Ziel:
- Einheitliche Klassifikation von Incidents aus **WinPerfWatch**
- Korrelation mit SIEM-Events
- Saubere Eskalation und Dokumentation

---

## 2. Event-Quellen

WinPerfWatch liefert Events über:
- NLog File Logs (JSON oder Text)
- Optional: Windows Eventlog
- Optional: SIEM Agent (Filebeat, NXLog, Sentinel Agent)

Empfohlene Quelle:
```
logs/perf.jsonl
```

---

## 3. Zentrale Event-Felder

| Feld | Beschreibung |
|----|-------------|
| `eventType` | Art des Events |
| `label` | CPU / MEM / DISK |
| `profile` | RDS / FILE / SQL / DC |
| `host` | Servername |
| `timestamp` | Ereigniszeit |
| `pid` | Prozess-ID |
| `processName` | Prozessname |
| `sessionUser` | Benutzer |
| `sessionId` | Session |
| `cpuPct` | CPU-Auslastung |
| `memCommittedPct` | Commit Memory |
| `diskMBps` | Disk Durchsatz |
| `diskLatencyMs` | Disk Latenz |
| `pagingUsagePct` | Paging-Auslastung |

---

## 4. Event-Typen (SIEM-relevant)

### 4.1 Incident Lifecycle

| eventType | Bedeutung |
|---------|----------|
| `incident_start` | Incident erkannt |
| `incident_ongoing` | Incident weiterhin aktiv |
| `incident_end` | Incident beendet |
| `incident_process` | Verursachender Prozess |
| `mem_paging_high` | Kritischer Paging-Zustand |

---

## 5. Severity-Mapping (empfohlen)

| Event | Severity |
|-----|----------|
| CPU Incident Start | Medium |
| CPU Incident > 15 min | High |
| MEM Commit > 90 % | High |
| Paging > 20 % | Critical |
| DISK Latency > Threshold | High |
| Wiederkehrender Incident | High |

---

## 6. SOC Analyse-Workflow

### Schritt 1 – Eingang
- Alert aus SIEM
- Host + eventType identifizieren

### Schritt 2 – Korrelation
- Weitere Events vom selben Host?
- Zeitliche Nähe zu:
  - Security Events
  - Backup Jobs
  - Patching
  - Login-Spikes

### Schritt 3 – Attribution
- `incident_process` Events prüfen
- Prozess + User bewerten
- Services / Session Kontext einbeziehen

### Schritt 4 – Klassifikation

| Kategorie | Beschreibung |
|---------|--------------|
| Operational | Backup, AV, Index |
| Application | Leak, Bug |
| User-driven | RDS User |
| Infrastructure | Storage / CPU |
| Security relevant | Crypto-Miner, Malware |

---

## 7. Security-Use-Cases

### 7.1 Crypto-Miner Verdacht
Indikatoren:
- Dauerhafte CPU > 90 %
- Unbekannter Prozessname
- Kein zugehöriger Service
- SessionUser leer oder ungewöhnlich

Aktion:
- Prozess hash prüfen
- Endpoint Scan
- Isolierung erwägen

---

### 7.2 Ransomware Vorstufe
Indikatoren:
- Stark steigende Disk Writes
- Viele Prozesse gleichzeitig
- Kontext: User-Session

Aktion:
- Immediate Eskalation
- Storage I/O stoppen
- Security Incident eröffnen

---

## 8. Eskalationspfad

| Severity | Aktion |
|-------|-------|
| Medium | Ticket |
| High | On-Call informieren |
| Critical | SOC Lead + Incident Manager |

---

## 9. Übergabe an Incident Manager

Beizulegen:
- Event-ID
- Host
- Zeitraum
- Top Prozesse
- Verdachtsklassifikation
- Erste Maßnahmen

---

## 10. Compliance & Audit

- Alle Incidents sind zeitlich nachvollziehbar
- Keine personenbezogenen Daten außerhalb `sessionUser`
- Logs SIEM-konform strukturierbar

---

**Ende des SIEM/SOC Runbooks**
