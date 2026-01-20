# config_json.md  
**WinPerfWatch – Konfigurationsreferenz**

Diese Datei beschreibt **vollständig und detailliert** alle Konfigurationsoptionen der `config.json` für **WinPerfWatch**.  
Sie dient als **Betriebs-, Tuning- und Übergabedokumentation** für Admins, IT-Betrieb und Architektur.

---

## 1. Grundprinzip

WinPerfWatch ist **profilbasiert** aufgebaut:

- Globale Einstellungen gelten **immer**
- `ServerProfile` bestimmt den **Server-Typ**
- Profile (`RDS`, `FILE`, `SQL`, `DC`) liefern **zusätzliche Metriken & Schwellen**
- Incidents werden **EWMA-geglättet**, mit **Hysterese** und **Sustained Samples** erkannt
- **Prozess-Attribution** läuft **nur bei Incident-Start** und dann **getaktet**

---

## 2. Service

```json
"Service": {
  "ServiceName": "WinPerfWatch",
  "RunAsWindowsService": false
}
````

| Feld                  | Beschreibung                                                 |
| --------------------- | ------------------------------------------------------------ |
| `ServiceName`         | Name des Windows-Dienstes                                    |
| `RunAsWindowsService` | Informativ (echter Modus wird über `-c / -s / -u` gesteuert) |

---

## 3. ServerProfile

```json
"ServerProfile": "RDS"
```

Bestimmt den **Server-Typ**:

| Wert   | Typische Rolle                  |
| ------ | ------------------------------- |
| `RDS`  | Remote Desktop / Terminalserver |
| `FILE` | Fileserver                      |
| `SQL`  | SQL Server                      |
| `DC`   | Domain Controller               |

---

## 4. Sampling

```json
"Sampling": {
  "IntervalSeconds": 5
}
```

| Feld              | Beschreibung             |
| ----------------- | ------------------------ |
| `IntervalSeconds` | Messintervall (Sekunden) |

**Empfehlung:**

* 5s → Echtzeitnah
* 10s → sehr große Server / viele Prozesse

---

## 5. Sustained (Entprellung)

```json
"Sustained": {
  "Samples": 6
}
```

| Feld      | Beschreibung                                       |
| --------- | -------------------------------------------------- |
| `Samples` | Anzahl aufeinanderfolgender Messungen bis Incident |

**Beispiel:**
5s × 6 = **30 Sekunden dauerhafte Überlast**, bevor Incident startet.

---

## 6. EWMA – Glättung

```json
"Ewma": {
  "Alpha": 0.30
}
```

Exponentially Weighted Moving Average zur **Glättung von Peaks**.

| Alpha | Verhalten    |
| ----- | ------------ |
| 0.2   | sehr stabil  |
| 0.3   | empfohlen    |
| 0.5   | sehr reaktiv |

---

## 7. Logging

```json
"Logging": {
  "IncidentCooldownSeconds": 120,
  "WritePeriodicSystemLine": true,
  "PeriodicSystemLineEverySamples": 6
}
```

| Feld                             | Beschreibung                             |
| -------------------------------- | ---------------------------------------- |
| `IncidentCooldownSeconds`        | Mindestpause zwischen gleichen Incidents |
| `WritePeriodicSystemLine`        | Regelmäßige SYS-Statuszeilen             |
| `PeriodicSystemLineEverySamples` | Wie oft (in Samples)                     |

---

## 8. Discovery (PerfCounter / WMI)

```json
"Discovery": {
  "EnableCounterDiscovery": true,
  "LogDiscoveredCounters": true,
  "CpuFallbackMinValidPercent": 0.5,
  "DiskFallbackMinValidMBps": 0.05
}
```

| Feld                         | Beschreibung                     |
| ---------------------------- | -------------------------------- |
| `EnableCounterDiscovery`     | Dynamische Counter-Erkennung     |
| `LogDiscoveredCounters`      | Protokolliert aktivierte Counter |
| `CpuFallbackMinValidPercent` | Unterhalb → WMI-Fallback         |
| `DiskFallbackMinValidMBps`   | Unterhalb → WMI-Fallback         |

**Hintergrund:**
Verhindert bekannte PerfCounter-0%-Bugs auf Servern.

---

## 9. SQL Discovery

```json
"SqlDiscovery": {
  "Enable": true,
  "PreferredInstanceName": "MSSQLSERVER",
  "LogDiscoveredCategories": false
}
```

| Feld                      | Beschreibung                    |
| ------------------------- | ------------------------------- |
| `Enable`                  | Aktiviert SQL Counter Discovery |
| `PreferredInstanceName`   | SQL Instanz                     |
| `LogDiscoveredCategories` | Debug-Ausgabe                   |

---

## 10. Thresholds (globale Incidents)

### CPU & Memory (Hysterese)

```json
"CpuTotalHighPercentStart": 85.0,
"CpuTotalHighPercentEnd": 75.0,
"MemCommittedHighPercentStart": 90.0,
"MemCommittedHighPercentEnd": 85.0,
"PagingHighPercent": 20.0
```

| Prinzip | Bedeutung                        |
| ------- | -------------------------------- |
| Start   | Incident beginnt                 |
| End     | Incident endet (tiefer = stabil) |

**Memory-Hinweis:**
`MemCommitted*` basiert auf **Committed Bytes in Use** (wie der Task-Manager).  
`PagingHighPercent` dient als kritisches Paging-Signal.

---

### Disk – Durchsatz, Latenz, Queue

```json
"DiskTotalHighMBpsStart": 120.0,
"DiskTotalHighMBpsEnd": 90.0,
"DiskAvgReadSecHighStart": 0.030,
"DiskAvgReadSecHighEnd": 0.020,
"DiskQueueLenHighStart": 8.0,
"DiskQueueLenHighEnd": 6.0
```

**Latenzwerte in Sekunden**
0.030 = 30 ms

---

### DiskMode (entscheidend!)

```json
"DiskMode": "AnyOf"
```

| Mode             | Bedeutung                                |
| ---------------- | ---------------------------------------- |
| `AnyOf`          | Durchsatz **oder** Latenz **oder** Queue |
| `ThroughputOnly` | Nur MB/s                                 |
| `LatencyOnly`    | Nur Latenz                               |
| `QueueOnly`      | Nur Queue                                |
| `LatencyOrQueue` | Optimal für SQL                          |

---

## 11. Attribution (Prozessanalyse)

```json
"Attribution": {
  "TopN": 10,
  "DuringIncidentEverySeconds": 30
}
```

### Kerngedanke

✔ **Keine Dauerlast**
✔ Attribution **nur bei Incident-Start**
✔ Danach **getaktet**

| Feld                         | Beschreibung          |
| ---------------------------- | --------------------- |
| `TopN`                       | Top Prozesse          |
| `DuringIncidentEverySeconds` | Attribution-Intervall |

---

### Filter

```json
"MinCpuPercentToList": 1.0,
"MinIoMBpsToList": 1.0,
"MinRamMBToList": 200.0
```

Verhindert Rauschen.

---

### Identität & Session

```json
"IncludeOwner": true,
"IncludeSessionUser": true,
"SessionUserFallbackToOwner": true
```

| Quelle | Priorität |
| ------ | --------- |
| WTS    | Primär    |
| Owner  | Fallback  |

---

### Caches

```json
"IdentityCacheTtlSeconds": 180,
"SessionCacheTtlSeconds": 120,
"ServiceCacheTtlSeconds": 300
```

Reduziert WMI-Last massiv.

---

## 12. Profiles – Zusatzmetriken je Servertyp

### RDS

```json
"ProcessorQueueLengthHigh": 10,
"ContextSwitchesPerSecHigh": 60000
```

### FILE

```json
"LanmanServerFilesOpenHigh": 2500,
"LanmanServerBytesTotalPerSecMBpsHigh": 300
```

### SQL

```json
"SqlPageLifeExpectancyLow": 300,
"SqlBufferCacheHitRatioLow": 90
```

### DC

```json
"ProcessorQueueLengthHigh": 10,
"ContextSwitchesPerSecHigh": 50000
```

---

## 13. Best Practices

### SQL Server

```json
"DiskMode": "LatencyOrQueue"
```

### File Server

```json
"DiskMode": "AnyOf"
```

### RDS
```json
"DiskMode": "AnyOf"
```
CPU + Memory priorisieren, Disk moderat.

---


