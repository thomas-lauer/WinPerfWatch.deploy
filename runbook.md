# Runbook.md
## WinPerfWatch – „Was tun bei CPU / MEM / DISK Incident“

---

## 1. Zweck des Runbooks

Dieses Runbook beschreibt **standardisierte Incident-Response-Schritte** für durch **WinPerfWatch** erkannte Incidents:

- CPU
- Memory (commit-basiert)
- Disk (Throughput / Latenz / Queue)

Ziel ist:
- schnelle Ursachenanalyse
- reproduzierbares Vorgehen
- saubere Übergabe an 2nd / 3rd Level
- Vermeidung von Schnellschüssen (z. B. unnötige Reboots)

---

## 2. Allgemeine Regeln (immer zuerst)

1. **Nicht sofort eingreifen**
   - WinPerfWatch meldet erst nach *Sustained Samples + EWMA + Hysterese*
   - Kurzzeit-Peaks sind bereits herausgefiltert

2. **Zeitfenster merken**
   - Incident-Start-Zeitpunkt notieren
   - Dauer beobachten

3. **TopN-Prozesse prüfen**
   - Prozesse werden automatisch im Log aufgelistet
   - Fokus auf:
     - ungewöhnliche Prozessnamen
     - stark steigende Werte
     - bekannte Wartungsjobs

---

## 3. CPU Incident

### 3.1 Bedeutung
- Anhaltend hohe CPU-Auslastung
- Oft Thread- oder Kontextwechsel-bedingt
- Auf Servern kritischer als auf Clients

### 3.2 Typische Ursachen
- fehlerhafte Applikation
- Endlosschleifen
- RDS-User mit rechenintensiven Jobs
- Virenscanner / Backup / Indexierung

### 3.3 Vorgehen

**Schritt 1 – Log prüfen**
- `INCIDENT START CPU`
- `INCIDENT CPU process …`

**Schritt 2 – Prozess bewerten**
- CPU > 30 % über längere Zeit → verdächtig
- Mehrere Prozesse → System-/Thread-Problem

**Schritt 3 – Kontext prüfen**
- RDS: SessionUser analysieren
- SQL: parallele Abfragen?
- FILE: AV/Backup-Lauf?

**Schritt 4 – Maßnahme**
- Prozess priorisieren (temporär)
- Applikationsverantwortlichen informieren
- Nur im Notfall: Prozess beenden

**Nicht empfohlen**
- Reboot ohne Ursachenanalyse
- Dauerhafte CPU-Affinitäten

---

## 4. Memory Incident (Commit-basiert)

### 4.1 Bedeutung
- Commit-Limit wird ausgeschöpft
- Task-Manager-Wert entspricht WinPerfWatch

### 4.2 Typische Ursachen
- Memory-Leaks
- zu viele gleichzeitige Sessions
- SQL Max Server Memory falsch konfiguriert
- Java/.NET Anwendungen ohne Limits

### 4.3 Wichtige Kennzahlen
- `memCommittedPct`
- `pagingUsagePct`
- `availMb` (nur Diagnose)

### 4.4 Vorgehen

**Schritt 1 – Paging prüfen**
- Paging > 20 % = kritisch

**Schritt 2 – TopN Prozesse**
- Stark wachsender WorkingSet?
- Bekannte Leaks?

**Schritt 3 – Kontext**
- SQL: Max Memory prüfen
- RDS: Useranzahl vs. RAM

**Schritt 4 – Maßnahme**
- Applikationsneustart geplant
- RAM erweitern (wenn dauerhaft)
- SQL Memory korrekt begrenzen

**Warnung**
- „Available MB niedrig“ allein ist **kein** Incident

---

## 5. Disk Incident

### 5.1 DiskMode verstehen

| Mode | Bedeutung |
|-----|----------|
| AnyOf | Durchsatz ODER Latenz ODER Queue |
| LatencyOrQueue | bevorzugt für SQL |
| ThroughputOnly | Kopier-/Fileserver |
| QueueOnly | Storage-Analyse |

### 5.2 Typische Ursachen
- Storage überlastet
- falsch platzierte Logs / TempDB
- gleichzeitige Backups
- Virenscanner auf Datenvolumen

### 5.3 Vorgehen

**Schritt 1 – Art des Incidents**
- Hohe MB/s → Kopierjob?
- Hohe Latenz → Storage-Problem
- Hohe Queue → IO-Stau

**Schritt 2 – TopN Prozesse**
- Schreib-/Leselast trennen
- Einzelprozess vs. viele

**Schritt 3 – Kontext**
- SQL: TempDB, Index-Rebuild?
- FILE: Benutzerkopien?
- DC: Replikation?

**Schritt 4 – Maßnahme**
- Jobs zeitlich entkoppeln
- AV-Exclusions setzen
- Storage-Team informieren

---

## 6. Eskalationsmatrix

| Dauer | Maßnahme |
|-----|---------|
| < 5 min | Beobachten |
| 5–15 min | Analyse & Verantwortliche informieren |
| > 15 min | Gezielte technische Maßnahme |
| Wiederkehrend | Architektur-/Kapazitätsprüfung |

---

## 7. Übergabe an 2nd / 3rd Level

Weiterzugeben:
- Log-Auszug (Incident Start → End)
- TopN-Prozesse
- ServerProfile
- Zeitpunkt & Dauer
- durchgeführte Maßnahmen

---

## 8. Grundsatz

> **WinPerfWatch ist kein Alarm, sondern ein Diagnose-Instrument.**  
> Ziel ist Verstehen – nicht reflexartiges Handeln.

---

**Ende des Runbooks**
